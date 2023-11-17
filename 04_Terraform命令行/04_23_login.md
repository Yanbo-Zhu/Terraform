https://developer.hashicorp.com/terraform/tutorials/aws-get-started/aws-remote

# 1 介绍 Terraform Cloud account 
## 1.1 Set up Terraform Cloud

If you have a HashiCorp Cloud Platform or Terraform Cloud account, log in using your existing credentials. For more detailed instructions on how to sign up for a new account and create an organization, review the [Sign up for Terraform Cloud](https://developer.hashicorp.com/terraform/tutorials/cloud-get-started/cloud-sign-up) tutorial.

Next, modify `main.tf` to add a `cloud` block to your Terraform configuration, and replace `organization-name` with your organization name.

main.tf

```hcl
terraform {
  cloud {
    organization = "organization-name"
    workspaces {
      name = "learn-tfc-aws"
    }
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
}
```

Note

Older version of Terraform do not support the `cloud` block, so you must use **1.1.0 or higher** in order to follow this tutorial. Previous versions can use the [`remote` backend block](https://developer.hashicorp.com/terraform/language/settings/backends/remote) to configure the CLI workflow and migrate state.

## 1.2 Login to Terraform Cloud

Next, log into your Terraform Cloud account with the Terraform CLI in your terminal.

```shell-session
 terraform login
Terraform will request an API token for app.terraform.io using your browser.

If login is successful, Terraform will store the token in plain text in
the following file for use by subsequent commands:
    /Users/<USER>/.terraform.d/credentials.tfrc.json

Do you want to proceed?
  Only 'yes' will be accepted to confirm.

  Enter a value:
```

Confirm with a `yes` and follow the workflow in the browser window that will automatically open. You will need to paste the generated API key into your Terminal when prompted. For more detail on logging in, follow the [Authenticate the CLI with Terraform Cloud tutorial](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-login).
