
# 1 graph

terraform graph命令可以用来生成代码描述的基础设施或是执行计划的可视化图形。它的输出是DOT格式，可以使用[GraphViz](http://www.graphviz.org/)来生成图片，也有许多网络服务可以读取这种格式。

## 1.1 用法

terraform graph [options] [DIR]

该命令生成DIR路径下的代码锁描述的Terraform资源的可视化依赖图(如果DIR参数缺省则使用当前工作目录)

-type参数被用来指定输出的图表的类型。Terraform为不同的操作创建不同的图。对于代码文件，默认类型为"plan"，对于变更计划文件，默认类型为"apply"。

参数：
- -draw-cycles：用彩色的边高亮图中的环，这可以帮助我们分析代码中的环错误(Terraform禁止环状依赖)
- -type=plan：生成图表的类型。可以是：plan、plan-destroy、apply、validate、input、refresh

## 1.2 创建图片文件

terraform graph命令输出的是DOT格式的数据，可以轻松地使用GraphViz转换为图形文件：

```
$ terraform graph | dot -Tsvg > graph.svg
```

输出的图片大概是这样的：

![生成的依赖图](https://raw.githubusercontent.com/lonegunmanb/introduction-to-terraform-pic/master/2020-11-25/1606272727693-image.png)

图 1.6.10/1 - 生成的依赖图

## 1.3 如何安装GraphViz

安装GraphViz也很简单，对于Ubuntu：

```
$ sudo apt install graphviz
```

对于CentOS：

```
$ sudo yum install graphviz
```

对于Windows，也可以使用choco：

```
> choco install graphviz
```

对于Mac用户：

```
$ brew install graphviz
```

