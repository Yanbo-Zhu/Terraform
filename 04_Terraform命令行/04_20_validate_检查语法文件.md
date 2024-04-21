
# 1 validate

==You must initialize your working directory before running terraform validate.==

==checks that your configuration syntax is correct.==


==The terraform validate command validates the configuration files in a directory, referring only to the configuration and NOT accessing any remote services such as remote state, provider APIs, etc. terraform validate does not uses provider APIs to verify your infrastructure settings.==

The terraform validate command checks the syntax and validates the configuration files in a Terraform module. It does not check for configuration consistency errors like differences between local and remote state or outdated module versions.

Before running the terraform validate command, it is recommended to initialize your working directory using the terraform init command. The Terraform init command initializes the working directory and downloads the necessary providers and modules specified in your Terraform configuration. After initialization, you can use Terraform validate to check the syntax and validity of your Terraform files.

terraform validate命令可以检查目录下Terraform代码，只检查语法文件，不会访问诸如远程Backend、Provider的API等远程资源。

validate检查代码的语法是否合法以及一致，不管输入变量以及现存状态。

validate命令需要已初始化的工作目录，所有引用的插件与模块都被安装完毕。如果只想检查语法而不想与Backend交互，可以这样初始化工作目录：

```
$ terraform init -backend=false
```

---

http://man.hubwiz.com/docset/Terraform.docset/Contents/Resources/Documents/docs/commands/validate.html
The terraform validate command is used to validate the syntax of the terraform files. Terraform performs a syntax check on all the
terraform files in the directory, and will display an error if any of the files doesn't validate.
This command does not check formatting (e.g. tabs vs spaces, newlines, comments etc.).

The following can be reported:
• invalid HCL syntax (e.g. missing trailing quote or equal sign)
• invalid HCL references (e.g. variable name or attribute which doesn't exist)
• same provider declared multiple times
• same module declared multiple times
• same resource declared multiple times
• invalid module name
• interpolation used in places where it's unsupported (e.g. variable , depends _ on, module. source , provider )
• missing value for a variable (none of -var foo=. flag, -var-file=foo.vars flag, TF VAR_foo environment variable,
terraform. tfvars , or default value in the configuration)


## 1.1 用法

terraform validate [options] [dir]

默认情况下validate命令不需要任何参数就可以在当前工作目录下进行检查。

可以使用如下可选参数：
- -json：使用JSON格式输出机器可读的结果
- -no-color：禁止使用彩色输出

