
https://www.terraform.io/docs/registry/modules/publish.html#requirements

Explanation 
The list below contains all the requirements for publishing a module. Meeting the requirements for publishing a module is extremely easy. The list may appear long only to ensure we're detailed, but adhering to the requirements should happen naturally. 

GitHub. 
The module must be on GitHub and must be a public repo. This is only a requirement for the public registry. If you're using a private registry, you may ignore this requirement. Named `terraform-<PROVIDER>-<NAME>`. Module repositories must use this three-part name format, where `<NAME>` reflects the type of infrastructure the module manages and `<PROVIDER>` is the main provider where it creates that infrastructure. The `<NAME>` segment can contain additional hyphens. Examples: terraform-google-vault or terraform-aws-ec2-instance. 

Repository description. 
The GitHub repository description is used to populate the short description of the module. This should be a simple one-sentence description of the module. 

Standard module structure. 
The module must adhere to the standard module structure. This allows the registry to inspect your module and generate documentation, track resource usage, parse submodules and examples, and more. 

x.y.z tags for releases. 
The registry uses tags to identify module versions. Release tag names must be a semantic version, which can optionally be prefixed with a v. For example, v1.0.4 and 0.9.2. To publish a module initially, at least one release tag must be present. Tags that don't look like version numbers are ignored. 


---


Anyone can publish and share modules on the Terraform Public Module Registry, and meeting the requirements for publishing a module is extremely easy. Select from the following list all valid requirements. (select three) 

A. The module must be PCI/HIPPA compliant. 
B. Module repositories must use this three-part name format, `terraform-<PROVIDER>-<NAME>`. 
C. The registry uses tags to identify module versions. Release tag names must be for the format x.y.z, and can optionally be prefixed with a v . 
D. The module must be on GitHub and must be a public repo.

A 是不对的

