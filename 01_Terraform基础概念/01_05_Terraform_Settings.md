
https://developer.hashicorp.com/terraform/language/settings

-> `terraform` configuration block

The special `terraform` configuration block type is used to configure some behaviors of Terraform itself, such as requiring a minimum Terraform version to apply your configuration.

# 1 Terraform_Settings Basic

## 1.1 Terraform Block Syntax

Terraform settings are gathered together into `terraform` blocks:

```hcl
terraform {
  # ...
}
```

Each `terraform` block can contain a number of settings related to Terraform's behavior. Within a `terraform` block, only constant values can be used; arguments may not refer to named objects such as resources, input variables, etc, and may not use any of the Terraform language built-in functions.

The various options supported within a `terraform` block are described in the following sections.

## 1.2 Configuring Terraform Cloud

The nested `cloud` block configures Terraform Cloud for enabling its [CLI-driven run workflow](https://developer.hashicorp.com/terraform/cloud-docs/run/cli).

- Refer to [Terraform Cloud Configuration](https://developer.hashicorp.com/terraform/language/settings/terraform-cloud) for a summary of the `cloud` block's syntax.
- Refer to [Using Terraform Cloud](https://developer.hashicorp.com/terraform/cli/cloud) in the Terraform CLI documentation for complete details about how to initialize and configure the Terraform Cloud CLI integration.

## 1.3 Configuring a Terraform Backend

The nested `backend` block configures which state backend Terraform should use.
The syntax and behavior of the `backend` block is described in [Backend Configuration](https://developer.hashicorp.com/terraform/language/settings/backends/configuration).

## 1.4 Specifying a Required Terraform Version `required_version`

> **Hands-on:** Try the [Manage Terraform Versions](https://developer.hashicorp.com/terraform/tutorials/configuration-language/versions) or [Manage Terraform Versions in Terraform Cloud](https://developer.hashicorp.com/terraform/tutorials/cloud/cloud-versions) tutorials.

The `required_version` setting accepts a [version constraint string,](https://developer.hashicorp.com/terraform/language/expressions/version-constraints) which specifies which versions of Terraform can be used with your configuration.

If the running version of Terraform doesn't match the constraints specified, Terraform will produce an error and exit without taking any further actions.

When you use [child modules](https://developer.hashicorp.com/terraform/language/modules), each module can specify its own version requirements. The requirements of all modules in the tree must be satisfied.

Use Terraform version constraints in a collaborative environment to ensure that everyone is using a specific Terraform version, or using at least a minimum Terraform version that has behavior expected by the configuration.

The `required_version` setting applies only to the version of Terraform CLI. Terraform's resource types are implemented by provider plugins, whose release cycles are independent of Terraform CLI and of each other. Use [the `required_providers` block](https://developer.hashicorp.com/terraform/language/providers/requirements) to manage the expected versions for each provider you use.

## 1.5 specifying Provider Requirements `required_providers` 

The `required_providers` block specifies all of the providers required by the current module, mapping each local provider name to a source address and a version constraint.

```hcl
terraform {
  required_providers {
    aws = {
      version = ">= 2.7.0"
      source = "hashicorp/aws"
    }
  }
}
```

For more information, see [Provider Requirements](https://developer.hashicorp.com/terraform/language/providers/requirements).

## 1.6 Experimental Language Features

The Terraform team will sometimes introduce new language features initially via an opt-in experiment, so that the community can try the new feature and give feedback on it prior to it becoming a backward-compatibility constraint.

In releases where experimental features are available, you can enable them on a per-module basis by setting the `experiments` argument inside a `terraform` block:

```hcl
terraform {
  experiments = [example]
}
```

The above would opt in to an experiment named `example`, assuming such an experiment were available in the current Terraform version.


## 1.7 Passing Metadata to Providers  `provider_meta`

The `terraform` block can have a nested `provider_meta` block for each provider a module is using, if the provider defines a schema for it. This allows the provider to receive module-specific information, and is primarily intended for modules distributed by the same vendor as the associated provider.

For more information, see [Provider Metadata](https://developer.hashicorp.com/terraform/internals/provider-meta).


