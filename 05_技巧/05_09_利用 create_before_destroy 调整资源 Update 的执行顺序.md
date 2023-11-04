
# 1 利用 create_before_destroy 调整资源 Update 的执行顺序

最近处理了一个问题，有人写了这样一段代码：

```
provider "azurerm" {
  features {
    resource_group {
      prevent_deletion_if_contains_resources = false
    }
  }
}

resource "azurerm_resource_group" "rg" {
  location = "eastus"
  name     = "example"
}

locals {
  environments = toset(["one", "two", "three"])
}

resource "azurerm_public_ip" "lb" {
  for_each = local.environments

  name                = "frontend-lb-${each.key}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  ip_version          = "IPv4"
  sku                 = "Standard"
  zones               = [1, 2, 3]
}

resource "azurerm_lb" "this" {
  name                = "azurelb"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  sku                 = "Standard"

  dynamic "frontend_ip_configuration" {
    for_each = local.environments
    content {
      name                 = frontend_ip_configuration.key
      public_ip_address_id = azurerm_public_ip.lb[frontend_ip_configuration.key].id
    }
  }
}
```

当他从 `local.environments` 中删除一个元素，然后执行 `terraform apply` 时，他遇到了下面的问题：

```
│ Error: deleting Public Ip Address: (Name "azurelb" / Resource Group "example"): network.PublicIPAddressesClient#Delete: Failure sending request: StatusCode=400 -- Original Error: Code="PublicIPAddressCannotBeDeleted" Message="Public IP address /subscriptions/subscription-id/resourceGroups/resource-group/providers/Microsoft.Network/publicIPAddresses/one can not be deleted since it is still allocated to resource /subscriptions/subscription-id/resourceGroups/resource-group/providers/Microsoft.Network/loadBalancers/azurelb/frontendIPConfigurations/one. In order to delete the public IP, disassociate/detach the Public IP address from the resource.  To learn how to do this, see aka.ms/deletepublicip." Details=[]
```

这其实是一个还挺常见的问题，`azurerm_lb.this` 依赖于 `azurerm_public_ip.lb[index]`，正确的变更顺序应该是先更新 `azurerm_lb.this`，再删除 `azurerm_public_ip.lb` 的成员，但是 Terraform 默认的执行顺序会首先尝试执行删除操作，这时因为 ip 仍然被 LoadBalancer 使用着，所以会引发一个错误。

解决方法是给 `azurerm_public.lb` 添加一个 `create_before_destroy`：

```
resource "azurerm_public_ip" "lb" {
  for_each = local.environments

  name                = "frontend-lb-${each.key}"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  ip_version          = "IPv4"
  sku                 = "Standard"
  zones               = [1, 2, 3]

  lifecycle {
    create_before_destroy = true
  }
}
```

`create_before_destroy` 名字里虽然看起来是与 Create 有关，实际上它也会将 Update 与 Create 放在一起调整，声明该参数后实际上是将 `azurerm_public_ip.lb` 的 Delete 推迟到执行 Update 之后再执行了，该问题得解。
