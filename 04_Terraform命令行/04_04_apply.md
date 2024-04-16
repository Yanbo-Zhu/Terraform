
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
- -destroy: 只破坏 不重建 


plan v reads the current state of any already-existing remote objects to MAKE SURE that the Terraform state is up-to-date.
- -refresh-onlf - creates a plan whose goal is ONLY to update the Terraform state and any root module output values to match changes made to remote objects outside of Terraform.
- -refresh=false - disables the DEFAULT behavior of synchronizing the Terraform state with remote objects before checking for configuration changes.


常用的是 
terraform apply -auto-approve
terraform apply -destroy

# 2 Terraform apply 到底在干什么

A is correct answer : Validates your expectations against the execution plan without permanently modifying state
"The terraform plan command creates an execution plan, which lets you preview the changes that Terraform plans to make to your infrastructure.

Terraform's plan command is used to create an execution plan that outlines the steps that Terraform will take to reach your desired infrastructure state. It allows you to preview and validate the changes that will be made to your infrastructure before actually making those changes. This can be helpful in the development process because it allows you to see exactly what will be changed and ensure that it aligns with your expectations before you apply those changes.

By default, when Terraform creates a plan it:
- Reads the current state of any already-existing remote objects to make sure that the Terraform state is up-to-date.
- Compares the current configuration to the prior state and noting any differences.
- Proposes a set of change actions that should, if applied, make the remote objects match the configuration.
- The plan command alone will not actually carry out the proposed changes, and so you can use this command to check whether the proposed changes match what you expected before you apply the changes or share your changes with your team for broader review.

If Terraform detects that no changes are needed to resource instances or to root module output values, terraform plan will report that no actions
need to be taken."

# 3 When does terraform apply reflect changes in the cloud environment

1. When you execute terraform apply, Terraform creates a new execution plan by comparing the current state file to the desired state declared in the configuration. 
    1. 比较current state in local state 和 真实的state in cloud. 创造 execution plan
2. After creating the execution plan, Terraform presents the proposed changes and asks for confirmation to apply them. 
3. Once you confirm the changes, Terraform updates the state file with the new state reflecting the changes that were made. 
    1. 更新 local state file 
4. Terraform then submits the chang requests to the resource provider to make the desired changes in the cloud environment. The amount of time it takes for the resource provider to fulfill the requests can vary depending on the resources being modified.
    1.  向 cloud 端 提出 更改云资源的申请


When you apply this configuration, Terraform will:
1) Lock your project's state
2) Create a plan, and wait for you to approve it.
3) Execute the steps defined in the plan using the providers you installed when you initialized your configuration. Terraform executes steps in parallel when possible, and sequentially when one resource depends on another.
4) Update your project's state file with a snapshot of the current state of your resources.
5) Unlock the state file.
6) Print out a report of the changes it made, as well as any output values defined in your configuration.



# 4 手动删除resource 后, 再次运行terraform apply 

D. Terraform will recreate the VM

When you delete a resource like a VM directly through the cloud provider's console (outside of Terraform), the Terraform state file still believes the resource exists, as it's unaware of any changes made outside its management. The next time you run terraform apply, Terraform compares the desired state (defined in your Terraform configuration) with the actual state (as recorded in the state file and observed in the cloud environment).

Since the actual VM no longer exists but your Terraform configuration still defines it, Terraform detects this discrepancy and takes action to reconcile the difference by creating a new VM to match the desired state defined in your Terraform configuration. Terraform's goal is always to make the real-world infrastructure match the configuration.


- It refresh the state, which detects that the storage account was gone
- Then it re-creates storage account with the same name but without the data from previous instance




