

# 1 利用 null_resource 的 triggers 触发其他资源更新

社区有人提了一个 Terraform 问题，他写了这样一段 Terraform 代码：

```json
resource "azurerm_key_vault_secret" "service_bus_connection_string" {

  name = "service-bus-connection-string"

  value        = azurerm_servicebus_topic_authorization_rule.mysb.primary_connection_string
  key_vault_id = azurerm_key_vault.main.id
}

resource "azurerm_function_app" "main" {

  name                = "myfn"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  app_service_plan_id = azurerm_app_service_plan.main.id

  enable_builtin_logging = true
  https_only             = true
  os_type                = "linux"

  storage_account_name       = azurerm_storage_account.main.name
  storage_account_access_key = azurerm_storage_account.main.primary_access_key

  version = "~3"

  app_settings = {
    AzureWebJobsServiceBus      = "@Microsoft.KeyVault(SecretUri=${azurerm_key_vault_secret.service_bus_connection_string.id})"
  }
}
```

意思大概是他把一段含有机密信息的连接字符串保存在 Azure KeyVault 服务中，然后创建了一个 Azure Faas 函数，通过 KeyVault 机密引用地址传递该机密。

## 1.1 问题描述

这位老兄发现，如果他修改了机密的内容，也就是`azurerm_key_vault_secret`声明里的`value = azurerm_servicebus_topic_authorization_rule.mysb.primary_connection_string`这一段的值的时候，KeyVault 保存的机密内容的确会正确更新，但 Azure Function 读取到的还是旧的机密引用地址，也就是这段代码中得到的 KeyVault 机密引用地址没有更新：

```
app_settings = {
    AzureWebJobsServiceBus      = "@Microsoft.KeyVault(SecretUri=${azurerm_key_vault_secret.service_bus_connection_string.id})"
  }
```

更加奇怪的是，这之后他什么都没有做，只是重新再执行一次`terraform apply`，该引用地址又被正确更新了？！

## 1.2 问题原因

因为 KeyVault Secret 被设计成是不可变的，所以更新`azurerm_key_vault_secret`的`value`会导致资源被重新创建。Terraform 官网上的相关文档中对该参数的定义如下：

> `value` - (Required) Specifies the value of the Key Vault Secret. 

在 Terraform 中 ，一个参数如果被标记为`Required`，那么它不但是必填项，同时类似数据库记录的主键的概念，主键不同的记录被认定是两条不同的记录，修改记录的主键值可以看作是删除重建之。Terraform 资源的`Required`参数如果发生变化会触发重新创建资源，这就导致了修改`value`后，该`azurerm_key_vault_secret`的`id`也会发生变化。

那么为什么在`azurerm_key_vault_secret`被重新创建之后，我们会发现`azurerm_function_app`中引用的`id`没有变化呢？

Terraform 的工作流含有 Plan 和 Apply 两个主要阶段，首先会分析 Terraform 代码，调用 `terraform refresh`（可以用参数跳过该步骤）读取资源在云端目前的最新状态，再加上 State 文件中记录的状态，三个状态对比出一个执行计划，使得最终产生的云端状态能够符合当前代码描述的状态。

就这个场景而言，Terraform 能够意识到 `azurerm_key_vault_secret`的参数发生了变化，这会导致某种程度的更新，但它无法意识到这个更新会导致`azurerm_key_vault_secret`的`id`发生变化，进而导致`azurerm_function_app`也必须进行更新，所以就发生了他第一次执行`terraform apply`后看到的情况。

当他第二次执行`terraform apply`时，Terraform 记录的 State 文件里，`azurerm_key_vault_secret`的`id`和`azurerm_function_app`里使用的`id`已经对不上了，这时 Terraform 会再生成一个更新`azurerm_function_app`的 Plan，执行后一切恢复正常。

有没有办法让`azurerm_function_app`能在第一次生成 Plan 时就感知到这个变更？

## 1.3 巧用 null_resource 的 triggers

HashiCorp 提供了一个非常常用的内建 Provider —— null。其中最知名的资源就是`null_resource`了，一般它都是和`provisioner`搭配出现，可以用来在某些资源创建完成后执行一些自定义脚本等等。但是它还有一个很有用的参数：

> The `triggers` argument allows specifying an arbitrary set of values that, when changed, will cause the resource to be replaced.

`triggers`参数可以用来指向一些值，只要这些值的内容发生了变动，会导致`null_resource`资源被重新创建，从而生成一个新的`id`。

## 1.4 一个小实验

我们尝试构建一个简单的实验环境来验证一下，首先是这样一段代码：

```
resource "azurerm_key_vault_secret" "example" {
  name         = "secret-sauce"
  value        = "szechuan"
  key_vault_id = azurerm_key_vault.example.id
}

resource "local_file" "output" {
  filename = "${path.module}/output.txt"
  content = azurerm_key_vault_secret.example.id
}
```

我们创建一个`azurerm_key_vault_secret`，然后把它的`id`输出到一个文件里。随后我们复制一下该文件，比如叫`output.bak`好了。随后我们修改`azurerm_key_vault_secret`的`value`到一个新的值，执行`terraform apply`以后，我们会发现`output.txt`与`output.bak`的内容完全一样，说明`value`的更新并没有触发`local_file`的更新。

随后我们把代码改成这样：

```
resource "azurerm_key_vault_secret" "example" {
  name         = "secret-sauce"
  value        = "szechuan2"
  key_vault_id = azurerm_key_vault.example.id
}

resource "null_resource" "example" {
  triggers = {
    trigger = azurerm_key_vault_secret.example.value
  }
}

resource "local_file" "output" {
  filename = "${path.module}/output.txt"
  content = null_resource.example.id == null_resource.example.id ? azurerm_key_vault_secret.example.id : ""
}
```

我们在代码中插入了一个`null_resource`，并设置`triggers`的内容，盯住`azurerm_key_vault_secret.example.value`。在`value`发生变化时，`null_resource`的`id`也会发生变化。

然后我们在`local_file`的代码中，`content`的赋值改成了这样一个三目表达式：`null_resource.example.id == null_resource.example.id ? azurerm_key_vault_secret.example.id : ""`。这个表达式里实际上`null_resource.example.id`是不起作用的，自己等于自己的永真条件会导致仍然使用`azurerm_key_vault_secret.example.id`作为值，但是由于掺入了`null_resource.example.id`，使得 Terraform 在第一次计算 Plan 时就感知到 `local_file` 的内容发生了变化，从而使得我们可以一次`terraform apply`搞定。








