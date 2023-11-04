
# 1 validate

terraform validate命令可以检查目录下Terraform代码，只检查语法文件，不会访问诸如远程Backend、Provider的API等远程资源。

validate检查代码的语法是否合法以及一致，不管输入变量以及现存状态。

validate命令需要已初始化的工作目录，所有引用的插件与模块都被安装完毕。如果只想检查语法而不想与Backend交互，可以这样初始化工作目录：

```
$ terraform init -backend=false
```

## 1.1 用法

terraform validate [options] [dir]

默认情况下validate命令不需要任何参数就可以在当前工作目录下进行检查。

可以使用如下可选参数：
- -json：使用JSON格式输出机器可读的结果
- -no-color：禁止使用彩色输出

