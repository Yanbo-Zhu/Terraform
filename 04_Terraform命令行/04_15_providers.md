

# 1 providers

terraform providers命令打印当前代码定义的Provider的信息。

Provider依赖关系通过几种不同方式创建：

- 在代码中显式使用terraform.required_providers块，包含可选的version约束
- 在代码中显式使用provider块，包含可选的version约束
- 在当前状态文件中存在属于某个Provider管理的资源实例。例如，如果代码中删除了特定资源对象的声明，那么仍然存在对相应Provider实例的依赖，直到对象被销毁

该命令总结了当前所有的Provider依赖，有助于理解为什么会需要特定Provider。

该命令是含有内嵌子命令，这些子命令我们会逐个解释。

## 1.1 用法

terraform providers [config-path]

可以通过显式传递config-path参数来指定根模块路径，默认为当前工作目录。


# 2 terraform providers mirror

该子命令从Terraform 0.13开始引入。

terraform providers mirror命令下载当前代码所需要的Provider并且将其拷贝到本地文件系统的一个目录下。

一般情况下，terraform init会在初始化当前工作目录时自动从registry下载所需的Provider。有时Terraform工作在无法执行该操作的环境下，例如一个无法访问registry的局域网内。这时可以通过配置Provider镜像存储来使得在这样的环境下Terraform可以从本地插件镜像存储中获取插件。

terraform providers mirror命令可以自动填充准备用以作为本地插件镜像存储的目录。

## 2.1 用法

terraform providers mirror [options] \

target-dir参数是必填的。Terraform会自动在目标目录下建立起插件镜像存储所需的文件结构，填充包含插件文件的.zip文件。

Terraform同时会生成一些包含了合法的[网络镜像协议](https://www.terraform.io/docs/internals/provider-network-mirror-protocol.html)响应的.json索引文件，如果你把填充好的文件夹上传到一个静态站点，那就能够得到一个静态的网络插件镜像存储服务。Terraform在使用本地文件镜像存储时会忽略这些镜像文件，因为使用本地文件镜像时文件夹本身的信息更加权威。

该命令支持如下可选参数：

- -platform=OS_ARCH：选择构建镜像的目标平台。默认情况下，Terraform会使用当前运行Terraform的平台。可以反复声明该参数以构建多目标平台插件镜像

目标平台必须包含操作系统以及CPU架构。例如：linux_amd64代表运行在AMD64或是X86_64 CPU之上的Linux系统。

你可以针对已构建的镜像文件夹重新运行terraform providers mirror来添加新插件。例如，你可以通过传入新的-platform参数来添加新目标平台的插件，Terraform会下载新平台插件同时保留原先的插件，将二者合并存储，并更新索引文件。


# 3 terraform providers schema

terraform providers schema命令被用来打印当前代码使用的Provider的架构。Provider架构包含了该Provider本身的参数信息，以及所提供的resource、data的架构信息。

## 3.1 用法

terraform providers schema [options]

可选参数为：
- -json：用机器可读的JSON格式打印架构

请注意，目前-json参数是必填的，未来该命令将允许使用其他参数。

