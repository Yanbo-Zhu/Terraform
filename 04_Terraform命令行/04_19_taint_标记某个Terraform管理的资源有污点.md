
# 1 taint

terrform taint命令可以手动标记某个Terraform管理的资源有"污点"，强迫在下一次执行apply时删除并重建之。
该命令并不会修改基础设施，而是在状态文件中的某个资源对象上标记污点。当一个资源对象被标记了污点，在下一次plan操作时会计划将之删除并且重建，apply操作会执行这个变更。
强迫重建某个资源可以使你能够触发某种副作用。举例来说，你想重新执行某个预置器操作，或是某些人绕过Terraform修改了虚拟机状态，而你想将虚拟机重置。
注意为某个资源标记污点并重建之会影响到所有依赖该资源的对象。举例来说，一条DNS记录使用了服务器的IP地址，我们在服务器上标记污点会导致IP发生变化从而影响到DNS记录。这种情况下可以使用plan命令查看变更计划。

terraform taint is already deprecated. 
Please use 
terraform apply -replace="null_resource.run_script"
Reference: https://developer.hashicorp.com/terraform/cli/commands/taint

## 1.1 用法

terraform taint [options] ADDRESS

ADDRESS参数是要标记污点的资源地址。

该命令可以使用如下可选参数：

- -allow-missing：如果声明该参数，那么即使资源不存在，命令也会返回成功(状态码0)
- -backup=path：写入备份文件的路径，默认为-state-out路径加上".backup"后缀。可以通过设置为"-"关闭备份
- -lock=true：与apply类似，不再赘述 
- -lock-timeout=0s：与apply类似，不再赘述 
- -state=path：要读写的状态文件路径。默认为"terraform.tfstate"。如果启用了远程Backend  则该参数设置无效
- -state-out=path：更新后的状态文件写入的路径。默认等于-state路径。如果启用了远程Backend  则该参数设置无效

## 1.2 标记单个资源

```
$ terraform taint aws_security_group.allow_all
The resource aws_security_group.allow_all in the module root has been marked as tainted.
```

## 1.3 标记使用for_each创建的资源的特定实例

```
$ terraform taint "module.route_tables.azurerm_route_table.rt[\"DefaultSubnet\"]"
The resource module.route_tables.azurerm_route_table.rt["DefaultSubnet"] in the module root has been marked as tainted.
```

## 1.4 标记模块中的资源

```
$ terraform taint "module.couchbase.aws_instance.cb_node[9]"
Resource instance module.couchbase.aws_instance.cb_node[9] has been marked as tainted.
```

虽然我们推荐模块深度不要超过1，但是我们仍然可以标记多层模块中的资源：

```
$ terraform taint "module.child.module.grandchild.aws_instance.example[2]"
Resource instance module.child.module.grandchild.aws_instance.example[2] has been marked as tainted.
```
