
# 1 利用 null_resource 搭配 replace_triggered_by 更新无法从服务端读取内容的属性

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
