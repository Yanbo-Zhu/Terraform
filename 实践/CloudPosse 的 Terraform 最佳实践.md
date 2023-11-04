

# 1 CloudPosse 的 Terraform 最佳实践

[CloudPosse](https://cloudposse.com/)是一家在美国的提供 DevOps 咨询顾问服务的公司，他们的口号是:

- DevOps Accelerator for Startups
- Own your infrastructure. We build it. You drive it.

我注意到这家公司是因为他们向社区贡献了大量高品质的 Terraform Aws Module，其代码之工整与完善与其他社区模块完全不在一个水平。经过研究找到了他们总结的一篇 Terraform 的最佳实践，在此翻译为中文以飨读者。

不过话说在前，该文档可能编写的时间较早，目前我并不完全同意列出的所有规则，某几条可能因为 Terraform 自身的发展已经显得不再那么绝对，并且由于 CloudPosse 是一家咨询公司，难免有以开源代码为自己打广告招揽顾客的动机，所以其代码在某些程度上甚至有些“过于工程化”。这只是 CloudPosse 的一家之言，读者还是需要结合自身实际情况来分析。

## 1.1 语言

### 1.1.1 使用带缩进的 HEREDOC 语法

使用`<<-EOT`（与之相对的是`<<EOT`，没有`-`）来确保内联代码可以与项目中其他部分的代码共同锁进。注意，`EOT`可以用任意大写字符串代替（例如`CONFIG_FILE`）

```
block {
  value = <<-EOT
  hello
    world
  EOT
}
```

### 1.1.2 不要使用 HEREDOC 语法来编写 JSON、YAML或者 Aws IAM 策略代码

Terraform 对于这几种格式有更好的方法来格式化：

- 对于 JSON，请在`locals`块中使用`jsonencode`函数
- 对于 YAML，请在`locals`块中使用`yamlencode`函数
- 对于 Aws IAM 策略代码，请使用名为`aws_iam_policy_document`的 Datasource。

### 1.1.3 不要编写过长的 HEREDOC 代码

如果内容很长，请把配置内容转移到一个独立的文件中，然后使用名为`template_file`的 Datasource 读取。

### 1.1.4 使用 Terraform Linting

Linting 工具确保代码格式的统一，提升代码质量，并且可以检查一些常见的语法错误。

在提交所有代码之前运行`terraform fmt`。创建一个`pre-commit`钩子来自动化调用该命令。

### 1.1.5 使用合适的数据类型

在 Terraform 中使用合适的数据类型可以更容易地验证输入参数以及编写相关文档。

- 使用`null`而不是空字符串（""）
- 当要表达`true`/`false`时，使用`bool`类型而不是`string`或者`number`
- 使用`string`存储文本
- 不要滥用`object`类型，为`object`类型编写校验规则以及文档比较困难

### 1.1.6 使用 CIDR 相关函数来计算网络地址空间

这可以降低其他合作者贡献代码的门槛并降低人为疏失的空间。CloudPosse 编写了一些相关的 Terraform 模块来帮助用户计算子网的地址空间：

- [terraform-aws-dynamic-subnets](https://github.com/cloudposse/terraform-aws-dynamic-subnets)
- [terraform-aws-multi-az-subnets](https://github.com/cloudposse/terraform-aws-multi-az-subnets)
- [terraform-aws-named-subnets](https://github.com/cloudposse/terraform-aws-named-subnets)

更多相关信息请阅读[官方文档](https://www.terraform.io/docs/configuration/interpolation.html#cidrsubnet-iprange-newbits-netnum-)。

### 1.1.7 在所有项目仓库中使用.editorconfig文件规定一致的空格风格

所有主流的 IDE 都有插件支持`.editorconfig`文件，使得我们可以强制实施一致的空格风格

我们推荐针对特定语言或项目规定空格风格。

CloudPosse 使用的标准`.editorconfig`文件内容如下：

```
# Override for Makefile
[{Makefile, makefile, GNUmakefile, Makefile.*}]
indent_style = tab
indent_size = 4

[*.yaml]
intent_style = space
indent_size = 2

[*.sh]
indent_style = tab
indent_size = 4

[*.{tf,tfvars,tpl}]
indent_size = 2
indent_style = space
```

### 1.1.8 锁定所使用的 Provider 的最低版本

Terraform 的 Provider 保持着持续的更新，在编写模块时很难确定模块代码是否能够在更早的 Provider 版本上正确工作，并且这种测试旧版本 Provider 的努力通常不值得。由此我们希望对外通告我们所测试过的最低的 Provider 版本。

当然未来的 Provider 版本也有可能引入破坏性更新，但在 CloudPosse 的实践中这并不太会发生。另外，对于 CloudPosse 发布的模块代码，他们无法测试限制了 Provider 最高版本后会引入什么样的问题。所以对于 CloudPosse 发布的模块，他们只会锁定 Provider 的最低版本。

在用户编写的根模块中，用户可以锁定 Provider 的最高版本，或是锁定使用指定版本的 Provider 来避免发生意外。这是一个在稳定性与易用性之间进行的权衡，用户必须按照自身情况进行决策。

### 1.1.9 使用locals改善可读性

使用`locals`使得代码更加声明式以及可维护。与其在某些 Terraform 资源代码参数中使用复杂的表达式，不如将该表达式封装成一个`local`，然后在声明资源时引用它。

## 1.2 输入参数

### 1.2.1 在合适的时候使用上游模块或 Provider 的变量名

当编写一个接受输入参数的模块时，确保参数名与上游模块的`output`名一致以防止误解以及二义性。

### 1.2.2 变量名使用小写字母，以下划线作为分隔符

避免使用其他语言的语法规则，例如驼峰命名。所有变量的命名要统一，要遵循 [HashiCorp 命名规范](https://www.terraform.io/docs/extend/best-practices/naming.html)。

### 1.2.3 使用肯定的变量名以避免双重否定

所有用来代表打开或者关闭某项设置的输入变量名都应该以`...._enabled`结尾（例如：`encryption_enabled`）。变量的默认值可以是`false`也可以是`true`。

### 1.2.4 使用特性开关来配置打开或关闭某项功能

所有模块都应该通过特性开关来设置打开或是关闭某项功能。所有特性开关都应该以`_enabled`结尾，数据类型必须为`bool`。

### 1.2.5 所有输入参数都应该声明description值

所有输入参数都需要声明`descripition`值。当该参数源自于另一个上游 Provider（例如：`terraform-aws-provider`），请完整照搬上游 Provider 文档中的字句。

### 1.2.6 在合适的时候定义合理的默认值

模块应尽量开箱即用。默认值应尽可能确保整体配置的安全性（例如：`encryption_enabled`为`true`）。

### 1.2.7 所有传递机密的输入变量不可定义默认值

所有用来传递机密的输入变量都不应该定义默认值，这可以确保 Terraform 可以校验用户的输入。唯一的例外是该机密是可选的，并且在用户输入`null`或是`""`（空字符串）时会自动生成一个。

## 1.3 输出值

### 1.3.1 所有的输出值都应该声明description值

所有输输出值都需要声明`descripition`值。尽可能照搬上游 Provider 中对应参数的`description`。避免在输出值的`description`中简单地重复输入参数名。

### 1.3.2 使用合规的蛇式命名法命名输出参数

避免使用其他语言的语法规则，例如驼峰命名。所有输出值的命名要统一，要遵循 [HashiCorp 命名规范](https://www.terraform.io/docs/extend/best-practices/naming.html)。

### 1.3.3 永远不要输出机密信息

模块永远不应该输出机密，相应的，机密信息应该被写入安全的存储，例如 AWS Secrets Manager，AWS SSM Parameter Store（由 KMS 加密），或是 S3 存储中（由 KMS 加密）。CloudPosse 更倾向于使用 AWS SSM Parameter Store。写入 SSM 的信息可以很容易地被其他 Terraform 模块读取，或是其他诸如`chamber`的命令行工具使用。

我们在编写根模块时严格执行该规定，因为这些机密信息很容易被泄漏到 CI/CD 流水线中。对于那些内嵌在其他模块中的子模块，我们的规定不会那么严格。

与其输出机密，我们可以输出一段文本指示机密存储的位置，例如创建 RDS 数据时，我们把管理员密码保存在路径为`/rds/master_password`的 SSM 存储中。我们可能还需要另一个输出值来保存该机密存储的密钥，这样其他需要读取管理员密码的程序可以使用该密钥读取到密码。

### 1.3.4 命名要对称

CloudPosse 喜欢确保 Terraform 输出值的名字尽可能与上游资源或模块对称，可以添加前缀。这能减少代码中的混乱或是二义性，同时提升一致性。下面是一个**反面**例子。输出值的名字应该是`user_secret_access_key`，这是因为它的取值来自于另一个模块的输出值`secret_access_key`，模块名含有`user`，所以可以添加前缀`user_`，最终处于一致性，输出值的名称应该是`user_secret_access_key`

![Terraform 输出值的命名要对称](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2021-11-28/1638086002974-image.png)

图 1.8/1 - Terraform 输出值的命名要对称

## 1.4 状态

### 1.4.1 使用远程状态存储

### 1.4.2 使用 Terraform 创建用以存储远程状态的存储桶

这需要一个两阶段步骤来实施，第一阶段我们使用本地状态文件来创建一个远程存储桶。第二阶段我们启用远程状态存储配置（例如使用`s3 {}`）并且将本地状态导入远程存储（添加相关配置文件后简单执行`terraform init`即可自动导入）。CloudPosse 推荐这种策略因为它可以使用最好的工具来简化工作以及使用一致的工具。

可以使用`terraform-aws-tfstate-backend`模块来简化创建状态存储桶的工作。

### 1.4.3 使用支持状态锁的远程存储

CloudPosse 推荐使用 S3 存储状态的同时使用 DynamoDB 提供状态锁控制。

提示：使用`terraform-aws-tfstate-backend`模块可以轻松完成这一目标。

[官方文档 https://www.terraform.io/docs/backends/types/s3.html](https://www.terraform.io/docs/backends/types/s3.html)

### 1.4.4 严格锁定使用的 Terraform CLI 版本

Terraform 状态文件有时在不同版本的 CLI 之间是不兼容的。CloudPosse 推荐开发人员通过容器使用 Terraform CLI 以锁定使用的版本。

提示：使用[geodesic(一款 CloudPosse 出品的开源工具)](https://github.com/cloudposse/geodesic)管理所有的 Terraform 交互。

### 1.4.5 使用 Terraform CLI 设置状态存储参数

为提升根模块在不同账号之间的可重用性，应避免硬编码状态存储参数。相应的，应使用 Terraform CLI 设置当前使用的参数。

### 1.4.6 不要锁定使用的 Terraform 的最高版本

```json
terraform {
  required_version = ">= 0.12.26"

  backend "s3" {}
}
```

Terraform 大多数情况下都能保持向前兼容，所以我们希望可以在未来使用新版本来测试现有的模块代码。所以请不要限制使用的 Terraform 的最高版本，例如`~>0.12.26`或是`>=0.13, <0.15`这样都会阻止未来的 Terraform 新版本运行当前模块。应使用`>=`限制最低版本即可。

### 1.4.7 使用加密的 S3 存储桶并开启版本控制、加密存储以及严格的 IAM 访问控制策略

CloudPosse 不推荐用一个存储桶存储不同栈的状态文件，这有可能会导致状态文件被错误覆盖或是泄漏。注意，状态文件中包含了所有输出值的内容。尽可能确保 100% 的物理隔离（每个 Stage 拥有独立的存储桶，独立的账号）

提示：使用`terraform-aws-tfstate-backend`可以轻松地为每一个 Stage 创建独立的状态存储桶。

### 1.4.8 启用状态存储桶的版本控制

### 1.4.9 启用状态存储桶的静态加密存储（Encryption at Rest）

### 1.4.10 使用.gitignore排除 Terraform 状态文件、状态文件备份、Terraform 文件夹等

```
.terraform
.terraform.tfstate.lock.info
*.tfstate
*.tfstate.backup
```

### 1.4.11 使用.dockerignore文件排除 Terraform 状态文件

样例：

```
**/.terraform*
```

## 1.5 命名规范

### 1.5.1 使用一致的编程命名规范

所有资源名（比如：在 AWS 上创建的那些资源）必须遵循一个一致的命名规范，这点之所以重要的原因是模块经常被用以组装成其他模块。强制实施一致的命名规范可以降低模块与其他模块创建的资源在名字上发生冲突的概率。

为了确保一致性，CloudPosse 要求所有模块都要调用`terraform-null-label`模块。使用该模块，用户可以通过修改参数的顺序或是分隔符的方式来修改生成资源名的方式。虽然该模块的使用不是必须的，但事实证明该机制是一种解决命名冲突非常有效的方法。

## 1.6 DNS 基础设施

### 1.6.1 使用独立的 DNS Zone

不要在不同 Stage 和环境之间混用 DNS Zone。

### 1.6.2 为每个 AWS 账户委派一个独享 DNS 区域

### 1.6.3 区分品牌域名与服务发现域名的管理

服务发现域名是指用来提供服务发现服务的域名。终端用户极少会直接使用这种域名。应该只有一个服务发现域名，但不同环境下使用各自独立的 DNS Zone 来管理该域名的解析。

品牌域名是终端用户用来访问服务所使用的域名，这些域名由产品、市场和业务场景来决定。可以有多个不同的品牌域名指向同一个服务发现域名。品牌域名的架构并非是服务发现域名架构的镜像。

## 1.7 模块的设计

### 1.7.1 小而精的模块

CloudePosse 认为一个模块应该把一件事做到最好。为了达到这个目标，简单地把 Terraform 资源打包成模块化代码并没有什么用。为这些资源设计一种专门的使用场景则更为有用。（译者理解：每一个模块都应有一个特定的使用场景，例如创建 Subnet ，CloudPosse 就编写了`dynamic-subnet`、`named-subnets`、`multi-az-subnets`三种模块

### 1.7.2 可组合的模块

模块应编写成易于与其他模块进行组合，这是 CloudPosse 用以提升规模经济性以及停止重新发明模式轮子大的方法。

### 1.7.3 使用输入参数

模块应尽可能使用输入参数。应避免定义类型为`object`的输入参数，因为该类型很难编写文档（译者理解：很难为`object`类型输入参数编写详尽的`description`提示调用者）。当然，这并不是一个绝对的禁令，有时候使用`object`的确更加合适。需要注意类似[`terraform-docs`](https://github.com/segmentio/terraform-docs)这样的工具能否生成有意义的文档。

## 1.8 模块的使用

### 1.8.1 使用 Terraform registry 格式锁定指定的模块版本

Terraform 模块的`source`参数有多种表达方式。CloudPosse 的传统是使用 Terraform registry 语法显式锁定一个确切的版本：

```json
  source  = "cloudposse/label/null"
  version = "0.22.0"
```

显式锁定确切版本的原因是因为，使用例如`>=0.22.0`这样可升级的版本约束可能会引入破坏性变更。对基础设施的所有变更都应该是由最终用户来控制和审查的，不能在部署变更时盲目信任变更结果。

（译者理解：由于模块版本变化带来的变更并非模块调用者所触发的，亦非模块调用者所能控制的，故应避免这种情况的发生。）

