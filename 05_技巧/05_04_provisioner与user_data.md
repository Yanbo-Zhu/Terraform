
# 1 provisioner与user_data

我们在介绍资源时介绍了预置器provisioner。同时不少公有云厂商的虚拟机都提供了cloud-init功能，可以让我们在虚拟机实例第一次启动时执行一段自定义的脚本来执行一些初始化操作。例如我们在"Terraform初步体验"一章里举的例子，在UCloud主机第一次启动时我们通过user_data来调用yum安装并配置了ngnix服务。预置器与cloud-init都可以用于初始化虚拟机，那么我们应该用哪一种呢？

首先要指出的是，provisioner的官方文档里明确指出，由于预置器内部的行为Terraform无法感知，无法将它执行的变更纳入到声明式的代码管理中，所以预置器应被作为最后的手段使用，那么也就是说，如果cloud-init能够满足我们的要求，那么我们应该优先使用cloud-init。

但是仍然存在一些cloud-init无法满足的场景。例如一个最常见的情况是，比如我们要在cloud-init当中格式化卷，后续的所有操作都必须在主机成功格式化并挂载卷之后才能顺利进行下去。但是比如`aws_instance`，它的创建是不会等待`user_data`代码执行完成的，只要虚拟机创建成功开始启动，Terraform就会认为资源创建完成从而继续后续的创建了。

## 1.1 预置器

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


### 1.2.1 null_resource的triggers参数

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



