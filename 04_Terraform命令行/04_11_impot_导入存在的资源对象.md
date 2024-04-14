
# 1 import

import  the exisiting infrastructure in your .tfstate file 

terraform import命令用来将已经存在的资源对象导入Terraform。

我们并不总是那么幸运，能够在项目一开始就使用Terraform来构建和管理我们的基础设施；有时我们有一组已经运行着的基础设施资源，然后我们为它们编写了相应的Terraform代码，我们进行了测试，确认了这组代码描述的基础设施与当前正在使用的基础设施是等价的；但是我们仍然无法直接使用这套代码来管理现有的基础设施，因为我们缺乏了相应的状态文件。这时我们需要使用terraform import将资源对象“导入”到Terraform状态文件中去。

## 1.1 用法

terraform import [options] ADDRESS ID

terraform import会根据资源ID找到相应资源，并将其信息导入到状态文件中ADDRESS对应的资源上。ADDRESS必须符合我们在资源地址中描述的合法资源地址格式，这样terraform import不但可以把资源导入到根模块中，也可以导入到子模块中。

ID取决于被导入的资源对象的类型。举例来说，AWS主机的ID格式类似于i-abcd1234，而AWS Route53 Zone的ID类似于Z12ABC4UGMOZ2N，请参考相关Provider文档来获取有关ID的详细信息。如果不确信的话，可以随便尝试任意ID。如果ID不合法，你会收到一个报错。

需要尤其注意的是，Terraform设想的是每一个资源对象都仅对应一个独一无二的实际基础设施对象，通常来说如果我们完全使用Terraform创建并管理基础设施时这一点不是问题；但如果你是通过导入的方式把基础设施对象导入到Terraform里，要绝对避免将同一个对象导入到两个以及更多不同的地址上，这会导致Terraform产生不可预测的行为。

该命令有以下参数可以使用：
- -backup=path：生成状态备份文件的地址，默认情况下是-state-out路径加上".backup"后缀名。设置为"-"可以关闭备份(不推荐)
- -config=path：包含含有导入目标的Terraform代码的文件夹路径。默认为当前工作目录
- -input=true：是否允许提示输入Provider配置信息
- -lock=true：如果Backend支持，是否锁定状态文件
- -lock-timeout=0s：重试获取状态锁的间隔
- -no-color：如果指定，则不会输出彩色信息
- -parallelism=n：限制Terraform遍历图的最大并行度，默认值为10(又是考点)
- -state=path：要读取的状态文件的地址。默认为配置的Backend存储地址，或是"terraform.tfstate"文件
- -state-out=path：指定修改后的状态文件的保存路径，默认情况下覆盖源状态文件。使用该参数可以生成一个新的状态文件，避免破坏现有状态文件
- -var 'foo=bar'：通过命令行设置输入变量值，类似apply命令中的介绍
- -var-file=foo：类似apply命令中的介绍

## 1.2 Provider配置

Terraform会尝试读取要导入的资源对应的Provider的配置信息。如果找不到相关Provider的配置，那么Terraform会提示你输入相关的访问凭据。你也可以通过环境变量来配置访问凭据。

Terraform在读取Provider配置时唯一的限制是不能依赖于"非输入变量"的输入。举例来说，Provider配置不能依赖于数据源的输出。

举一个例子，如果你想要导入AWS资源而你有这样的一份代码文件，那么Terraform会使用这两个输入变量来配置AWS Provier：

```
variable "access_key" {}
variable "secret_key" {}

provider "aws" {
  access_key = var.access_key
  secret_key = var.secret_key
}
```

## 1.3 例子

```
$ terraform import aws_instance.foo i-abcd1234
```

![](image/Pasted%20image%2020231119163543.png)
后面的 为 aws instance 的 id 

```
$ terraform import module.foo.aws_instance.bar i-abcd1234
```

```
$ terraform import 'aws_instance.baz[0]' i-abcd1234
```

```
$ terraform import 'aws_instance.baz["example"]' i-abcd1234
```

上面这条命令如果是在PowerShell下：

```
$ terraform import 'aws_instance.baz[\"example\"]' i-abcd1234
```

如果是cmd：

```
$ terraform import aws_instance.baz[\"example\"] i-abcd1234
```


# 2 terraform import  只输入state

terraform import  只输入state, 还需要自己 手写 resource configuration block 

"The current implementation of Terraform import can only import resources into the state. It does not generate configuration. A future version of Terraform will also generate configuration.
Because of this, prior to running terraform import it is necessary to write manually a resource configuration block for the resource, to which the imported object will be mapped.


Import will find the existing resource from ID and import it into your Terraform state at the given ADDRESS.
- ADDRESS must be a valid resource address. Because any resource address is valid, the import command can import resources into modules as well as directly into the root of your state.
- ID is dependent on the resource type being imported. If the ID is invalid, you'll just receive an error message.

Warning: Terraform expects that each remote object it is managing will be bound to only one resource address, which is normally guaranteed b Terraform itself having created all objects. If you import existing objects into Terraform, be careful to import each remote object to only one Terraform resource address. If you import the same object multiple times, Terraform may exhibit unwanted behavior.

# 3 example 2 

You have provisioned some virtual machines (VMs) on Google Cloud Platform (GCP) using the gcloud command line tool. However, you are
standardizing with Terraform and want to manage these VMS using Terraform instead.
What are the two things you must do to achieve this? (Choose two.)

B. Use the terraform import command for the existing VMS
C. Write Terraform configuration for the existing VMS


B. Use the terraform import command for the existing VMs. This command allows you to import existing infrastructure into your Terraform state fil
so that Terraform can manage it. You will need to provide the resource type and name, along with any required attributes, for each VM you want
import.
C. Write Terraform configuration for the existing VMs. Once the VMS have been imported into the Terraform state file, you will need to write
configuration code that describes the desired state of the VMs. This will typically involve creating a new Terraform module or modifying an existing
one to include the imported resources.

