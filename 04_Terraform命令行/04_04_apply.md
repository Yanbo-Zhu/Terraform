

# 1 apply

Terraform最重要的命令就是apply。apply命令被用来生成执行计划(可选)并执行之，使得基础设施资源状态符合代码的描述。

## 1.1 用法

terraform apply [options] [dir-or-plan]

默认情况下，apply会扫描当前目录下的代码文件，并执行相应的变更。然而，也可以通过参数指定其他代码文件目录。在设计自动化流水线时也可以显式分为创建执行计划、使用apply命令执行该执行计划两个独立步骤。

如果没有显式指定变更计划文件，那么terraform apply会自动创建一个新的变更计划，并提示用户是否批准执行。如果生成的计划不包含任何变更，那么terraform apply会立即退出，不会提示用户输入。

该命令有以下参数可以使用：

- -backup-path：保存备份文件的路径。默认等于-state-out参数后加上".backup"后缀。设置为"-"可关闭
- -compact-warnings：如果Terraform生成了一些告警信息而没有伴随的错误信息，那么以只显示消息总结的精简形式展示告警
- -lock=true：执行时是否先锁定状态文件
- -lock-timeout=0s：尝试重新获取状态锁的间隔
- -input=true：在无法获取输入变量的值是是否提示用户输入
- -auto-approve：跳过交互确认步骤，直接执行变更
- -no-color：禁用输出中的颜色
- -parallelism=n：限制Terraform[遍历图](https://www.terraform.io/docs/internals/graph.html#walking-the-graph)时的最大并行度，默认值为10(考试高频考点)
- -refresh=true：指定变更计划及执行变更前是否先查询记录的基础设施对象现在的状态以刷新状态文件。如果命令行指定了要执行的变更计划文件，该参数设置无效
- -state=path：保存状态文件的路径，默认值是"terraform.tfstate"。如果使用了远程Backend该参数设置无效。该参数不影响其他命令，比如执行init时会找不到它设置的状态文件。如果要使得所有命令都可以使用同一个特定位置的状态文件，请使用Local Backend(译者也不知道这是什么，官方文档相关链接404，并且搜索不到)
- -state-out=path：写入更新的状态文件的路径，默认情况使用-state的值。该参数在使用远程Backend时设置无效
- -target=resource：通过指定资源地址指定更新目标资源。我们会在随后介绍plan命令时详细介绍
- -var 'foo=bar'：设置一组输入变量的值。该参数可以反复设置以传入多个输入变量值
    - terraform apply -var "instance_name=YetAnotherName"
- -var-file=foo：指定一个输入变量文件。具体内容我们在介绍输入变量的章节已有介绍，在此不再赘述

常用的是 
terraform apply -auto-approve

