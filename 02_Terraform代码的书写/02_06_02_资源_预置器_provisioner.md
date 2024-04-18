
https://developer.hashicorp.com/terraform/language/resources/provisioners/syntax#provisioners
# 1 预置器 provisioner 
## 1.1 预置器 provisioner 的 类型 

- provisioner "file"  把文件放到远端服务器上 
- provisioner "remote-exec"  {} 在远端server 上执行 {} 中的内容 
- provisioner "local-exec"


### 1.1.1 provisioner "remote-exec"  {} 
![](image/Pasted%20image%2020231117231015.png)


```json
resource "aws_key_pair" "mykey" {
  key_name   = "mykey"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  key_name      = aws_key_pair.mykey.key_name

  provisioner "file" {
    source      = "script.sh"
    destination = "/tmp/script.sh"
  }
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/script.sh",
      "sudo sed -i -e 's/\r$//' /tmp/script.sh",  # Remove the spurious CR characters.
      "sudo /tmp/script.sh",
    ]
  }
  connection {
    host        = coalesce(self.public_ip, self.private_ip)
    type        = "ssh"
    user        = var.INSTANCE_USERNAME
    private_key = file(var.PATH_TO_PRIVATE_KEY)
  }
}

```

### 1.1.2 provisioner "file"
![](image/Pasted%20image%2020231117231331.png)

![](image/Pasted%20image%2020231117231343.png)


![](image/Pasted%20image%2020231119133502.png)

![](image/Pasted%20image%2020231119133521.png)

## 1.2 创建时预置器 Creation-Time Provisioners

默认情况下，资源对象被创建时会运行预置器，在对象更新、销毁时则不会运行。预置器的默认行为时为了引导一个系统。 

如果创建时预置器失败了，那么资源对象会被标记污点(我们将在介绍`terraform taint`命令时详细介绍)。一个被标记污点的资源在下次执行`terraform apply`命令时会被销毁并重建。Terrform的这种设计是因为当预置器运行失败时标志着资源处于半就绪的状态。由于Terraform无法衡量预置器的行为，所以唯一能够完全确保资源被正确初始化的方式就是删除重建。

我们可以通过设置`on_failure`参数来改变这种行为。

By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during _creation_, not during updating or any other lifecycle. They are meant as a means to perform bootstrapping of a system.

If a creation-time provisioner fails, the resource is marked as **tainted**. A tainted resource will be planned for destruction and recreation upon the next `terraform apply`. Terraform does this because a failed provisioner can leave a resource in a semi-configured state. Because Terraform cannot reason about what the provisioner does, the only way to ensure proper creation of a resource is to recreate it. This is tainting.

You can change this behavior by setting the `on_failure` attribute, which is covered in detail below.


### 1.2.1 If a Terraform creation-time provisioner fails, what will occur by default?

If a creation-time provisioner fails, the resource is marked as tainted. A tainted resource will be planned for destruction and recreation upon the next terraform apply .

If a creation-time provisioner fails, the resource is marked as tainted. A tainted resource will be planned for destruction and recreation upon the next terraform apply. Terraform does this because a failed provisioner can leave a resource in a semi-configured state. Because Terraform cannot reason about what the provisioner does, the only way to ensure proper creation of a resource is to recreate it. This is tainting.

## 1.3 销毁时预置器 Destroy-Time Provisioners

如果我们设置预置器的`when`参数为`destroy`，那么预置器会在资源被销毁时执行：

```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    when    = destroy
    command = "echo 'Destroy-time provisioner'"
  }
}
```

销毁时预置器在资源被实际销毁前运行。如果运行失败，Terraform会报错，并在下次运行`terraform apply`操作时重新执行预置器。在这种情况下，需要仔细关注销毁时预置器以使之能够安全地反复执行。

销毁时预置器只有在存在于代码中的情况下才会在销毁时被执行。如果一个resource块连带内部的销毁时预置器块一起被从代码中删除，那么被删除的预置器在资源被销毁时**不会**被执行。要解决这个问题，我们需要使用多个步骤来绕过这个限制：

- 修改资源声明代码，添加`count = 0`参数
- 执行`terraform apply`，运行删除时预置器，然后删除资源实例
- 删除resource块
- 重新执行`terraform apply`，此时应该不会有任何变更需要执行

该限制在未来将会得到解决，但目前来说我们必须节制使用销毁时预置器。

## 1.4 预置器失败行为

默认情况下，预置器运行失败会导致`terraform apply`执行失败。可以通过设置`on_failure`参数来改变这一行为。可以设置的值为：

- continue：忽视错误，继续执行创建或是销毁
- fail：报错并终止执行变更(这是默认行为)。如果这是一个创建时预置器，则在对应资源对象上标记污点

样例：

```
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command    = "echo The server's IP address is ${self.private_ip}"
    on_failure = continue
  }
}
```



# 2 provisioner与user_data

我们在介绍资源时介绍了预置器provisioner。同时不少公有云厂商的虚拟机都提供了cloud-init功能，可以让我们在虚拟机实例第一次启动时执行一段自定义的脚本来执行一些初始化操作。例如我们在"Terraform初步体验"一章里举的例子，在UCloud主机第一次启动时我们通过user_data来调用yum安装并配置了ngnix服务。预置器与cloud-init都可以用于初始化虚拟机，那么我们应该用哪一种呢？

首先要指出的是，provisioner的官方文档里明确指出，由于预置器内部的行为Terraform无法感知，无法将它执行的变更纳入到声明式的代码管理中，所以预置器应被作为最后的手段使用，那么也就是说，如果cloud-init能够满足我们的要求，那么我们应该优先使用cloud-init。

但是仍然存在一些cloud-init无法满足的场景。例如一个最常见的情况是，比如我们要在cloud-init当中格式化卷，后续的所有操作都必须在主机成功格式化并挂载卷之后才能顺利进行下去。但是比如`aws_instance`，它的创建是不会等待`user_data`代码执行完成的，只要虚拟机创建成功开始启动，Terraform就会认为资源创建完成从而继续后续的创建了。

## 2.1 预置器

解决这个问题目前来看还是只能依靠预置器。我们以一段UCloud云主机代码为例：

```json
resource "ucloud_instance" "web" {
  availability_zone         = "cn-bj2-03"
  image_id                  = data.ucloud_images.centos.images[0].id
  instance_type             = "n-standard-1"
  charge_type               = "dynamic"
  
  network_interface {
    eip_internet_type = "bgp"
    eip_charge_mode   = "traffic"
    eip_bandwidth     = 1
  }
  
  delete_eips_with_instance = true
  root_password             = var.root_password
  
  provisioner "remote-exec" {
    connection {
      type     = "ssh"
      host     = [for ipset in self.ip_set: ipset.ip if ipset.internet_type=="BGP"][0]
      user     = "root"
      password = var.root_password
      timeout  = "1h"
    }
    inline = [
      "sleep 1h"
    ]
  }
}
```

我们在资源声明中附加了一个`remote-exec`类型的预置器，它的`host`取值使用了`self.ip_set`，`self`在当前上下文中指代provisioner所属的`ucloud_instance.web`，`ip_set`是`ucloud_instance`的一个输出属性，内含云主机的内网IP以及绑定的弹性公网IP信息。

我们用一个`for`表达式过滤出弹性公网IP地址，然后使用ssh连接。预置器执行的脚本代码很简单，休眠一小时。如果我们执行这段代码：

```
$ terraform apply -auto-approve
data.ucloud_images.centos: Refreshing state...
ucloud_instance.web: Creating...
ucloud_instance.web: Still creating... [10s elapsed]
ucloud_instance.web: Still creating... [20s elapsed]
ucloud_instance.web: Provisioning with 'remote-exec'...
ucloud_instance.web (remote-exec): Connecting to remote host via SSH...
ucloud_instance.web (remote-exec):   Host: 106.75.87.148
ucloud_instance.web (remote-exec):   User: root
ucloud_instance.web (remote-exec):   Password: true
ucloud_instance.web (remote-exec):   Private key: false
ucloud_instance.web (remote-exec):   Certificate: false
ucloud_instance.web (remote-exec):   SSH Agent: true
ucloud_instance.web (remote-exec):   Checking Host Key: false
ucloud_instance.web: Still creating... [30s elapsed]
ucloud_instance.web (remote-exec): Connecting to remote host via SSH...
ucloud_instance.web (remote-exec):   Host: 106.75.87.148
ucloud_instance.web (remote-exec):   User: root
ucloud_instance.web (remote-exec):   Password: true
ucloud_instance.web (remote-exec):   Private key: false
ucloud_instance.web (remote-exec):   Certificate: false
ucloud_instance.web (remote-exec):   SSH Agent: true
ucloud_instance.web (remote-exec):   Checking Host Key: false
ucloud_instance.web: Still creating... [40s elapsed]
ucloud_instance.web (remote-exec): Connecting to remote host via SSH...
ucloud_instance.web (remote-exec):   Host: 106.75.87.148
ucloud_instance.web (remote-exec):   User: root
ucloud_instance.web (remote-exec):   Password: true
ucloud_instance.web (remote-exec):   Private key: false
ucloud_instance.web (remote-exec):   Certificate: false
ucloud_instance.web (remote-exec):   SSH Agent: true
ucloud_instance.web (remote-exec):   Checking Host Key: false
ucloud_instance.web (remote-exec): Connected!
ucloud_instance.web: Still creating... [50s elapsed]
ucloud_instance.web: Still creating... [1m0s elapsed]
ucloud_instance.web: Still creating... [1m10s elapsed]
ucloud_instance.web: Still creating... [1m20s elapsed]
ucloud_instance.web: Still creating... [1m30s elapsed]
ucloud_instance.web: Still creating... [1m40s elapsed]
...
```

不出所料的话，该过程会持续一小时，也就是说，无论预置器脚本中执行的操作耗时多长，`ucloud_instance`的创建都会等待它完成，或是触发超时。

## 2.2 null_resource

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


### 2.2.1 null_resource的triggers参数

另外这个例子里我们运行的脚本非常简单，考虑一种更加复杂一些的场景，我们运行的脚本是通过文件读取的，我们希望在文件内容发生变化时能够重新在服务器上运行该脚本，这时我们可以使用`null_resource`的`triggers`参数：

```json
resource "null_resource" "web_init" {
  depends_on = [
    ucloud_eip_association.eip_association,
    ucloud_disk_attachment.data_disk
  ]
  triggers = {
    script_hash = filemd5("${path.module}/init.sh")
  }
  provisioner "remote-exec" {
    connection {
      type     = "ssh"
      host     = ucloud_eip.eip.public_ip
      user     = "root"
      password = var.root_password
    }
    script = "${path.module}/init.sh"
  }
}
```

现在provisioner运行的脚本是通过`script`参数传入的脚本文件路径，而我们通过`filemd5`函数把文件内容的哈希值传入了`triggers`。`triggers`会在值发生改变时触发`null_resource`的重建，这样脚本发生些许变化都会导致重新执行。



官方文档上还给出了对于`triggers`的另一个妙用：

```json
resource "aws_instance" "cluster" {
  count = 3

  # ...
}

resource "null_resource" "cluster" {
  # Changes to any instance of the cluster requires re-provisioning
  triggers = {
    cluster_instance_ids = "${join(",", aws_instance.cluster.*.id)}"
  }

  # Bootstrap script can run on any instance of the cluster
  # So we just choose the first in this case
  connection {
    host = "${element(aws_instance.cluster.*.public_ip, 0)}"
  }

  provisioner "remote-exec" {
    # Bootstrap script called with private_ip of each node in the clutser
    inline = [
      "bootstrap-cluster.sh ${join(" ", aws_instance.cluster.*.private_ip)}",
    ]
  }
}
```

这个例子里，我们需要所有AWS主机的内网IP参与才能够成功初始化集群，可能是类似Kafka或是RabbitMQ这样的应用，我们需要把集群节点的IP写入配置文件。如何确保未来机器数量发生调整以后，机器上的配置文件始终能够获得完整的集群内网IP信息，这里使用`triggers`就可以轻松完成目标。

另外在绝大多数生产环境中，服务器都不允许拥有独立的公网IP，或是禁止从服务器对外服务的公网IP直接连接ssh。这时一般我们会在集群中配置一台堡垒机，通过堡垒机进行跳转连接。可以访问[通过堡垒机使用SSH的官方文档](https://www.terraform.io/docs/provisioners/connection.html#connecting-through-a-bastion-host-with-ssh)获取详细信息，在此不再赘述。

# 3 destroy-provisioner中使用变量

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

## 3.1 解决方法

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



