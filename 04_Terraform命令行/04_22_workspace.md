

1.6.22.1. workspace

terraform workspace命令可以用来管理当前使用的工作区。我们在状态管理章节中介绍过工作区的概念。

该命令包含一系列子命令，我们将会一一介绍。

terraform workspace -help
Usage: terraform [global options] workspace
new, list, show, select and delete Terraform workspaces.

Subcommands:
delete Delete a workspace
list List Workspaces
new Create a new workspace
select Select a workspace
show Show the name of the current workspace
# 1 list

terraform workspace list命令列出当前存在的工作区。

## 1.1 用法

terraform workspace list

该命令会打印出存在的工作区。当前工作会使用*号标记：

```
$ terraform workspace list
  default
* development
  jsmith-test
```

# 2 select

terraform workspace select命令用来选择使用的工作区。

## 2.1 用法

terraform workspace select [NAME]

NAME指定的工作区必须已经存在：

```
$ terraform workspace list
  default
* development
  jsmith-test

$ terraform workspace select default
Switched to workspace "default".
```


# 3 new

terraform workspace new命令用来创建新的工作区。

## 3.1 用法

terraform workspace new [NAME]

该命令使用给定名字创建一个新的工作区。不可存在同名工作区。

如果使用了-state参数，那么给定路径的状态文件会被拷贝到新工作区。

该命令支持以下可选参数：
- -state=path：用来初始化新环境所使用的状态文件路径

创建新工作区：

```
$ terraform workspace new example
Created and switched to workspace "example"!

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```

使用状态文件创建新工作区：

```
$ terraform workspace new -state=old.terraform.tfstate example
Created and switched to workspace "example".

You're now on a new, empty workspace. Workspaces isolate their state,
so if you run "terraform plan" Terraform will not see any existing state
for this configuration.
```


# 4 delete

terraform workspace delete命令被用以删除已经存在的工作区。

## 4.1 用法

terraform workspace delete [NAME]

被删除的工作区必须已经存在，并且**不可以删除当前正在使用的工作区**。如果工作区状态不是空的，Terraform会禁止删除，除非声明-force参数。

如果使用-force删除非空状态，那么这些资源饿状态出于"dangling"，*也就是实际基础设施资源仍然存在，但脱离了Terraform的管理*。有时我们希望这样，只是希望当前Terraform项目不再管理这些资源，交由其他项目管理。但大多数情况下并非这样，所以Terraform默认会禁止删除非空工作区。

该命令可以使用如下可选参数：
- -force：删除含有非空状态文件的工作区。默认为false

例子：
```
$ terraform workspace delete example
Deleted workspace "example".
```


# 5 show

terraform workspace show命令被用以输出当前使用的工作区。

## 5.1 用法

terraform workspace show

例子：

```
$ terraform workspace show
development
```

