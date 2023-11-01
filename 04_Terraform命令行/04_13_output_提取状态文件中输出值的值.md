

# 1 output

terraform output命令被用来提取状态文件中输出值的值。

## 1.1 用法

terraform output [options] [NAME]

如果不添加参数，output命令会展示根模块内定义的所有输出值。如果指定了NAME，只会输出相关输出值。

可以使用以下参数：

- -json：使用该参数后Terraform会使用JSON格式输出。如果指定了NAME，只会输出相关输出值。该参数搭配jq使用可以构建复杂的流水线
- -no-color：不输出颜色
- -state=path：状态文件的路径，默认为"terraform.tfstate"。启用远程Backend时该参数无效

## 1.2 样例

假设有如下输出值代码：

```
output "lb_address" {
  value = aws_alb.web.public_dns
}

output "instance_ips" {
  value = [aws_instance.web[*].public_ip]
}

output "password" {
  sensitive = true
  value = [var.secret_password]
}
```

列出所有输出值：

```
$ terraform output
```

注意password输出值定义了sensitive = true，所以它的值在输出时会被隐藏：

```
$ terraform output password
password = <sensitive>
```

要查询负载均衡的DNS地址：

```
$ terraform output lb_address
my-app-alb-1657023003.us-east-1.elb.amazonaws.com
```

查询所有主机的IP：

```
$ terraform output instance_ips
test = [
    54.43.114.12,
    52.122.13.4,
    52.4.116.53
]
```

使用-json和jq查询指定主机的ip：

```
$ terraform output -json instance_ips | jq '.value[0]'
```

