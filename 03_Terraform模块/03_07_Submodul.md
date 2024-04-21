
# 1 总览 

## 1.1 variables inheritance

==Modules do not inherit variables from the parent module. ==
All modules are self-contained units. So you have to explicitly define variables in the child module, and then explicit set these variables in the parent module, when you instantiate the child module.
Module variable assignments must be explicitly set by the parent module when it instantiates the child module.

# 2 用其他方式来代替 submodul 

https://stackoverflow.com/questions/66873794/how-to-create-terraform-sub-module

it would be good to create composable modules rather than submodules.

```bash
❯❯ tree
.
├── main.tf
├── sub_module_1
│   └── main.tf
├── sub_module_2
│   └── main.tf
└── terraform.tfstate

2 directories, 4 files
tmp/boo/parent_module
❯❯ ls
main.tf            sub_module_1/      sub_module_2/      terraform.tfstate
```

```bash
❯❯ terraform state list
aws_iam_role_policy_attachment.lambda_logs
aws_sns_topic.user_updates
module.iam_role_info.aws_iam_role.iam_for_lambda
module.logging_policy.aws_iam_policy.lambda_logging
```

```bash
❯❯ ls
main.tf        parent_module/
/private/tmp/boo
❯❯ cat main.tf
variable "env" {
  type    = string
  default = "dev"
}
locals {
  default_tags = {
    Product     = "wallaby",
    Environment = var.env,
    Application = "wallaby-api"
  }
}

module "parent_module" {
source = "./parent_module"
}

output "sns_info" {
  value = module.parent_module.sns_info
}
```

in the root of the `parent_module` `main.tf`

```java
❯❯ cat main.tf
module "iam_role_info" {
  source = "./sub_module_1"
}

module "logging_policy" {
  source = "./sub_module_2"
}

resource "aws_iam_role_policy_attachment" "lambda_logs" {
  role       = module.iam_role_info.lambda_role_name
  policy_arn = module.logging_policy.iam_policy_arn
}

resource "aws_sns_topic" "user_updates" {
  name = "user-updates-topic"
}

output "sns_info" {
  value = aws_sns_topic.user_updates.arn
}
```

```lua
❯❯ cat sub_module_1/main.tf
resource "aws_iam_role" "iam_for_lambda" {
  name = "iam_for_lambda"

  assume_role_policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "sts:AssumeRole",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Effect": "Allow",
      "Sid": ""
    }
  ]
}
EOF
}

output "lambda_role_name" {
    value = aws_iam_role.iam_for_lambda.name
}
```

```makefile
❯❯ cat sub_module_2/main.tf
# See also the following AWS managed policy: AWSLambdaBasicExecutionRole
resource "aws_iam_policy" "lambda_logging" {
  name        = "lambda_logging"
  path        = "/"
  description = "IAM policy for logging from a lambda"

  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*",
      "Effect": "Allow"
    }
  ]
}
EOF
}

output "iam_policy_arn" {
value = aws_iam_policy.lambda_logging.arn
```

