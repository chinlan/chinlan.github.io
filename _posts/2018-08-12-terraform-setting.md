---
layout: post
title: "Terraform setting"
description: ""
categories: [devops]
tags: [Terraform]
redirect_from:
  - /2018/08/12/
---
- Download an appropriate version of zip file from official website
- Unzip the file  
- Add terraform binary to PATH environment variable, or just put it in somewhere already exported as PATH (ex: home/bin)
- AWS configuration
`Export AWS_ACCESS_KEY_ID=xxxxx`
`Export AWS_SECRET_ACCESS_KEY=xxxxx`
Notice these environment variables will only apply to the current shell.
- Configure the provider you want to use. 
/terraform-ex/main.tf
~~~~~~~~
provider "aws" {

  region = "us-east-1"

}
~~~~~~~~
- The general syntax for a Terraform resource is:
~~~~~~~~
resource "PROVIDER_TYPE"  "NAME" {

  [CONFIG ...]

}
~~~~~~~~
- `terraform init`
Getting plugin needed based on the given provider in main.tf
- `terraform plan`
Showing what is going to be done
- `terraform apply`
Applying everything in the plan

- Accessing resource attributes by interpolation. The syntax is: `"${TYPE.NAME.ATTRIBUTE}"`

- Writing dry code using variables. The syntax is:
~~~
variable "NAME" {
    [CONFIG ...]
}
~~~

- Fetching read-only information from the provider(AWS). The syntax is:
declaration
~~~
data "TYPE" "NAME" {}
~~~
reference
~~~
"${data.TYPE.NAME.ATTRIBUTE}"
~~~

For small project developed by one person (and probably not many environments), it is convenient to put all resource in one file and to build up all resources with one command, but when it comes to bigger project developed by a team, it is better to seperate resources into different files based on environments and usage.
State management will also be needed,
version control may not be the best way to do state management,
it is easy to forget to pull before revising and apply the code, this can be avoided by preventing anyone to apply from local. By setting things like circleCI to apply the updated code, concurrent changes can be avoided.
The best way to manage shared storage for state files: `Remote State Storage`, ex: AWS s3.
