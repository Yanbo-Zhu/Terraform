

# 1 destroy-provisioner中使用变量

我们可以在定义一个`provisioner`块时设置`when`为`destroy`，资源在销毁**之前**会首先执行`provisioner`，可以帮助我们执行一些析构逻辑。但是如果我们在 Destroy-Provisioner 中引用了变量的话，比如这样的代码：

```json
resource "aws_volume_attachment" "attachement_myservice" {
  count         = "${length(var.network_myservice_subnet_ids)}"
  device_name   = "/dev/xvdg"
  volume_id     =   "${element(aws_ebs_volume.ebs_myservice.*.id, count.index)}"
  instance_id   =   "${element(aws_instance.myservice.*.id, count.index)}"

  provisioner "local-exec" {
    command = "aws ec2 stop-instances --instance-ids ${element(aws_instance.myservice.*.id, count.index)} --region ${var.region} && sleep 30"
    when = "destroy"
  }
}
```

那么我们会看见这样的报错信息：

```
|  Error: Invalid reference from destroy provisioner
│ 
│ Destroy-time provisioners and their connection configurations may only reference attributes of the related resource, via 'self', 'count.index', or 'each.key'.
│ 
│ References to other resources during the destroy phase can cause dependency cycles and interact poorly with create_before_destroy.
```

从`0.12`开始 Terraform 会对在 Destroy-Time Provisioner 中引用除`self`、`count.index`、`each.key`以外的变量做警告，从`0.13`开始则会直接报错。

## 1.1 解决方法

目前官方推荐的做法是把需要引用的变量值通过`triggers`“捕获”一下再引用，例如：

```
resource "null_resource" "foo" {
  triggers {
    interpreter = var.local_exec_interpreter
  }
  provisioner {
    when = destroy

    interpreter = self.triggers.interpreter
    ...
  }
}
```

通过这种方法就可以避免这个问题。


