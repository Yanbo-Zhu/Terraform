
# 1 untaint

terraform untaint命令可以手动清除一个Terraform管理的资源对象上的污点，恢复它在状态文件中的状态。它是terraform taint的逆向操作。

该命令不会修改实际的基础设施资源，只会在资源文件中清除资源对象上的污点标记。

## 1.1 用法

terraform untaint [options] name

name参数是要清除污点的资源的资源名称。该参数的格式为TYPE.NAME，比如aws_instance.foo。

可以使用如下可选参数：

- -allow-missing：如果声明该参数，那么即使name指定的资源不存在，命令执行也会返回成功(状态码0)。命令执行仍然可能报错，但只在发生严重错误时
- -backup=path：写入备份文件的路径。默认为-state-out路径加上".backup"后缀。设置为"-"可关闭备份
- -lock=true：类似apply，不再赘述
- -lock-timeout=0s：类似apply，不再赘述
- -module=path：资源所在模块的名称。默认情况下使用根模块。可以通过"."号分隔的路径指定模块，例如"foo"指定使用foo模块，"foo.bar"指定使用foo模块中的bar模块
- -no-color：类似apply，不再赘述
- -state=path：读写的状态文件路径。默认为"terraform.tfstate"。使用远程Backend时该参数设置无效
- -state-out=path：修改后的状态文件的写入路径。默认情况下使用-state参数。使用远程Backend时该参数设置无效
