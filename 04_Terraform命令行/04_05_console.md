
# 1 console

有时我们想要一个安全的调试工具来帮助我们确认某个表达式是否合法，或者表达式的值是否符合预期，这时我们可以使用terraform console启动一个交互式控制台。

lets you experiment with Terraform's built-in functions

## 1.1 用法

terraform console [options] [dir]

console命令提供了一个用以执行和测试各种表达式的命令行控制台。在编码时如果我们不确定某个表达式的最终结果时(例如使用字符串模版)，我们可以在这个控制台中搭配当前状态文件中的数据进行各种测试。

如果当前状态是空的或还没有创建状态文件，那么控制台可以用来测试各种表达式语法以及内建函数。

dir参数指定了根模块的路径。如果没有指定path参数，那么则会使用当前工作目录。

支持的参数有：
- -state=path：指向本机状态文件的路径。表达式计算会使用该状态文件中记录的值。如果没有指定，则会使用当前工作区(Workspace)关联的状态文件

在控制台中可以使用exit命令或是Ctrl-C或是Ctrl-D退出。
\

输入 terraform console 出现 > 就是代表控制台已经打开了 
![](image/Pasted%20image%2020231118171908.png)

## 1.2 脚本化

terraform console命令可以搭配非交互式脚本使用，可以使用管道符将其他命令输出接入控制台执行。如果没有发生错误，只有最终结果会被打印。

样例：

```
$ echo "1 + 5" | terraform console
6
```

## 1.3 远程状态

如果使用了远程Backend存储状态，Terraform会从远程Backend读取当前工作区的状态数据来计算表达式。

