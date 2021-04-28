---
layout: post
title: "AWS: Session Manager"
description: ""
categories: [devops]
tags: [AWS]
redirect_from:
  - /2020/10/21/
---
[Getting started](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started.html)

# Prerequisites:
- (Advanced-instances tier ? not sure if need to enable this)
  [Enable advanced tier](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-managedinstances-advanced.html)

- Need `SSM Agent` >= 2.3.672.0 (having both encryption and port forwading functions) to be installed on the EC2 instances which are the targets to connect to.
  [Working with SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent.html)
  [Automating updates to SSM Agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-automatic-updates.html)
  - ssm-user account: A local user account called `ssm-user`, created the first time a session starts on the instance, having root or administrator privileges. (Added to `/etc/sudoers` when is it Linux) The ssm-user account is not removed from the system when SSM Agent is uninstalled. No passwords are set for ssm-user on Linux managed instances.
  - Systems Manager relies on EC2 instance metadata to function correctly.
  - Requires permissions in order to communicate with the Systems Manager Service, on EC2 instances, these permissions are provided in an `instance profile` that is attached to the instance.
  [Session Manager getting started instance profile](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-instance-profile.html)
  [Attach an IAM instance profile to an EC2 instance](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-launch-managed-instance.html)
  - `Amazon Linux 2 ECS-Optimized AMIs` preinstalled SSM Agent by default. May need to change to use these, or will need to manually install SSM Agent ourselves.
  [Manually install SSM Agent on Amazon Linux instances](https://docs.aws.amazon.com/systems-manager/latest/userguide/agent-install-al.html)
  [sysman install ssm agent](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html)

# Enable SSH connections:
[session manager getting started enable ssh connections](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-getting-started-enable-ssh-connections.html)


1. Current AMI does not preinstall SSM Agent, need to manually install it:
```
sudo yum install -y https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm
```

2. Update EC2 instance profile role to have required permissions:
~~~ tf
resource "aws_iam_role_policy_attachment" "ec2-role-for-ssm-attachment" {
  role = aws_iam_role.production-ecs-instance-role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM"
  depends_on = [aws_iam_role.production-ecs-instance-role]
  lifecycle{
    prevent_destroy = true
  }
}

resource "aws_iam_role_policy" "ecs-instance-role-inline-policy-1" {
  name = "SessionManagerPermissionWithoutKMSEncrypt"
  role = aws_iam_role.production-ecs-instance-role.id
  policy = <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
          "ssmmessages:CreateControlChannel",
          "ssmmessages:CreateDataChannel",
          "ssmmessages:OpenControlChannel",
          "ssmmessages:OpenDataChannel"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
          "s3:GetEncryptionConfiguration"
      ],
      "Resource": "*"
    }
  ]
}
EOF
  depends_on = [
    aws_iam_role.production-ecs-instance-role
  ]
  lifecycle {
    prevent_destroy = true
  }
}
~~~

Did not set cloudwatch or s3 log.

[reference](https://masakimisawa.com/ssm-ec2-ssh/)



