R
Terraform Cloud is more powerful when you integrate it with your version control system (VCS) provider Although you can use many of Terraform Cloud's features without one, a VCS connection provides additional features and improved workflows. 
In particular: When workspaces are linked to a VCS repository, ==Terraform Cloud can automatically initiate Terraform runs when changes are committed to the specified branch.== Terraform Cloud makes code review easier by automatically predicting how pull requests will affect infrastructure-Publishing new versions of a private Terraform module is as easy as pushing a tag to the module's repository

We recommend configuring VCS access when first setting up an organization, and you might need to add additional VCS providers later depending on how your organization grows. Configuring a new VCS provider requires permission to manage VCS settings for the organization. (More about permissions.)


How can you trigger a run in a Terraform Cloud workspace that is connected to a Version Control System (VCS) repository?
C. Only members of a VCS organization can open a pull request against repositories that are connected to Terraform Cloud workspaces

# 1 不同的 support plan

Terraform Cloud is a commercial SaaS product developed by HashiCorp. Many of its features are free for small teams, including remote state storage, remote runs, and VCS connections. We also offer paid plans for larger teams that include additional collaboration and governance features.
Each higher paid upgrade plan is a strict superset of any lower plans - for example, the Team & Governance plan includes all of the features of this Team plan.


# 2 speculative plan

In a Terraform Cloud workspace linked to a version control repository, speculative plan runs start automatically when you merge or commit changes to version control.

It is true because Terraform Cloud can monitor the linked version control repository for changes and automatically trigger a speculative plan run into response to each commit or merge. This allows users to quickly see the expected changes that would result from the proposed change, without actually applying those changes. Speculative plan runs can be a useful tool for catching errors early in the development process and avoiding potentially costly mistakes.

Whether to perform speculative plans on pull requests to the connected repository, to assist in reviewing proposed changes. Automatic speculativ plans are enabled by default, but you can disable them for any workspace.

# 3 Terraform Cloud's features and free tirer

Terraform Cloud's free tier
A. Workspace and workspace management 
B. Remote state management
D. Private module registry
VCS Integration
They can securely store cloud credentials


不是免费的, 但是可以用 feature is not included in Terraform Cloud's free tier?
C. Audit logging
B  roles and team management


根本就不提供使用 
C. Automatic backups
D. Automated infrastructure deployment visualization
You  use the CLI to switch between workspace

## 3.1 题目 

Which of these are features of Terraform Cloud? (Choose two.) 选 ab 
A. Remote state storage
B. A web-based user interface (Ul)

C. Automatic backups
D. Automated infrastructure deployment visualization

---



Which of these statements about Terraform Cloud workspaces is false? C 
A. They can securely store cloud credentials
B. They have role-based access controls
C. You must use the CLI to switch between workspaces
D. Plans and applies can be triggered via version control system integrations


# 4 Terraform Cloud supports the following VCS providers:

https://www.terraform.io/docs/cloud/vcs/index.html#supported-vcs-providers
- GitHub 
- GitHub.com (OAuth) 
- GitHub Enterprise 
- GitLab.com 
- GitLab EE and CE 
- Bitbucket Cloud
- Bitbucket Server 
- Azure DevOps Server 
- Azure DevOps Services

不被支持: 
B. CVS Version Control
