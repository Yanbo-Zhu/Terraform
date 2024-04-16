
Terraform Cloud is more powerful when you integrate it with your version control system (VCS) provider Although you can use many of Terraform Cloud's features without one, a VCS connection provides additional features and improved workflows. 
In particular: When workspaces are linked to a VCS repository, ==Terraform Cloud can automatically initiate Terraform runs when changes are committed to the specified branch.== Terraform Cloud makes code review easier by automatically predicting how pull requests will affect infrastructure-Publishing new versions of a private Terraform module is as easy as pushing a tag to the module's repository

We recommend configuring VCS access when first setting up an organization, and you might need to add additional VCS providers later depending on how your organization grows. Configuring a new VCS provider requires permission to manage VCS settings for the organization. (More about permissions.)


How can you trigger a run in a Terraform Cloud workspace that is connected to a Version Control System (VCS) repository?
C. Only members of a VCS organization can open a pull request against repositories that are connected to Terraform Cloud workspaces

# 1 不同的 support plan

Terraform Cloud is a commercial SaaS product developed by HashiCorp. Many of its features are free for small teams, including remote state storage, remote runs, and VCS connections. We also offer paid plans for larger teams that include additional collaboration and governance features.
Each higher paid upgrade plan is a strict superset of any lower plans - for example, the Team & Governance plan includes all of the features of this Team plan.

