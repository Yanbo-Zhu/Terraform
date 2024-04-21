
Which of these statements about Terraform Enterprise workspaces is false?
A. They can securely store cloud credentials
B. You must use the CLI to switch between workspaces
C. Plans and applies can be triggered via version control system integrations
D. They have role-based access controls

B. You must use the CLI to switch between workspaces: false
Terraform Enterprise provides a web-based Ul that allows you to switch between workspaces, view the state of your infrastructure, and run Terraform commands without having to use the command line interface.


# 1 Terraform Enterprise 独有的特性 

ACDF
A. SAML/SSO
C. Audit Logs
D. Clustering
F. Private Network Connectivity

不是 terraform Enterprise 独有的特性
B. Sentinel
E. Private Module Registry
A. Cost Estimation

Sentinel and Cost Estimation are both available in Terraform Cloud, though not at the free tier level.

# 2 type of backend database
Terraform Enterprise (also referred to as pTFE) requires what type of backend database for a clustered deployment?
A. PostgreSQL 
B. Cassandra
C. MySQL 
D. MSSQL

PostgreSQL , S3-compatible endpoint, Azure blob storage 是可以的

Explanation
External Services mode stores the majority of the stateful data used by the instance in an external PostgreSQL database and an external S3-compatible endpoint or Azure blob storage. 
There is still critical data stored on the instance that must be managed with snapshots. Be sure to check the PostgreSQL Requirements for information that needs to be present for Terraform Enterprise to work.
This option is best for users with expertise managing PostgreSQL or users that have access to managed PostgreSQL offerings like AWS RDS.


