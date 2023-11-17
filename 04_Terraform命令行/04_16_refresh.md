
# 1 refresh

terraform refresh命令将实际存在的基础设施对象的状态同步到状态文件中记录的对象状态。它可以用来检测真实状态与记录状态之间的漂移并更新状态文件。

> 警告！！！该命令已在最新版本 Terraform 中被废弃，因为该命令的默认行为在当前用户错误配置了使用的云平台令牌时会引发对状态文件错误的变更。

该命令并不会修改基础设施对象，只修改状态文件。如果状态文件发生改变，将有可能在下次执行`plan`或`apply`时引发变更计划。

## 1.1 用法

terraform refresh [options] [dir]

该命令本质上是以下命令的别名，具有完全相同的效果：

> terraform apply -refresh-only -auto-approve

因此，该命令支持所有 `terraform apply` 所支持的参数，除了它不接受一个现存的变更计划文件，不允许选择"refresh only"之外的模式，并且始终应用`-auto-approve`选项。

主动使用`refresh`是很危险的，因为如果当前用户错误配置了使用的 Provider 的令牌，那么 Terraform 会错误地以为当前状态文件中记录的所有资源都被删除了，随即从状态文件中无预警地删除所有相关记录。

作为替代我们推荐使用如下命令来取得相同的效果，同时可以在修改状态文件之前预览即将对其作出的修改：

> terraform apply -refresh-only

该命令将会在交互界面中提示用户检测到的变更，并提示用户确认执行。

`terraform apply`和`terraform plan`命令的`-refresh-only`选项是从 Terraform v0.15.4 版本开始被引入的。对更早的版本，用户只能直接使用`terraform refresh`命令，同时要小心本篇警告过的危险副作用。尽可能避免显式使用`terraform refresh`命令，Terraform 在执行`terraform plan`和`terraform apply`命令时都会自动执行刷新状态的操作以生成变更计划，尽可能依赖该机制来维持状态文件的同步。
