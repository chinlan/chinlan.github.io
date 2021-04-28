---
layout: post
title: "AWS: CodeDeploy + circleCI"
description: ""
categories: [devops]
tags: [AWS]
redirect_from:
  - /2020/10/26/
---

[reference](https://dev.classmethod.jp/articles/deploy-with-codedeploy-circleci2-0/)
[circleCI blog](https://circleci.com/blog/graceful-shutdown-using-aws/)
[circleCI doc: deployment-integrations](https://circleci.com/docs/2.0/deployment-integrations/#section=deployment)
[aws-code-deploy orb](https://circleci.com/developer/orbs/orb/circleci/aws-code-deploy)
[terraform CodeDeploy doc](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/codedeploy_deployment_group)

[Continuous Blue-Green Deployment to Highly Automated AWS ECS Fargate Cluster via AWS CodeDeploy, Gitlab CI/CD and Terraform](https://medium.com/@ahmetatalay/continuous-blue-green-deployment-to-highly-automated-aws-ecs-fargate-cluster-via-aws-codedeploy-f4d44fc28231)

## Terraform add CODEDEPLOY related settings:
staging/code_deploy.tf
~~~tf
data "aws_iam_role" "ecs-code-deploy" {
  name = "ecsCodeDeployRole"
}

resource "aws_codedeploy_app" "ninninjob-web-staging" {
  compute_platform = "ECS"
  name = "ninninjob-web-staging"
}

resource "aws_codedeploy_deployment_group" "ninninjob-web-staging-group" {
  # count                  = "${var.enable_codedeploy ? 1 : 0}"
  app_name               = aws_codedeploy_app.ninninjob-web-staging.name
  deployment_config_name = "CodeDeployDefault.ECSAllAtOnce"
  deployment_group_name  = "ninninjob-web-staging-group"
  service_role_arn       = data.aws_iam_role.ecs-code-deploy.arn

  auto_rollback_configuration {
    enabled = true
    events  = ["DEPLOYMENT_FAILURE"]
  }

  blue_green_deployment_config {
    deployment_ready_option {
      action_on_timeout = "CONTINUE_DEPLOYMENT"
    }

    terminate_blue_instances_on_deployment_success {
      action                           = "TERMINATE"
      termination_wait_time_in_minutes = 5
    }
  }

  deployment_style {
    deployment_option = "WITH_TRAFFIC_CONTROL"
    deployment_type   = "BLUE_GREEN"
  }

  ecs_service {
    cluster_name = aws_ecs_cluster.ninninjob-staging.name
    service_name = aws_ecs_service.ninninjob-web-staging.name
  }

  load_balancer_info {
    target_group_pair_info {
      prod_traffic_route {
        listener_arns = [aws_lb_listener.https-listener-green.arn] # Production traffic route must have exactly one listener Arn
      }

      test_traffic_route {
        listener_arns = [aws_lb_listener.https-listener-blue.arn]
      }

      target_group {
        name = aws_lb_target_group.ninninjob-web-staging-green.name
      }

      target_group {
        name = aws_lb_target_group.ninninjob-web-staging-blue.name
      }
    }
  }

  depends_on = [
    aws_codedeploy_app.ninninjob-web-staging,
    aws_ecs_cluster.ninninjob-staging,
    aws_ecs_service.ninninjob-web-staging,
    aws_lb_listener.https-listener-blue,
    aws_lb_listener.https-listener-green,
    aws_lb_target_group.ninninjob-web-staging-blue,
    aws_lb_target_group.ninninjob-web-staging-green
  ]
}

resource "aws_codedeploy_app" "ninninjob-frontend-staging" {
  compute_platform = "ECS"
  name = "ninninjob-frontend-staging"
}

resource "aws_codedeploy_deployment_group" "ninninjob-frontend-staging-group" {
  # count                  = "${var.enable_codedeploy ? 1 : 0}"
  app_name               = aws_codedeploy_app.ninninjob-frontend-staging.name
  deployment_config_name = "CodeDeployDefault.ECSAllAtOnce"
  deployment_group_name  = "ninninjob-frontend-staging-group"
  service_role_arn       = data.aws_iam_role.ecs-code-deploy.arn

  auto_rollback_configuration {
    enabled = true
    events  = ["DEPLOYMENT_FAILURE"]
  }

  blue_green_deployment_config {
    deployment_ready_option {
      action_on_timeout = "CONTINUE_DEPLOYMENT"
    }

    terminate_blue_instances_on_deployment_success {
      action                           = "TERMINATE"
      termination_wait_time_in_minutes = 5
    }
  }

  deployment_style {
    deployment_option = "WITH_TRAFFIC_CONTROL"
    deployment_type   = "BLUE_GREEN"
  }

  ecs_service {
    cluster_name = aws_ecs_cluster.ninninjob-staging.name
    service_name = aws_ecs_service.ninninjob-frontend-staging.name
  }

  load_balancer_info {
    target_group_pair_info {
      prod_traffic_route {
        listener_arns = [aws_lb_listener.https-listener-green.arn] # Production traffic route must have exactly one listener Arn
      }

      test_traffic_route {
        listener_arns = [aws_lb_listener.https-listener-blue.arn]
      }

      target_group {
        name = aws_lb_target_group.ninninjob-frontend-staging-green.name
      }

      target_group {
        name = aws_lb_target_group.ninninjob-frontend-staging-blue.name
      }
    }
  }

  depends_on = [
    aws_codedeploy_app.ninninjob-frontend-staging,
    aws_ecs_cluster.ninninjob-staging,
    aws_ecs_service.ninninjob-frontend-staging,
    aws_lb_listener.https-listener-blue,
    aws_lb_listener.https-listener-green,
    aws_lb_target_group.ninninjob-frontend-staging-blue,
    aws_lb_target_group.ninninjob-frontend-staging-green
  ]
}

resource "null_resource" "redeploy-ninninjob-web-staging" {
  triggers = {
    task_definition_arn = aws_ecs_task_definition.ninninjob-web-staging.arn
  }

  provisioner "local-exec" {
    command = "bash ./redeploy.sh ${aws_codedeploy_app.ninninjob-web-staging.name} ${aws_codedeploy_deployment_group.ninninjob-web-staging-group.deployment_group_name} ${aws_ecs_task_definition.ninninjob-web-staging.family} 3000 ${aws_ecs_task_definition.ninninjob-web-staging.arn}"
  }

  depends_on = [
    aws_ecs_task_definition.ninninjob-web-staging,
    aws_codedeploy_app.ninninjob-web-staging,
    aws_codedeploy_deployment_group.ninninjob-web-staging-group
  ]
}

resource "null_resource" "redeploy-ninninjob-frontend-staging" {
  triggers = {
    task_definition_arn = aws_ecs_task_definition.ninninjob-frontend-staging.arn
  }

  provisioner "local-exec" {
    command = "bash ./redeploy.sh ${aws_codedeploy_app.ninninjob-frontend-staging.name} ${aws_codedeploy_deployment_group.ninninjob-frontend-staging-group.deployment_group_name} ${aws_ecs_task_definition.ninninjob-frontend-staging.family} 4000 ${aws_ecs_task_definition.ninninjob-frontend-staging.arn}"
  }

  depends_on = [
    aws_ecs_task_definition.ninninjob-frontend-staging,
    aws_codedeploy_app.ninninjob-frontend-staging,
    aws_codedeploy_deployment_group.ninninjob-frontend-staging-group
  ]
}
~~~

Revise ALB listener settings:
staging/lb.tf
~~~tf
resource "aws_lb" "ninninjob-staging-alb" {
  name = "ninninjob-staging-alb"
  load_balancer_type = "application"
  security_groups = [aws_security_group.ninninjob-staging-alb.id]
  subnets = [aws_subnet.ninninjob-staging-subnet1.id, aws_subnet.ninninjob-staging-subnet2.id]
  enable_deletion_protection = false
  depends_on = [
    aws_security_group.ninninjob-staging-alb,
    aws_subnet.ninninjob-staging-subnet1,
    aws_subnet.ninninjob-staging-subnet2,
  ]
}

# resource "aws_lb_target_group" "ninninjob-web-staging" {
#   name = "ninninjob-web-staging"
#   port = 80
#   protocol = "HTTP"
#   vpc_id = aws_vpc.ninninjob-staging-vpc.id
#   health_check {
#     healthy_threshold = 5
#     unhealthy_threshold = 2
#     path = "/healthcheck"
#     matcher = "200"
#   }
#   depends_on = [
#     aws_vpc.ninninjob-staging-vpc,
#     aws_lb.ninninjob-staging-alb
#   ]
# }

resource "aws_lb_target_group" "ninninjob-web-staging-blue" {
  name = "ninninjob-web-staging-blue"
  port = 8080
  protocol = "HTTP"
  vpc_id = aws_vpc.ninninjob-staging-vpc.id
  health_check {
    healthy_threshold = 5
    unhealthy_threshold = 2
    path = "/healthcheck"
    matcher = "200"
  }
  depends_on = [
    aws_vpc.ninninjob-staging-vpc,
    aws_lb.ninninjob-staging-alb
  ]
}

resource "aws_lb_target_group" "ninninjob-web-staging-green" {
  name = "ninninjob-web-staging-green"
  port = 80
  protocol = "HTTP"
  vpc_id = aws_vpc.ninninjob-staging-vpc.id
  health_check {
    healthy_threshold = 5
    unhealthy_threshold = 2
    path = "/healthcheck"
    matcher = "200"
  }
  depends_on = [
    aws_vpc.ninninjob-staging-vpc,
    aws_lb.ninninjob-staging-alb
  ]
}

# resource "aws_lb_target_group" "ninninjob-frontend-staging" {
#   name = "ninninjob-frontend-staging"
#   port = 80
#   protocol = "HTTP"
#   vpc_id = aws_vpc.ninninjob-staging-vpc.id
#   health_check {
#     healthy_threshold = 5
#     unhealthy_threshold = 2
#     path = "/manager/login"
#   }
#   depends_on = [
#     aws_vpc.ninninjob-staging-vpc,
#     aws_lb.ninninjob-staging-alb
#   ]
# }

resource "aws_lb_target_group" "ninninjob-frontend-staging-blue" {
  name = "ninninjob-frontend-staging-blue"
  port = 8080
  protocol = "HTTP"
  vpc_id = aws_vpc.ninninjob-staging-vpc.id
  health_check {
    healthy_threshold = 5
    unhealthy_threshold = 2
    path = "/manager/login"
  }
  depends_on = [
    aws_vpc.ninninjob-staging-vpc,
    aws_lb.ninninjob-staging-alb
  ]
}

resource "aws_lb_target_group" "ninninjob-frontend-staging-green" {
  name = "ninninjob-frontend-staging-green"
  port = 80
  protocol = "HTTP"
  vpc_id = aws_vpc.ninninjob-staging-vpc.id
  health_check {
    healthy_threshold = 5
    unhealthy_threshold = 2
    path = "/manager/login"
  }
  depends_on = [
    aws_vpc.ninninjob-staging-vpc,
    aws_lb.ninninjob-staging-alb
  ]
}

resource "aws_lb_listener" "http-listener-green" {
  load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
  port = "80"
  protocol = "HTTP"
  default_action {
    # type = "forward"
    # target_group_arn = aws_lb_target_group.ninninjob-frontend-staging-green.arn
    type = "redirect"
    redirect {
      port = "443"
      protocol = "HTTPS"
      status_code = "HTTP_301"
    }
  }
  depends_on = [
    aws_lb.ninninjob-staging-alb,
    # aws_lb_target_group.ninninjob-frontend-staging-green,
  ]
}

resource "aws_lb_listener" "http-listener-blue" {
  load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
  port = "8080"
  protocol = "HTTP"
  default_action {
    # type = "forward"
    # target_group_arn = aws_lb_target_group.ninninjob-frontend-staging-blue.arn
    type = "redirect"
    redirect {
      port = "8443"
      protocol = "HTTPS"
      status_code = "HTTP_301"
    }
  }
  depends_on = [
    aws_lb.ninninjob-staging-alb,
    # aws_lb_target_group.ninninjob-frontend-staging-blue,
  ]
}

# resource "aws_lb_listener" "https-listener" {
#   load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
#   port = "443"
#   protocol = "HTTPS"
#   ssl_policy = "ELBSecurityPolicy-2016-08"
#   # certificate_arn = aws_acm_certificate.ninninjob-staging-certificate.arn
#   certificate_arn = data.aws_acm_certificate.ninninjob-certificate.arn
#   default_action {
#     type = "forward"
#     target_group_arn = aws_lb_target_group.ninninjob-frontend-staging.arn
#   }
#   depends_on = [
#     aws_lb.ninninjob-staging-alb,
#     aws_lb_target_group.ninninjob-frontend-staging,
#   ]
# }

resource "aws_lb_listener" "https-listener-blue" {
  load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
  port = "8443"
  protocol = "HTTPS"
  ssl_policy = "ELBSecurityPolicy-2016-08"
  # certificate_arn = aws_acm_certificate.ninninjob-staging-certificate.arn
  certificate_arn = data.aws_acm_certificate.ninninjob-certificate.arn
  default_action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-frontend-staging-green, aws_lb_target_group.ninninjob-frontend-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-frontend-staging-blue.arn
  }
  depends_on = [
    aws_lb.ninninjob-staging-alb,
    aws_lb_target_group.ninninjob-frontend-staging-blue
  ]
}

resource "aws_lb_listener" "https-listener-green" {
  load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
  port = "443"
  protocol = "HTTPS"
  ssl_policy = "ELBSecurityPolicy-2016-08"
  # certificate_arn = aws_acm_certificate.ninninjob-staging-certificate.arn
  certificate_arn = data.aws_acm_certificate.ninninjob-certificate.arn
  default_action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-frontend-staging-green, aws_lb_target_group.ninninjob-frontend-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-frontend-staging-green.arn
  }
  depends_on = [
    aws_lb.ninninjob-staging-alb,
    aws_lb_target_group.ninninjob-frontend-staging-green
  ]
}

# resource "aws_lb_listener_rule" "web" {
#   listener_arn = aws_lb_listener.http-listener.arn
#   priority = 100
#   action {
#     type = "forward"
#     target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
#   }
#   condition {
#     path_pattern {
#       values = [
#         "/healthcheck",
#         "/api/*",
#         "/admin_users/*",
#         "/users/*",
#         "/admins",
#       ]
#     }
#   }
#   depends_on = [
#     aws_lb_listener.http-listener,
#     aws_lb_target_group.ninninjob-web-staging,
#   ]
# }

resource "aws_lb_listener_rule" "web-blue" {
  listener_arn = aws_lb_listener.http-listener-blue.arn
  priority = 100
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-blue.arn
  }
  condition {
    path_pattern {
      values = [
        "/healthcheck",
        "/api/*",
        "/admin_users/*",
        "/users/*",
        "/admins",
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.http-listener-blue,
    aws_lb_target_group.ninninjob-web-staging-blue,
  ]
}

resource "aws_lb_listener_rule" "web-green" {
  listener_arn = aws_lb_listener.http-listener-green.arn
  priority = 100
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-green.arn
  }
  condition {
    path_pattern {
      values = [
        "/healthcheck",
        "/api/*",
        "/admin_users/*",
        "/users/*",
        "/admins",
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.http-listener-green,
    aws_lb_target_group.ninninjob-web-staging-green,
  ]
}

# resource "aws_lb_listener_rule" "web-https" {
#   listener_arn = aws_lb_listener.https-listener.arn
#   priority = 100
#   action {
#     type = "forward"
#     target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
#   }
#   condition {
#     path_pattern {
#       values = [
#         "/healthcheck",
#         "/api/*",
#         "/admin_users/*",
#         "/users/*",
#         "/admins",
#       ]
#     }
#   }
#   depends_on = [
#     aws_lb_listener.https-listener,
#     aws_lb_target_group.ninninjob-web-staging,
#   ]
# }

resource "aws_lb_listener_rule" "web-https-blue" {
  listener_arn = aws_lb_listener.https-listener-blue.arn
  priority = 100
  action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-web-staging-green, aws_lb_target_group.ninninjob-web-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-blue.arn
  }
  condition {
    path_pattern {
      values = [
        "/healthcheck",
        "/api/*",
        "/admin_users/*",
        "/users/*",
        "/admins",
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.https-listener-blue,
    aws_lb_target_group.ninninjob-web-staging-blue
  ]
}

resource "aws_lb_listener_rule" "web-https-green" {
  listener_arn = aws_lb_listener.https-listener-green.arn
  priority = 100
  action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-web-staging-green, aws_lb_target_group.ninninjob-web-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-green.arn
  }
  condition {
    path_pattern {
      values = [
        "/healthcheck",
        "/api/*",
        "/admin_users/*",
        "/users/*",
        "/admins",
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.https-listener-green,
    aws_lb_target_group.ninninjob-web-staging-green
  ]
}

# resource "aws_lb_listener_rule" "web2" {
#   listener_arn = aws_lb_listener.http-listener.arn
#   priority = 200
#   action {
#     type = "forward"
#     target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
#   }
#   condition {
#     path_pattern {
#       values = [
#         "/rails/*",
#         "/admins/*",
#         "/sitemap"
#       ]
#     }
#   }
#   depends_on = [
#     aws_lb_listener.http-listener,
#     aws_lb_target_group.ninninjob-web-staging,
#   ]
# }

resource "aws_lb_listener_rule" "web2-blue" {
  listener_arn = aws_lb_listener.http-listener-blue.arn
  priority = 200
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-blue.arn
  }
  condition {
    path_pattern {
      values = [
        "/rails/*",
        "/admins/*",
        "/sitemap"
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.http-listener-blue,
    aws_lb_target_group.ninninjob-web-staging-blue,
  ]
}

resource "aws_lb_listener_rule" "web2-green" {
  listener_arn = aws_lb_listener.http-listener-green.arn
  priority = 200
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-green.arn
  }
  condition {
    path_pattern {
      values = [
        "/rails/*",
        "/admins/*",
        "/sitemap"
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.http-listener-green,
    aws_lb_target_group.ninninjob-web-staging-green,
  ]
}

# resource "aws_lb_listener_rule" "web-https2" {
#   listener_arn = aws_lb_listener.https-listener.arn
#   priority = 200
#   action {
#     type = "forward"
#     target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
#   }
#   condition {
#     path_pattern {
#       values = [
#         "/rails/*",
#         "/admins/*",
#         "/sitemap"
#       ]
#     }
#   }
#   depends_on = [
#     aws_lb_listener.https-listener,
#     aws_lb_target_group.ninninjob-web-staging,
#   ]
# }

resource "aws_lb_listener_rule" "web-https2-blue" {
  listener_arn = aws_lb_listener.https-listener-blue.arn
  priority = 200
  action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-web-staging-green, aws_lb_target_group.ninninjob-web-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-blue.arn
  }
  condition {
    path_pattern {
      values = [
        "/rails/*",
        "/admins/*",
        "/sitemap"
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  }
  depends_on = [
    aws_lb_listener.https-listener-blue,
    aws_lb_target_group.ninninjob-web-staging-blue
  ]
}

resource "aws_lb_listener_rule" "web-https2-green" {
  listener_arn = aws_lb_listener.https-listener-green.arn
  priority = 200
  action {
    type = "forward"
    # forward {
    #   dynamic "target_group" {
    #     for_each = [aws_lb_target_group.ninninjob-web-staging-green, aws_lb_target_group.ninninjob-web-staging-blue]
    #     content {
    #       arn = target_group.value["arn"]
    #     }
    #   }
    # }
    target_group_arn = aws_lb_target_group.ninninjob-web-staging-green.arn
  }
  condition {
    path_pattern {
      values = [
        "/rails/*",
        "/admins/*",
        "/sitemap"
      ]
    }
  }
  lifecycle {
    ignore_changes = [
      action[0].target_group_arn
    ]
  } # For using null_resource to trigger CODEDEPLOY deployment.
  depends_on = [
    aws_lb_listener.https-listener-green,
    aws_lb_target_group.ninninjob-web-staging-green
  ]
}
~~~

Revise Security Group setting:
Add ingress rules for added port.

Add CODEDEPLOY setting to ECS service, revise to use new lb target_group:
staging/ecs_frontend.tf
~~~tf
...
load_balancer {
  target_group_arn = aws_lb_target_group.ninninjob-frontend-staging-green.arn # Will be altered later by CodeDeploy during new deployment. Default green or blue does not make much difference.
}
ordered_placement_strategy {
  type = "binpack"
  field = "memory"
}
deployment_controller {
  type = "CODE_DEPLOY"
}
lifecycle {
  ignore_changes = [
    task_definition,
    load_balancer
  ]
} # For using null_resource to trigger CODEDEPLOY deployment.
...

~~~

Script to trigger CODEDEPLOY deployment:
~~~sh
#!/usr/bin/env bash
set -e

application_name=$1
deployment_group_name=$2
container_name=$3
container_port=$4
task_definition_arn=$5

app_spec_content_string=$(jq -nc \
  --arg container_name "$container_name" \
  --arg container_port "$container_port" \
  --arg task_definition_arn "$task_definition_arn" \
  '{version: 1, Resources: [{TargetService: {Type: "AWS::ECS::Service", Properties: {TaskDefinition: $task_definition_arn, LoadBalancerInfo: {ContainerName: $container_name, ContainerPort: $container_port}}}}]}')
app_spec_content_sha256=$(echo -n "$app_spec_content_string" | shasum -a 256 | sed 's/ .*$//')
revision="revisionType=AppSpecContent,appSpecContent={content='$app_spec_content_string',sha256=$app_spec_content_sha256}"

aws deploy create-deployment \
  --application-name="$application_name" \
  --deployment-group-name="$deployment_group_name" \
  --revision="$revision"

~~~

## Revise to use CODEDEPLOY as deployment-controller:

old:
~~~yml
...
      - aws-ecs/deploy-service-update:
          name: deploy-to-staging
          family: "ninninjob-web-staging"
          cluster-name: "ninninjob-staging"
          skip-task-definition-registration: true
          force-new-deployment: true
          requires:
            - build-staging-image
            - build-staging-batch-image
          filters:
            branches:
              only: staging
...
~~~

new:
~~~yml
...
      - aws-ecs/deploy-service-update:
          name: deploy-to-staging
          family: "ninninjob-web-staging"
          cluster-name: "ninninjob-staging"
          deployment-controller: "CODE_DEPLOY"
          codedeploy-application-name: "ninninjob-web-staging"
          codedeploy-deployment-group-name: "ninninjob-web-staging-group"
          codedeploy-load-balanced-container-name: "ninninjob-web-staging"
          codedeploy-load-balanced-container-port: 3000
          skip-task-definition-registration: true
          force-new-deployment: true
          requires:
            - build-staging-image
            - build-staging-batch-image
          filters:
            branches:
              only: staging
...
~~~


References:
https://medium.com/capital-one-tech/seamless-blue-green-deployment-using-aws-codedeploy-4c36c0bbeef4
https://github.com/hashicorp/terraform-provider-aws/issues/6802

Send notification of Code Deploy result to slack:
References:
https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-sns-event-notifications-create-trigger.html
https://docs.aws.amazon.com/codedeploy/latest/userguide/monitoring-sns-event-notifications-permisssions.html
https://medium.com/@nonb3ing/slack-notifications-for-aws-codedeploy-events-b2a771f29e8d
1. Add SNS permission policy to CodeDeploy operation role.
2. Add a new SNS topic.
3. Add trigger configuration to CodeDeploy deployment group.
4. Add a new channel in Slack, add a new app in Slack, add a new webhook url with the app and channel.
5. Add a new Lambda function using the new webhook url.
6. Testing the Lambda function.

~~~
var https = require('https');
var util = require('util');

const SlackServicePath = "callback url of your specified slack channel";

exports.handler = function (event, context) {
  console.log(JSON.stringify(event, null, 2));
  console.log('From SNS:', event.Records[0].Sns.Message);

  var severity = "good";
  var message = event.Records[0].Sns.Message;
  var messageJSON = JSON.parse(message);
  var subject = event.Records[0].Sns.Subject;

  var postData = {
    channel: "#ambar-test",
    username: "CodeDeploy",
    icon_emoji: ":codedeploy:"
  };

  if (messageJSON.status == "FAILED") {
    severity = "danger"
  }
  if (messageJSON.status == "STOPPED") {
    severity = "warning"
  }

  var fields = [];
  for (var key in messageJSON) {
    if (key == 'deploymentOverview') {
      var value = [];
      var deploymentOverview = JSON.parse(messageJSON[key]);
      for (var status in deploymentOverview) {
        value.push(status + ': ' + deploymentOverview[status]);
      }
      fields.push({
        "title": key
          .replace(/([A-Z])/g, ' $1')
          .replace(/^./, function (str) {
            return str.toUpperCase();
          }),
        "value": value.join(', '),
      });
    } else {
      fields.push({
        "title": key
          .replace(/([A-Z])/g, ' $1')
          .replace(/^./, function (str) {
            return str.toUpperCase();
          }),
        "value": messageJSON[key],
        "short": true
      });
    }
  }

  postData.attachments = [
    {
      "color": severity,
      "fallback": message,
      "title": subject,
      "title_link": "https://console.aws.amazon.com/codedeploy/home?region=" + messageJSON.region + "#/deployments/" + messageJSON.deploymentId,
      "fields": fields
    }
  ];

  var options = {
    method: 'POST',
    hostname: 'hooks.slack.com',
    port: 443,
    path: SlackServicePath
  };

  var req = https.request(options, function (res) {
    res.setEncoding('utf8');
    res.on('data', function (chunk) {
      context.done(null);
    });
  });

  req.on('error', function (e) {
    console.log('problem with request: ' + e.message);
  });

  req.write(util.format("%j", postData));
  req.end();
};
~~~
