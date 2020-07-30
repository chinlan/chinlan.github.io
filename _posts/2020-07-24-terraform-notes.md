---
layout: post
title: "terraform notes"
description: "Deploy api backend app (Rails 6) and frontend app (Next.js + React.js + TypeScript) to AWS ECS"
categories: [devops]
tags: []
redirect_from:
  - /2020/07/24/
---

- If there is already some resources built by using web console or cli command:
`terraform import xxxxxxx yyyyyyy`

Note that `import` will only revise the tfstate, the configuration in the code will need to be done by yourself.

- If some resources cannot be destroyed by running `terraform destroy`:
`terraform state rm xxxxxxx`, then manually destroy the resource using web console or cli command.

- To show the list or details of resources under managed:
`terraform state list`
`terraform state show xxxxxx`

- When revising `aws_security_group`, moving the `ingress {}` block out as an isolated `aws_security_group_rule` resource:
[ref](https://github.com/hashicorp/terraform/pull/2376)
`terraform state rm <target_security_group>`
Revise the configuration to use non-inline rules.
`terraform import <target_security_group>`
`terraform state mv` each rule into its proper location.
