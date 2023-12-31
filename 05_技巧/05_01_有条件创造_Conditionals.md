
使用 condtion ? true_val : fals_val

# 1 理论

![](image/Pasted%20image%2020231121205619.png)

![](image/Pasted%20image%2020231121205805.png)


# 2 有条件创建

Terraform被设计成声明式而非命令式，例如没有常见的`if`条件语句，后来才加上了`count`和`for_each`实现的循环语句(但循环的次数必须是在plan阶段就能够确认的，无法根据其他resource的输出动态决定)

有时候我们需要根据某种条件来判断是否创建一个资源。虽然我们无法使用if来完成条件判断，但我们还有`count`和`for_each`可以帮助我们完成这个目标。

我们以UCloud为例，假如我们正在编写一个旨在被复用的模块，模块的逻辑要创建一台虚拟机，我们的代码可以是这样的：

```
data ucloud_vpcs "default" {
  name_regex = "^Default"
}

data "ucloud_images" "centos" {
  name_regex = "^CentOS 7"
}

resource "ucloud_instance" "web" {
  availability_zone = "cn-bj2-02"
  image_id = data.ucloud_images.centos.images[0].id
  instance_type = "n-basic-2"
}

output "uhost_id" {
  value = ucloud_instance.web.id
}
```

非常简单。但是如果我们想进一步，让模块的调用者决定创建的主机是否要搭配一个弹性公网IP该怎么办？

我们可以在上面的代码后面接上这样的代码：

```
variable "allocate_public_ip" {
  description = "Decide whether to allocate a public ip and bind it to the host"
  type = bool
  default = false
}

resource "ucloud_eip" "public_ip" {
  count = var.allocate_public_ip ? 1 : 0
  name = "public_ip_for_${ucloud_instance.web.name}"
  internet_type = "bgp"
}

resource "ucloud_eip_association" "public_ip_binding" {
  count = var.allocate_public_ip ? 1 : 0
  eip_id = ucloud_eip.public_ip[0].id
  resource_id = ucloud_instance.web.id
}
```

我们首先创建了名为`allocate_public_ip`的输入变量，然后在编写弹性IP相关资源代码的时候都声明了`count`参数，值使用了条件表达式，根据`allocate_public_ip`这个输入变量的值决定是`1`还是`0`.  这实际上实现了按条件创建资源。

需要注意的是，由于我们使用了`count`，所以现在弹性IP相关的资源实际上是多实例资源类型的。我们在`ucloud_eip_association.public_ip_binding`中引用`ucloud_eip.public`时，还是要加上访问下标。由于`ucloud_eip_association.public_ip_binding`与`ucloud_eip.public`实际上是同生同死，所以在这里他们之间的引用还比较简单；

如果是其他没有声明`count`的资源引用它们的话，还要针对`allocate_public_ip`为`false`时`ucloud_eip.public`实际为空做相应处理，比如在`output`中：

```
output "public_ip" {
  value = join("", ucloud_eip.public_ip[*].public_ip)
}
```

使用`join`函数就可以在即使没有创建弹性IP时也能返回空字符串。或者我们也可以用条件表达式：

```
output "public_ip" {
  value = length(ucloud_eip.public_ip[*].public_ip) > 0 ? ucloud_eip.public_ip[0].public_ip : ""
}
```


# 3 有条件选择

![](image/Pasted%20image%2020231118170120.png)


如果 值等于 test, count = 2, 
如果不等于 test, 则 count = 4



