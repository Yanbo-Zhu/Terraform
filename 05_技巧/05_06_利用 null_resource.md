
# 1 什么是 null_resource

## 1.2 null_resource

在这里我们可以使用这种方法的前提是我们使用的UCloud云主机的资源定义允许我们定义资源时声明`network_interface`属性，直接绑定一个公网IP。如果我们使用的云厂商Provider无法让我们在创建主机时绑定公网IP，而是必须事后绑定弹性IP呢？又或者，初始化脚本必须在云主机成功绑定了云盘之后才能成功运行？这种情况下我们还有最后的武器，就是`null_resource`。

`null_resource`可能是Terraform体系中最"不Terraform"的存在，它就是我们用来在Terraform这样一个声明式世界里干各种命令式脏活的工具。
`null_resouce`本身是一个空的resource，只有一个名为`triggers`的参数以及`id`作为输出属性。 

我们看下这个例子：

```json
data "ucloud_images" "centos" {
  name_regex = "^CentOS 7"
}

resource "ucloud_eip" "eip" {
  internet_type = "bgp"
  bandwidth     = 1
  charge_mode   = "traffic"
}

resource "ucloud_disk" "data_disk" {
  availability_zone = "cn-bj2-03"
  disk_size         = 10
  charge_type       = "dynamic"
  disk_type         = "data_disk"
}

resource "ucloud_instance" "web" {
  availability_zone = "cn-bj2-03"
  image_id          = data.ucloud_images.centos.images[0].id
  instance_type     = "n-standard-1"
  charge_type       = "dynamic"
  root_password     = var.root_password
}

resource "ucloud_eip_association" "eip_association" {
  eip_id      = ucloud_eip.eip.id
  resource_id = ucloud_instance.web.id
}

resource "ucloud_disk_attachment" "data_disk" {
  availability_zone = "cn-bj2-03"
  disk_id           = ucloud_disk.data_disk.id
  instance_id       = ucloud_instance.web.id
}

resource "null_resource" "web_init" {
  depends_on = [
    ucloud_eip_association.eip_association,
    ucloud_disk_attachment.data_disk
  ]
  provisioner "remote-exec" {
    connection {
      type     = "ssh"
      host     = ucloud_eip.eip.public_ip
      user     = "root"
      password = var.root_password
    }
    inline = [
      "echo hello"
    ]
  }
}
```

我们假设需要远程执行的操纵是必须在云盘挂载成功以后才可以运行的，那么我们可以声明一个`null_resource`，把provisioner声明放在那里，通过显式声明`depends_on`确保它的执行一定是在云盘挂载结束以后。


# 2 利用 null_resource 的 triggers 触发其他资源更新

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

## 2.1 问题描述

这位老兄发现，如果他修改了机密的内容，也就是`azurerm_key_vault_secret`声明里的`value = azurerm_servicebus_topic_authorization_rule.mysb.primary_connection_string`这一段的值的时候，KeyVault 保存的机密内容的确会正确更新，但 Azure Function 读取到的还是旧的机密引用地址，也就是这段代码中得到的 KeyVault 机密引用地址没有更新：

```
app_settings = {
    AzureWebJobsServiceBus      = "@Microsoft.KeyVault(SecretUri=${azurerm_key_vault_secret.service_bus_connection_string.id})"
  }
```

更加奇怪的是，这之后他什么都没有做，只是重新再执行一次`terraform apply`，该引用地址又被正确更新了？！

## 2.2 问题原因

因为 KeyVault Secret 被设计成是不可变的，所以更新`azurerm_key_vault_secret`的`value`会导致资源被重新创建。Terraform 官网上的相关文档中对该参数的定义如下：

> `value` - (Required) Specifies the value of the Key Vault Secret. 

在 Terraform 中 ，一个参数如果被标记为`Required`，那么它不但是必填项，同时类似数据库记录的主键的概念，主键不同的记录被认定是两条不同的记录，修改记录的主键值可以看作是删除重建之。Terraform 资源的`Required`参数如果发生变化会触发重新创建资源，这就导致了修改`value`后，该`azurerm_key_vault_secret`的`id`也会发生变化。

那么为什么在`azurerm_key_vault_secret`被重新创建之后，我们会发现`azurerm_function_app`中引用的`id`没有变化呢？

Terraform 的工作流含有 Plan 和 Apply 两个主要阶段，首先会分析 Terraform 代码，调用 `terraform refresh`（可以用参数跳过该步骤）读取资源在云端目前的最新状态，再加上 State 文件中记录的状态，三个状态对比出一个执行计划，使得最终产生的云端状态能够符合当前代码描述的状态。

就这个场景而言，Terraform 能够意识到 `azurerm_key_vault_secret`的参数发生了变化，这会导致某种程度的更新，但它无法意识到这个更新会导致`azurerm_key_vault_secret`的`id`发生变化，进而导致`azurerm_function_app`也必须进行更新，所以就发生了他第一次执行`terraform apply`后看到的情况。

当他第二次执行`terraform apply`时，Terraform 记录的 State 文件里，`azurerm_key_vault_secret`的`id`和`azurerm_function_app`里使用的`id`已经对不上了，这时 Terraform 会再生成一个更新`azurerm_function_app`的 Plan，执行后一切恢复正常。

有没有办法让`azurerm_function_app`能在第一次生成 Plan 时就感知到这个变更？

## 2.3 巧用 null_resource 的 triggers

HashiCorp 提供了一个非常常用的内建 Provider —— null。其中最知名的资源就是`null_resource`了，一般它都是和`provisioner`搭配出现，可以用来在某些资源创建完成后执行一些自定义脚本等等。但是它还有一个很有用的参数：

> The `triggers` argument allows specifying an arbitrary set of values that, when changed, will cause the resource to be replaced.

`triggers`参数可以用来指向一些值，只要这些值的内容发生了变动，会导致`null_resource`资源被重新创建，从而生成一个新的`id`。

## 2.4 一个小实验

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











# 3 利用 null_resource 搭配 replace_triggered_by 更新无法从服务端读取内容的属性

曾经处理的一个提问，有人写了这样一段 Terraform 代码：

```json
resource "azurerm_container_group" "this" {
  name                = var.name
  location            = var.location
  resource_group_name = var.resource_group_name
  ip_address_type     = "Private"
  network_profile_id  = azurerm_network_profile.this.id
  os_type             = "Linux"

  container {
    name   = "someName"
    image  = "someImage"
    cpu    = "0.5"
    memory = "0.5"

    commands = ["some", "commands"]

    ports {
      port     = 53
      protocol = "UDP"
    }

    volume {
      mount_path = "/app/conf"
      name       = "someName"
      read_only  = true
      secret = {
        Corefile = base64encode(someContent)
      }
    }

  }

  tags = var.tags
}
```



结果每次执行 `apply` 操作时，都会发现 Terraform 试图重建这个容器：

```json
# module.dns_forwarder.azurerm_container_group.this must be replaced
-/+ resource "azurerm_container_group" "this" {
      ~ exposed_port        = [
          - {
              - port     = 53
              - protocol = "UDP"
            },
        ] -> (known after apply)
      + fqdn                = (known after apply)
      ~ id                  = "/subscriptions/<mySubId>/resourceGroups/<myRgName>/providers/Microsoft.ContainerInstance/containerGroups/<myContainerGroupName>" -> (known after apply)
      ~ ip_address          = "someIp" -> (known after apply)
        name                = "someName"
      - tags                = {} -> null
        # (6 unchanged attributes hidden)

      ~ container {
          - environment_variables        = {} -> null
            name                         = "someName"
          - secure_environment_variables = (sensitive value)
            # (4 unchanged attributes hidden)


          ~ volume {
                name       = "someName"
              ~ secret     = (sensitive value) # forces replacement
                # (3 unchanged attributes hidden)
            }
            # (1 unchanged block hidden)
        }
    }
```

这个问题的原因是 API 在读取容器信息时不会返回 `volume` 的 `secret` 数据，这其实是一个还挺合理的设定，机密数据的确不应该可以直接从 API 返回，但这就导致 Terraform 每次制定变更计划时都会试图重新设置这个值(因为会理解成服务端这个值被修改成了空)，而容器是不可变的，要修改容器的任何配置都会导致容器被重建。


---


有没有办法能够避免这种问题？经验告诉我们，可以使用 `ignore_changes` 让 Terraform 忽略这个属性的变更来避免重建，但如果 `secret` 真的变了怎么办？



我们可以这样干，第一，在 `azurerm_container_group` 添加这样一段 `lifecycle` 块：

```json
lifecycle {
    ignore_changes       = [container[0].volume[0].secret]
    replace_triggered_by = [null_resource.secret_trigger.id]
}
```

这会忽略 `secret` 的变化，但我们同时声明了一个 `replace_triggered_by`，在 `null_resource.secret_trigger.id` 的值发生变化时可以删除重建 `azurerm_container_group` 实例。

其次，我们把 `secret` 的内容提取到一个 `local` 里，这时 `azurerm_container_group` 的 `volume` 看起来大概是这样的：

```json
volume {
    mount_path = "/app/conf"
    name       = "somename"
    read_only  = true
    secret     = {
      Corefile = local.secret
    }
}
```

`local.secret` 存放着使用的机密数据。这时我们再定义一个 `null_resource` 充当触发器：

```json 
locals {
  secret = base64encode("abcdefg")
}

resource "null_resource" "secret_trigger" {
  triggers = {
    trigger = local.secret
  }
}
```

这样在机密数据真的发生变化的时候，`triggers` 会触发 `null_resource` 的重建，导致 `null_resource.secret_trigger.id` 发生变化，进而触发 `azurerm_container_group` 的重建。


