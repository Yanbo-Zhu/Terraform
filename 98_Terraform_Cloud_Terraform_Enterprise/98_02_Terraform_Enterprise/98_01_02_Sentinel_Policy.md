
sentinel 哨兵

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

---

Policies are enforced after the plan and before the apply commands. so, the answer is option C.
terraform plan > >sentinel policy> >>terraform apply

Terraform Enterprise enforces Sentinel policies between the plan and apply phases of a run, preventing out of policy infrastructure from being
provisioned. Unless overridden by an authorized user, only plans that pass all Sentinel policies checked against them are allowed to proceed to
the apply step.

Sentinel is a policy-as-code framework integrated with Terraform Enterprise that allows organizations to define and enforce policies on
infrastructure changes. Sentinel enforces policy logic during the plan phase of a Terraform Enterprise run, before any changes are applied to the
infrastructure. During the plan phase, Terraform generates an execution plan that describes the changes that will be made to the infrastructure.
Sentinel evaluates policy rules against this execution plan to determine whether the proposed changes comply with the defined policies. If any
violations are detected, the plan is rejected, and the changes are not applied.
Sentinel does not enforce policy logic before the plan phase or after the apply phase. However, Sentinel policies can also be used to enforce
compliance on policy requirements that are not directly related to infrastructure changes, such as resource tagging or naming conventions. In thes cases, Sentinel policies may be evaluated at other points in the Terraform Enterprise workflow, such as during VCS (version control system)
integration or during cost estimation.

