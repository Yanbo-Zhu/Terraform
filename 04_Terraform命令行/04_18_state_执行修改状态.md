
# 1 state

terraform state命令可以用来进行复杂的状态管理操作。随着你对Terraform的使用越来越深入，有时候你需要对状态文件进行一些修改。由于我们在状态管理章节中提到过的，状态文件的格式属于HashiCorp未公开的私有格式，所以直接修改状态文件是不适合的，我们可以使用terraform state命令来执行修改。

该命令含有数个子命令，我们会一一介绍。

## 1.1 用法

terraform state \ [options] [args]

## 1.2 远程状态

所有的state子命令都可以搭配本地状态文件以及远程状态使用。使用远程状态时读写操作可能用时稍长，因为读写都要通过网络完成。备份文件仍然会被写入本地磁盘。

## 1.3 备份

所有会修改状态文件的terraform state子命令都会生成备份文件。可以通过-backup参数指定备份文件的位置。

只读子命令(例如list)由于不会修改状态，所以不会生成备份文件。

注意修改状态的state子命令无法禁用备份。由于状态文件的敏感性，Terraform强制所有修改状态的子命令都必须生成备份文件。如果你不想保存备份，可以手动删除。

## 1.4 命令行友好

state子命令的输出以及命令结构都被设计成易于同Unix下其他命令行工具搭配使用，例如grep、awk等等。同样的，输出结果也可以在Windows上轻松使用PowerShell处理。

对于复杂场景，我们建议使用管道组合state子命令与其他命令行工具一同使用。

## 1.5 资源地址

state子命令中大量使用了资源地址，我们在资源地址章节中做了相关的介绍。

# 2 terraform state list

terraform state list命令可以列出状态文件中记录的资源对象。

## 2.1 用法

terraform state list [options] [address...]

该命令会根据address列出状态文件中相关资源的信息(如果给定了address的话)。如果没有给定address，那么所有资源都会被列出。

列出的资源根据模块深度以及字典序进行排序，这意味着根模块的资源在前，越深的子模块定义的资源越在后。

对于复杂的基础设施，状态文件可能包含成千上万到的资源对象。可以通过提供一个或多个资源地址来进行过滤。

可以使用的可选参数有：

- -state=path：指定使用的状态文件地址。默认为"terraform.tfstate"。使用远程Backend时该参数设置无效
- -id=id：要显示的资源ID

## 2.2 例子：所有资源

```
$ terraform state list
aws_instance.foo
aws_instance.bar[0]
aws_instance.bar[1]
module.elb.aws_elb.main
```

## 2.3 例子：根据资源地址过滤

```
$ terraform state list aws_instance.bar
aws_instance.bar[0]
aws_instance.bar[1]
```

## 2.4 例子：根据模块过滤

```
$ terraform state list module.elb
module.elb.aws_elb.main
```

## 2.5 例子：根据ID过滤

下面的例子显示了根据资源对象ID过滤资源：

```
$ terraform state list -id=sg-1234abcd
module.elb.aws_security_group.sg
```



# 3 terraform state mv

terraform state mv命令可以在状态文件中移动资源。该命令可以移动单个资源对象、多实例资源对象中特定实例、整个模块以及其他对象。
该命令也可以在不同的状态文件之间移动对象，以配合代码重构。

## 3.1 用法

terraform state mv [options] SOURCE DESTINATION

该命令将会把资源对象从SOURCE地址移动到DESTINATION地址。这可以用来实现单个简单资源的重命名、在模块之间移动对象、移动整个模块等操作。它也可以用来在Terraform管理的不同基础设施栈之间移动对象。

该命令在进行任意修改之前会先生成一个备份文件。备份机制不可关闭。如果是在不同状态文件之间移动对象，那么每个状态文件都会生成一个备份。

该命令的SOURCE与DESTINATION是必填参数，必须是合法的资源地址。

该命令提供以下可选参数：

- -backup=path：指定源状态文件的备份地址，默认为源状态文件加上".backup"后缀
- -bakcup-out=path：指定目标状态文件的备份地址，默认为目标状态文件加上".backup"后缀
- -state=path：源状态文件地址，默认为当前Backend或是"terraform.tfstate"
- -state-out=path：目标状态文件地址。如果不指定则使用源状态文件。可以是一个已经存在的文件或新建一个文件

## 3.2 例子：重命名一个资源

```
$ terraform state mv 'packet_device.worker' 'packet_device.helper'
```

## 3.3 例子：将一个资源移动进一个模块

以下例子展示了将packet_device.worker资源移动进名为app的模块。如果模块目前不存在，则会创建模块。

```
$ terraform state mv 'packet_device.worker' 'module.app.packet_device.worker'
```

## 3.4 例子：移动一个模块进入另一个模块

```
$ terraform state mv 'module.app' 'module.parent.module.app'
```

## 3.5 例子：移动一个模块到另一个状态文件

```
$ terraform state mv -state-out=other.tfstate 'module.app' 'module.app'
```

## 3.6 移动一个带有count参数的资源

```
$ terraform state mv 'packet_device.worker[0]' 'packet_device.helper[0]'
```

## 3.7 移动一个带有for_each参数的资源

Linux、MacOS以及Unix：

```
$ terraform state mv 'packet_device.worker["example123"]' 'packet_device.helper["example456"]'
```

PowerShell：

```
$ terraform state mv 'packet_device.worker[\"example123\"]' 'packet_device.helper[\"example456\"]'
```

Windows命令行：

```
$ terraform state mv packet_device.worker[\"example123\"] packet_device.helper[\"example456\"]
```



# 4 terraform state pull 从远程Backend中人工下载状态

terraform state pull命令可以从远程Backend中人工下载状态并输出。该命令也可搭配本地状态文件使用。

## 4.1 用法

terraform state pull

该命令下载当前位置对应的状态文件，并以原始格式打印到标准输出流。
由于状态文件使用JSON格式，该功能可以搭配例如jq这样的命令行工具使用，也可以用来人工修改状态文件。


# 5 terraform state push 上传本地状态文件到远程Backend

terraform push命令被用来手动上传本地状态文件到远程Backend。该命令也可以被用在当前使用的本地状态文件上。

## 5.1 用法

terraform state push [options] PATH

该命令会把PATH位置的状态文件推送到当前使用的Backend上(可以是当前使用的terraform.tfstate文件)。

如果PATH的值为"-"那么会从标准输入流读取要推送的状态数据。这时数据会完全被加载进内存，并且在写入目标状态前进行检查。

Terraform会进行一系列检查以防止你进行一些不安全的变更：

- 检查lineage：如果两个状态文件的lineage值不同，Terraform会禁止推送。一个不同的lineage说明两个状态文件描述的是完全不同的基础设而你可能会因此丢失重要数据
- 序列号检查：如果目标状态文件的serial值大于你要推送的状态的serial值，Terraform会禁止推送。一个更高的serial值说明目标状态文件已经无法与要推送的状态文件对应上了

这两种检查都可以通过添加-force参数禁用，但**不推荐这样做**。如果禁用安全检查直接推送，那么目标状态文件将被覆盖。


# 6 terraform state replace-provider

terraform state replace-provider命令可以替换状态文件中资源对象所使用的Provider的源.

## 6.1 用法

terraform state replace-provider [options] FROM_PROVIDER_FQN TO_PROVIDER_FQN

该命令会更新所有使用from Provider的资源，将资源使用的Provider更新为to Provider。这允许我们更新状态文件中资源所使用的Provider的源。

该命令在进行任意修改之前会先生成一个备份文件。备份机制不可关闭。

支持以下可选参数：
- -auto-approve：跳过交互式提示确认环节
- -backup=path：将备份写入指定路径。如果没有该参数，则会写入当前状态文件加上“.backup"后缀的路径
- -lock=true：类似apply，不再赘述
- -lock-timeout=0s：类似apply，不再赘述
- -state=path：要读取的状态文件地址。默认为当前使用的Backend或是"terraform.tfstate"文件

## 6.2 样例

```
$ terraform state replace-provider hashicorp/aws registry.acme.corp/acme/aws
```

# 7 terraform state rm 可以用来从状态文件中删除对象

terraform state rm命令可以用来从状态文件中删除对象。该命令可以删除单个资源、多实例资源中特定实例、整个模块以及等等。

## 7.1 用法

terraform state rm [options] ADDRESS...

从状态文件中删除一个或多个对象。

==删除对象并非删除实际基础设施对象，而只是状态不再由Terraform管理，从状态文件中删除而已。==   举例来说，如果你从状态文件中删除一个AWS虚拟机，那么该虚拟机仍然会保持运行，只是运行terraform plan再也看不到该实例了。

从状态文件中删除对象有着广泛的用途。最常见的就是进行代码重构，不再管理某个资源(也许是转移到另一个项目管理了)。

只有当所有要删除的资源被删除成功状态文件才会保存。如果由于任何原因导致某个资源删除发生错误(比如语法错误)，那么状态文件完全不会被修改。

该命令在进行任意修改之前会先生成一个备份文件。备份机制不可关闭。

该命令需要传递一个或多个待删除资源的资源地址。

可以使用如下可选参数：
- -backup=path：写入备份文件的路径
- -state=path：要操作的资源文件路径。如果没有该参数，则会使用当前Backend或是"terraform.tfstate"文件

### 7.1.1 删除一个资源

```
$ terraform state rm 'packet_device.worker'
```

### 7.1.2 删除一个模块

```
$ terraform state rm 'module.foo'
```

### 7.1.3 删除一个模块内资源

```
$ terraform state rm 'module.foo.packet_device.worker'
```

### 7.1.4 删除一个声明count的资源

```
$ terraform state rm 'packet_device.worker[0]'
```

### 7.1.5 删除一个声明for_each的资源

Linux, MacOS, and Unix：
```
$ terraform state rm 'packet_device.worker["example"]'
```

PowerShell：
```
$ terraform state rm 'packet_device.worker[\"example\"]'
```

Windows命令行：
```
$ terraform state rm packet_device.worker[\"example\"]
```



# 8 terraform state show_展示状态文件中单个资源的属性

terraform state show命令可以展示状态文件中单个资源的属性。

### 8.1.1 用法

terraform state show [options] ADDRESS

该命令需要指定一个资源地址。

该命令支持以下可选参数：
- -state=path：指向状态文件的路径。默认情况下使用"terraform.tfstate"。如果启用了远程Backend则该参数设置无效

terraform state show 的输出被设计成人类可读而非机器可读。如果想要从输出中提取数据，请使用 terraform show -json

### 8.1.2 展示单个资源

```
$ terraform state show 'packet_device.worker'
# packet_device.worker:
resource "packet_device" "worker" {
    billing_cycle = "hourly"
    created       = "2015-12-17T00:06:56Z"
    facility      = "ewr1"
    hostname      = "prod-xyz01"
    id            = "6015bg2b-b8c4-4925-aad2-f0671d5d3b13"
    locked        = false
}
```

### 8.1.3 展示单个模块资源

```
$ terraform state show 'module.foo.packet_device.worker'
```

### 8.1.4 展示声明count资源中特定实例

```
$ terraform state show 'packet_device.worker[0]'
```

### 8.1.5 展示声明for_each资源中特定实例

Linux, MacOS, and Unix：

```
$ terraform state show 'packet_device.worker["example"]'
```

PowerShell：

```
$ terraform state show 'packet_device.worker[\"example\"]'
```

Windows命令行：

```
$ terraform state show packet_device.worker[\"example\"]
```

