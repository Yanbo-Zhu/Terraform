
# 1 Sentinel policy


Your risk management organization requires that new AWS S3 buckets must be private and encrypted at rest. How can Terraform Enterprise automatically and proactively enforce this security control?
A. With a Sentinel policy, which runs before every apply.

Terraform Enterprise can enforce security controls through the use of Sentinel policies. 
Sentinel is a policy as code framework that integrates with Terraform Enterprise and can be used to enforce specific security controls. In this case, the Sentinel policy could check that all new S3 buckets are set to be private and encrypted at rest and prevent the Terraform apply from proceeding if the buckets do not meet this requirement. This ensures that the security control is automatically and proactively enforced every time Terraform makes changes to the infrastructure.

Terraform Enterprise provides the ability to enforce security controls through Sentinel policies, which are a form of policy as code. 
Sentinel policies allow you to define and enforce organizational or regulatory policies by creating a set of rules that run before each Terraform operation.


----

When does Sentinel enforce policy logic during a Terraform Enterprise run?
A. Before the plan phase
B. During the plan phase
C. Before the apply phase
D. After the apply phase
选C

"Enforcing policy checks on runs - Policies are checked when a run is performed, after the terraform plan but before it can be confirmed or the terraform apply is executed."
