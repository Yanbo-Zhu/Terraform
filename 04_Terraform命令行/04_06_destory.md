

# 1 destroy

terraform destroy命令可以用来销毁并回收所有Terraform管理的基础设施资源。

## 1.1 用法

terraform destroy [options] [dir]

Terraform管理的资源会被销毁，在执行销毁动作前会通过交互式界面征求用户的确认。

该命令可以接收所有apply命令的参数，除了不可以指定plan文件。

如果-auto-approve参数被设置为true，那么将不会征求用户确认直接销毁。

如果用-target参数指定了某项资源，那么不但会销毁该资源，同时也会销毁一切依赖于该资源的资源。我们会在随后介绍plan命令时详细介绍。

==terraform destroy将执行的所有操作都可以随时通过执行terraform plan -destroy命令来预览。==



