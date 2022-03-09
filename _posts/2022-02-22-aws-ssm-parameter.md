---
layout: post
title: "AWS: using SSM Parameter to set secrets in ECS containers"
description: ""
categories: [devops]
tags: [AWS]
redirect_from:
  - /2022/02/22/
---
[Necessary policies for ECS task execution role](https://docs.aws.amazon.com/kms/latest/developerguide/services-parameter-store.html#parameter-store-policies)
[Security controls with KMS and IAM](https://aws.amazon.com/tw/blogs/security/how-to-use-kms-and-iam-to-enable-independent-security-controls-for-encrypted-data-in-s3/)


### TODOs
- Adding SSM Parameter value setting.
- Change ECS container settings from environment to secrets for sensitive values.

* Note that it is different from using `environment`, when using secrets from SSM Parameter,
only updating the variable value in `terraform.tfvar.json` is not going to trigger a new deployment of the ECS service because the task definition is not updated.
TODO: adding a null_resource for triggering new deployment?

ecs_web.tf
```tf
resource "aws_ecs_task_definition" "web-staging" {
  ...
  - "environment": [
  -  {
  -    "name": "RAILS_MASTER_KEY",
  -    "value": "${var.rails_master_key}"
  -  },
  -  {
  -    "name": "DB_HOST",
  -    "value": "${aws_db_instance.staging.address}"
  -  },
  ...
  + "secrets": [
  +  {
  +    "name": "RAILS_MASTER_KEY",
  +    "valueFrom": "/project-name/staging/rails_master_key"
  +  },
  +  {
  +    "name": "DB_HOST",
  +    "valueFrom": "/project-name/staging/db_host"
  +  },

}
```

ssm_parameter.tf
```tf
resource "aws_ssm_parameter" "rails_master_key" {
  name  = "/project-name/staging/rails_master_key"
  type  = "SecureString"
  value = var.rails_master_key
}

resource "aws_ssm_parameter" "db_host" {
  name  = "/project-name/staging/db_host"
  type  = "SecureString"
  value = aws_db_instance.staging.address
}

...
```
