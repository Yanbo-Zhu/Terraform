
Terraform providers are part of the Terraform core binary.: False
Terraform is logically split into two main parts: <u>Terraform Core and Terraform Plugins</u>. Terraform Core uses remote procedure calls (RPC) to communicate with Terraform Plugins, and offers multiple ways to discover and load plugins to use. Terraform Plugins expose an implementation for a specific service, such as AWS, or provisioner, such as bash.



# 1 Terraform Plugins


Which of these is true about Terraform's plugin-based architecture? 选 B
A. Terraform can only source providers from the internet
B. You can create a provider for your API if none exists
C. Every provider in a configuration has its own state file for its resources
D. All providers are part of the Terraform core binary

Terraform's plugin-based architecture allows users to create custom providers to manage resources that aren't supported by default Terraform providers.



## 1.1 Terraform provider

a plugin that Terraform uses to translate the API interactions with the service or provider

A provider is responsible for understanding API interactions and exposing resources. 
Providers generally are an IaaS (e.g., Alibaba Cloud, AWS, GCP, Microsoft Azure, OpenStack), PaaS (e.g., Heroku), or SaaS services (e.g., Terraform Cloud, DNSimple, CloudFlare).

Terraform is built on a plugin-based architecture. All providers and provisioners that are used in Terraform configurations are plugins, even the core types such as AWS and Heroku.
Users of Terraform are able to write new plugins in order to support new functionality in Terraform.

## 1.2 init 后 provider plugins 插件安装到哪里去: .terraform/plugins 

A. The .terraform.plugins directory in the directory terraform init was executed in. 
B. The .terraform/plugins directory in the directory terraform init was executed in. 
C. /etc/terraform/plugins 
D. The .terraform.d directory in the directory terraform init was executed in.

选 B