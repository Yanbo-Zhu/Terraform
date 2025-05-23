https://developer.hashicorp.com/terraform/cloud-docs/workspaces
https://developer.hashicorp.com/terraform/language/state/workspaces
# 1 总览 

Each Terraform configuration has an associated backend that defines how Terraform executes operations and where Terraform stores persistent data, like state.

The persistent data stored in the backend belongs to a workspace. The backend initially has only one workspace containing one Terraform state associated with that configuration. Some backends support multiple named workspaces, allowing multiple states to be associated with a single configuration. ==The configuration still has only one backend, but you can deploy multiple distinct instances of that configuration without configuring a new backend or changing authentication credentials.==

Terraform工作空间（workspace）是一种用于管理多个环境的机制。它允许用户在同一个Terraform配置文件的基础上，创建多个环境，并使用不同的变量值和资源配置。

# 2 workspace state files在哪里 

In which folder are the custom workspace state files are stored ?
terraform
terraform/workspace
terraform.d
terraform.d
terraform.tfstate.d

Correct: terraform.tfstate.d


# 3 Workspace and the repo 

A workspace can only be configured to a single VCS repo, however, multiple workspaces can
use the same repo, if needed.

# 4 Purpose of a Terraform workspace

A. Workspaces allow you to manage collections of infrastructure in state files.
The purpose of a Terraform workspace, both in open source and enterprise, is to allow users to manage multiple "instances" of infrastructure within the same configuration codebase. Each workspace is a separate instance of a Terraform state file that can be managed independently. This means that a single Terraform configuration can be used to manage multiple sets of resources, with each set of resources represented by a separate workspace.
Workspaces are useful when there is a need to manage multiple versions of the same infrastructure in the same configuration codebase, such as different environments (dev, staging, prod) or different regions. By creating a separate workspace for each instance of infrastructure, users can manage them independently without causing conflicts or overwriting state.

There is no discussion of business units in the documentation so (A) seems to make the most sense, based on these two sources both of which
mention state :
Terraform CLI workspaces are associated with a specific working directory and isolate multiple state files in the same working directory, letting you manage multiple groups of resources with a single configuration.

The persistent data stored in the backend belongs to a workspace. The backend initially has only one workspace containing one Terraform state
associated with that configuration. Some backends support multiple named workspaces, allowing multiple states to be associated with a single
configuration. The configuration still has only one backend, but you can deploy multiple distinct instances of that configuration without configuring a new backend or changing authentication credentials.

# 5 the difference between terraform CLI workspace and cloud workspace


Terraform开源版的Workspace与Terraform Cloud云服务的Workspace 实际上是不同的概念
- Terraform开源版的Workspace
- Terraform Cloud云服务的Workspace 
- Terraform Enterprise Workspace 

While Terraform Cloud workspaces facilitate executing runs, storing, and versioning state files in a shared, cloud-based environment, CLI workspaces offer a simpler approach for managing different states of infrastructure in less complex scenarios


Workspaces, managed with the terraform workspace command, isn't the same thing as Terraform Cloud's workspaces. Terraform Cloud workspaces act more like completely separate working directories. ==CLI workspaces (OSS) are just alternate state files.==


# 6 Operation 

创建工作空间
使用terraform workspace new命令创建一个新的工作空间。例如，创建一个名为dev的工作空间：terraform workspace new dev


切换工作空间
使用terraform workspace select命令切换到另一个工作空间。例如，切换到名为prod的工作空间： terraform workspace select prod  如果该工作空间不存在，Terraform会自动创建一个新的工作空间。

配置变量
在每个工作空间中，可以使用不同的变量值来配置资源。可以在terraform.tfvars文件中为每个工作空间设置不同的变量值，也可以在命令行中使用-var参数设置变量值。


部署资源
使用terraform apply命令部署资源。Terraform会根据当前选择的工作空间，使用相应的变量值和资源配置来创建或更新基础设施。


# 7 例子 

https://blog.csdn.net/dongshi_89757/article/details/127904243
## 7.1 方案

当Terraform部署资源时，它将元数据存储在状态文件中以跟踪资源。因此，如果我们希望为多个环境(如DEV/QA/PROD)部署资源，我们还需要区分部署操作和状态文件。然而，通过terrform工作区Workspace和Reusable Modules （可重用模块）体系结构，我们可以实现单独的环境供应。

    Terraform Workspace——Terraform Workspace允许我们从单个配置文件源为所提供的后端中每个工作区管理单独的状态文件
    Reusable Modules ——在这种体系结构中，配置文件存储在单个目录中。通过共有模块，我们可以查找到对应目录并传递变量。

## 7.2 案例

公司使用Terraform在多个环境（INT,Staging, Prod）上部署Elastic Cloud，用作系统及客户日志存储分析等 。结合 Terraform Workspace 我们可以实现在不同环境中使用同个Terraform Module 来动态部署资源。例如，在INT环境中部署的 ES 节点使用更小的内存，分布在较少的Zone（以节省开支）， 而在Prod环境中需要启用更多的节点，来提高ES性能以及大量数据存储。

结论

我们看到了Workspace和reusable modules(可重用模块)如何帮助我们使用独立的状态文件管理独立的基础设施。Reusable modules可重用模块在env下各自的目录架构中运行，包含各个模块bolck、变量、提供程序和后端配置等。其中工作区工作在单个配置文件源上，并在后端创建工作区特定的状态文件。


## 7.3 具体实现

整体目录结构

-- terraform
|   |-- envs
|   |   |-- us-production
    |   |-- staging
    |   |-- qa
    |   |-- dev
|   |-- modules
|   |   |-- ec_deployment
    |   |-- ec_backup
    |   |-- ec_index_magement



首先，来看下 Reusable Modules (可重用模块), 放置在terraform/modules目录下
ec_deployment module 实现了整个ElastcCloud 部署代码

 modules
    |-- ec_deployment
    |   |-- README.md
    |   |-- data.tf
    |   |-- locals.tf
    |   |-- main.tf
    |   |-- outputs.tf
    |   |-- provider.tf
    |   |-- variables.tf


其中main.tf 为主代码实现elasticsearch主体结构，语法具体参考官网
```

resource "ec_deployment" "main" {
  name                   = var.deployment_name
  region                 = var.deployment_region
  version                = var.deployment_stack_version
  deployment_template_id = var.deployment_template_id
  alias                  = var.deployment_alias != "" ? var.deployment_alias : ""

  tags = var.tags

  elasticsearch {
    autoscale = "false"

    config {
      plugins            = var.elasticsearch_plugins
    }


    topology {
      id            = "hot_content"
      zone_count    = var.elasticsearch_hot_zones
      size          = var.elasticsearch_hot_size
      size_resource = "memory"
    }

  }

  kibana {
    topology {
      zone_count    = var.kibana_zones
      size          = var.kibana_size
      size_resource = "memory"
    }
  }
```



variables.tf 放置main.tf所引用的变量

```

variable "deployment_name" {
  type        = string
  description = "Identify deployment within ElasticCloud managed service"
}

variable "deployment_region" {
  type        = string
  description = "Select one of the available regions, https://www.elastic.co/guide/en/cloud/current/ec-reference-regions.html"
}


```

接着，在envs 目录中 定制各个ElasticCloud Terraform Code具体变量值

|   |-- envs
|   |   |-- us-production
|   |   |   |-- ec_deployment
|   |   |   |       |-- locals.tf
|   |   |   |       |-- main.tf
|   |   |   |       |-- outputs.tf
|   |   |   |       `-- provider.tf
|   |   |-- staging
|   |   |   |-- ec_deployment
|   |   |   |       |-- locals.tf
|   |   |   |       |-- main.tf
|   |   |   |       |-- outputs.tf
|   |   |   |       `-- provider.tf
|   |   |-- int
|   |   |   |-- ec_deployment
|   |   |   |       |-- locals.tf
|   |   |   |       |-- main.tf
|   |   |   |       |-- outputs.tf
|   |   |   |       `-- provider.tf



如INT下 main.tf
```
module "ec_deployment" {
  source = "../../../../modules/ec_deployment/"

  deployment_name          = "aws-integration-infra"
  deployment_region        = "us-east-1"
  deployment_stack_version = "8.4.1"
  deployment_template_id   = "aws-storage-optimized-v3"

  elasticsearch_hot_size     = "15g"
  elasticsearch_hot_zones    = 2
  elasticsearch_warm_size    = "4g"
  elasticsearch_warm_zones   = 0


```


PROD下 main.tf
```
module "ec_deployment" {
  source = "../../../../modules/ec_deployment/"

  deployment_name          = "aws-production-us-infra"
  deployment_region        = "us-east-1"
  deployment_stack_version = "8.4.1"
  deployment_template_id   = "aws-storage-optimized-v3"

  elasticsearch_hot_size     = "15g"
  elasticsearch_hot_zones    = 4
  elasticsearch_warm_size    = "4g"
  elasticsearch_warm_zones   = 2

```

可以看到 INT和PROD的main.tf 中 使用source引用同个ec_deployment模块,并配置相应的环境变量参数



最后，重要的事，我们将不同Terraform Workspace的定义放在各自env下 provider.tf 中
如INT下provider.tf

```

terraform {
  required_version = "~> 1.3.2"

  required_providers {
    ec = {
      source  = "elastic/ec"
      version = "~> 0.4.0"
    }
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.5"
    }
  }

  cloud {
    organization = "terraform-cloud"

    workspaces {
      name = "infra-elasticsearch--ec_deployment--aws--integration--infra"
    }
  }
}

```


这样，该目录下只为该工作区完成ElasticCloud的部署

例如，我们在envs/integration/ec_deployment/目录下 执行terraform plan
![](images/507c6909109b430b893cbe1679c404ee.png)

Terraform Plan 之后， 该目录下会生产terraform.tfstate 文件，用以保存当前工作区下已部署的资源状态
![](images/3bd77c3e8f52417faa1bf57306bf0437.png)

## 7.4 结论

我们看到了Workspace和reusable modules(可重用模块)如何帮助我们使用独立的状态文件管理独立的基础设施。Reusable modules可重用模块在env下各自的目录架构中运行，包含各个模块bolck、变量、提供程序和后端配置等。其中工作区工作在单个配置文件源上，并在后端创建工作区特定的状态文件。


![](images/b80e8f09614e4a3aade8d16f30b1f8c7.png)



Prod 环境下的 ES hot 节点状态：
![](images/8f4e930305334c04bd0328f48a703ec9.png)

# 8 the safest way to inject sensitive values

Which of the following is the safest way to inject sensitive values into a Terraform Cloud workspape?
A. Write the value to a file and specify the file with the -var-file flag
B. Set a value for the variable in the Ul and check the "Sensitive" check box
C. Edit the state file directly just before running terraform apply
D. Set the variable value on the command line with the -var flag

选 B
B. Set a value for the variable in the Ul and check the "Sensitive" check box.
When working with Terraform Cloud workspaces, the safest way to inject sensitive values into a Terraform Cloud workspace is to set a value for the variable in the Ul and check the "Sensitive" check box. This will ensure that the value is stored securely and not visible in plain text in the Terraform Cloud Ul or API.

B. Set a value for the variable in the Ul and check the "Sensitive" check box is the safest way to inject sensitive values into a Terraform Cloud workspace. This ensures that the sensitive values are securely stored and encrypted in the workspace, and are not visible in the Terraform logs or state file. Writing the value to a file or setting the variable value on the command line can expose the sensitive data to unauthorized access, and editing the state file directly is not a recommended practice and can potentially corrupt the state.

Option A, writing the value to a file and specifying the file with the -var-file flag, may be less secure because the file could potentially be accessed by unauthorized users.

Option C, editing the state file directly just before running terraform apply, is not a best practice and could result in data loss or corruption.

Option D, setting the variable value on the command line with the -var flag, could result in the sensitive value being stored in plain text in the command history or other logs, which could be accessed by unauthorized users.


# 9 One cloud configuration always maps to a single remote workspace: false

from chatgpt
False.
In Terraform, a remote workspace is a part of the Terraform Cloud or Terraform Enterprise service that represents a separate instance of your infrastructure. While it's common to have a one-to-one mapping between configurations and workspaces, it's not a strict requirement.

In practice, you can have multiple remote workspaces associated with a single Terraform configuration. Each workspace can have its own state, variables, and environment. This allows you to manage different instances or environments of your infrastructure using the same configuration.

For example, you might have separate workspaces for development, staging, and production environments, all using the same Terraform configuration To summarize, while it's typical to have a one-to-one mapping between a Terraform configuration and a remote workspace, it's not a strict limitation, and you can use multiple workspaces with a single configuration if it suits your organizational or project needs.


