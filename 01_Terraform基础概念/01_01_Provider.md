

Terraform被设计成一个多云基础设施编排工具，不像CloudFormation那样绑定AWS平台，Terraform可以同时编排各种云平台或是其他基础设施的资源。Terraform实现多云编排的方法就是Provider插件机制。

![Terraform通过RPC调用插件，插件代码通过调用SDK操作远程资源](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-16/1605496195438-image.png)

图 1.3.1/1 - Terraform通过RPC调用插件，插件代码通过调用SDK操作远程资源

Terraform使用的是HashiCorp自研的go-plugin库([https://github.com/hashicorp/go-plugin](https://github.com/hashicorp/go-plugin))，本质上各个Provider插件都是独立的进程，与Terraform进程之间通过rpc进行调用。Terraform引擎首先读取并分析用户编写的Terraform代码，形成一个由data与resource组成的图(Graph)，再通过rpc调用这些data与resource所对应的Provider插件；Provider插件的编写者根据Terraform所制定的插件框架来定义各种data和resource，并实现相应的CRUD方法；在实现这些CRUD方法时，可以调用目标平台提供的SDK，或是直接通过调用Http(s) API来操作目标平台。


# 1 下载Provider


我们在第一章的小例子中，写完代码后在apply之前，首先我们执行了一次`terraform init`。`terraform init`会分析代码中所使用到的Provider，并尝试下载Provider插件到本地。如果我们观察执行完第一章例子的文件夹，我们会发现有一个`.terraform`文件夹，我们所使用的UCloud Provider插件就被下载安装在里面。

```
.terraform
└── plugins
    ├── registry.terraform.io
    │   └── ucloud
    │       └── ucloud
    │           └── 1.22.0
    │               └── darwin_amd64 -> /Users/byers/.terraform.d/plugin-cache/registry.terraform.io/ucloud/ucloud/1.22.0/darwin_amd64
    └── selections.json
```

有的时候下载某些Provider会非常缓慢，或是在开发环境中存在许多的Terraform项目，每个项目都保有自己独立的插件文件夹非常浪费磁盘，这时我们可以使用插件缓存。



有两种方式可以启用插件缓存：

**第一种方法**是配置`TF_PLUGIN_CACHE_DIR`这个环境变量：
```
export TF_PLUGIN_CACHE_DIR="$HOME/.terraform.d/plugin-cache"
```

**第二种方法**是使用CLI配置文件。Windows下是在相关用户的%APPDATA%目录下创建名为"terraform.rc"的文件，Macos和Linux用户则是在用户的home下创建名为".terraformrc"的文件。在文件中配置如下：
```
plugin_cache_dir = "$HOME/.terraform.d/plugin-cache"
```

当启用插件缓存之后，每当执行`terraform init`命令时，Terraform引擎会首先检查期望使用的插件在缓存文件夹中是否已经存在，如果存在，那么就会将缓存的插件拷贝到当前工作目录下的`.terraform`文件夹内。如果插件不存在，那么Terraform仍然会像之前那样下载插件，并首先保存在插件文件夹中，随后再从插件文件夹拷贝到当前工作目录下的`.terraform`文件夹内。为了尽量避免同一份插件被保存多次，只要操作系统提供支持，Terraform就会使用符号连接而不是实际从插件缓存目录拷贝到工作目录。

**需要特别注意的是，Windows 系统下`plugin_cache_dir`的路径也必须使用`/`作为分隔符，应使用`C:/somefolder/plugin_cahce`而不是`C:\somefolder\plugin_cache`**

Terrafom引擎永远不会主动删除缓存文件夹中的插件，缓存文件夹的尺寸可能会随着时间而增长到非常大，这时需要手工清理。


# 2 搜索Provider


想要了解有哪些被官方接纳的Provider，有两种方法：

**第一种方法**是访问[Terraform 官方 Provider 文档](https://www.terraform.io/docs/providers/index.html)，该页面中列出了主流的Provider：

![terraform.io上的插件页面](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-16/1605498841554-image.png)

图 1.3.1/2 - terraform.io上的插件页面

![terraform.io上的主流云厂商插件](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-19/1605799271127-image.png)

图 1.3.1/3 - terraform.io上的主流云厂商插件



**第二种方法**就是前往[registry.terraform.io](https://registry.terraform.io/browse/providers)进行搜索：

![registry.terraform.io上的插件页面](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-19/1605799350775-image.png)

图 1.3.1/4 - registry.terraform.io上的插件页面

目前推荐在registry搜索Provider，因为大量由社区开发的Provider都被注册在了那里。

![registry.terraform.io的搜索页面](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-16/1605498907942-image.png)

图 1.3.1/5 - registry.terraform.io的搜索页面

![优刻得插件主页](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-16/1605498945181-image.png)

图 1.3.1/6 - 优刻得插件主页

![优刻得插件文档](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-16/1605499003904-image.png)

图 1.3.1/7 - 优刻得插件文档

一般来说，相关Provider如何声明，以及相关data、resource的使用说明，都可以在registry上查阅到相关文档。
registry.terraform.io不但可以查询Provider，也可以用来发布Provider；并且它也可以用来查询和发布模块(Module)，不过模块将是我们后续篇章讨论的话题。


# 3 Provider的声明

The `provider` block configures the specified provider, in this case `aws`. A provider is a plugin that Terraform uses to create and manage your resources.

You can use multiple provider blocks in your Terraform configuration to manage resources from different providers. You can even use different providers together. For example, you could pass the IP address of your AWS EC2 instance to a monitoring resource from DataDog.

一组Terraform代码要被执行，相关的Provider必须在代码中被声明。不少的Provider在声明时需要传入一些关键信息才能被使用，例如我们在第一章的例子中，必须给出访问密钥以及期望执行的UCloud区域（Region）信息。

```
terraform {
  required_providers {
    ucloud    = {
      source  = "ucloud/ucloud"
      version = ">=1.24.1"
    }
  }
}

provider "ucloud" {
  public_key  = "your_public_key"
  private_key = "your_private_key"
  project_id  = "your_project_id"
  region      = "cn-bj2"
}
```

在这段Provider声明中，首先在terraform节的`required_providers`里声明了本段代码必须要名为`ucloud`的Provider才可以执行，`source = "ucloud/ucloud"`这一行声明了ucloud这个插件的源地址(Source Address)。一个源地址是全球唯一的，它指示了Terraform如何下载该插件。



一个源地址由三部分组成：

```
[<HOSTNAME>/]<NAMESPACE>/<TYPE>
```

`HostName`是选填的，默认是官方的 `registry.terraform.io`，读者也可以构建自己私有的Terraform仓库。`Namespace`是在Terraform仓库内得到组织名，这代表了发布和维护插件的组织或是个人。`Type`是代表插件的一个短名，在特定的`HostName`/`Namespace`下`Type`必须唯一。

`required_providers`中的插件声明还声明了该源码所需要的插件的版本约束，在例子里就是`version = ">=1.24.1"`。Terraform插件的版本号采用MAJOR.MINOR.PATCH的语义化格式，版本约束通常使用操作符和版本号表达约束条件，条件之间可以用逗号拼接，表达AND关联，例如">= 1.2.0, < 2.0.0"。可以采用的操作符有：

- =(或者不加=，直接使用版本号)：只允许特定版本号，不允许与其他条件合并使用
- !=：不允许特定版本号
- >,>=,<,<=：与特定版本号进行比较，可以是大于、大于等于、小于、小于等于
- ~>：锁定MAJOR与MINOR，允许PATCH号大于等于特定版本号，例如，~>0.9等价于>=0.9, <1.0，\~>0.8.4等价于>=0.8.4, <0.9

Terraform会检查当前工作环境或是插件缓存中是否存在满足版本约束的插件，如果不存在，那么Terraform会尝试下载。如果Terraform无法获得任何满足版本约束条件的插件，那么它会拒绝继续执行任何后续操作。
可以用添加后缀的方式来声明预览版，例如：`1.2.0-beta`。预览版只能通过"="操作符(或是空缺操作符)后接明确的版本号的方式来指定，不可以与`>=`、`~>`等搭配使用。
推荐使用">="操作符约束最低版本。如果你是在编写旨在由他人复用的模块代码时，请避免使用"~>"操作符，即使你知道模块代码与新版本插件会有不兼容。


# 4 内建Provider

绝大多数Provider是以插件形式单独分发的，但是目前有一个Provider是内建于Terraform主进程中的，那就是`terraform_remote_state` data source。该Provider由于是内建的，所以使用时不需要在terraform中声明`required_providers`。这个内建Provider的源地址是`terraform.io/builtin/terraform`。




# 5 多Provider实例

`provider`节声明了`ucloud`这个Provider所需要的各项配置。在上文的代码示例中，`provider "ucloud"`和`required_providers`中`ucloud = {...}`块里的`ucloud`，都是Provider的Local Name，一个Local Name是在一个模块中对一个Provider的唯一的标识。

我们也可以声明多个同类型的Provider，并给予不同的Local Name：

```
terraform {
  required_version = ">=0.13.5"
  required_providers {
    ucloudbj  = {
      source  = "ucloud/ucloud"
      version = ">=1.24.1"
    }
    ucloudsh  = {
      source  = "ucloud/ucloud"
      version = ">=1.24.1"
    }
  }
}

provider "ucloudbj" {
  public_key  = "your_public_key"
  private_key = "your_private_key"
  project_id  = "your_project_id"
  region      = "cn-bj2"
}

provider "ucloudsh" {
  public_key  = "your_public_key"
  private_key = "your_private_key"
  project_id  = "your_project_id"
  region      = "cn-sh2"
}

data "ucloud_security_groups" "default" {
  provider = ucloudbj
  type     = "recommend_web"
}

data "ucloud_images" "default" {
  provider          = ucloudsh
  availability_zone = "cn-sh2-01"
  name_regex        = "^CentOS 6.5 64"
  image_type        = "base"
}
```

例如上面的例子，我们声明了两个UCloud Provider，分别定位在北京区域和上海区域。我们在接下来的data声明中显式指定了provider的Local Name，这使得我们可以在一组配置文件中同时操作不同区域、不同账号的资源。

我们也可以使用alias别名来区隔同类Provider的不同实例：

```
terraform {
  required_version = ">=0.13.5"
  required_providers {
    ucloud    = {
      source  = "ucloud/ucloud"
      version = ">=1.24.1"
    }
  }
}

provider "ucloud" {
  public_key  = "your_public_key"
  private_key = "your_private_key"
  project_id  = "your_project_id"
  region      = "cn-bj2"
}

provider "ucloud" {
  alias       = "ucloudsh"
  public_key  = "your_public_key"
  private_key = "your_private_key"
  project_id  = "your_project_id"
  region      = "cn-sh2"
}

data "ucloud_security_groups" "default" {
  type = "recommend_web"
}

data "ucloud_images" "default" {
  provider          = ucloud.ucloudsh
  availability_zone = "cn-sh2-01"
  name_regex        = "^CentOS 6.5 64"
  image_type        = "base"
}
```

和多Local Name相比，使用别名允许我们区分 provider 的不同实例。`terraform`节的`required_providers`中只声明了一次`ucloud`，并且在data中指定provider时传入的是`ucloud.ucloudsh`。多实例Provider请使用别名。

每一个**不带**alias属性的provider声明都是一个**默认**provider声明。没有显式指定provider的data以及resource都使用默认资源名第一个单词所对应的provider，例如，`ucloud_images`这个data对应的默认provider就是`ucloud`，`aws_instance`这个resource对应的默认provider就是`aws`。

假如代码中所有显式声明的provider都有别名，那么Terraform运行时会构造一个所有配置均为空值的默认provider。假如provider有必填字段，并且又有资源使用了默认provider，那么Terraform会抛出一个错误，抱怨默认provider缺失了必填字段。



