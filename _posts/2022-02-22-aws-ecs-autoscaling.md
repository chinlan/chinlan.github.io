---
layout: post
title: "AWS: adding autoscaling settings for ECS containers"
description: ""
categories: [devops]
tags: [AWS]
redirect_from:
  - /2022/02/22/
---

[Reference 1](https://towardsaws.com/aws-ecs-service-autoscaling-terraform-included-d4b46997742b)
[Reference 2](https://dev.to/kieranjen/ecs-fargate-service-auto-scaling-with-terraform-2ld)

### TODOs

ecs_web.tf
```tf
resource "aws_ecs_service" "web-production" {
  ...
  lifecycle {
    ignore_changes = [
      task_definition,
      load_balancer,
+     desired_count # Preserve desired count when updating an autoscaled ECS Service
    ]
  }

}
```

ecs_web_autoscaling.tf
```tf
resource "aws_appautoscaling_target" "production-ecs" {
  max_capacity       = 5
  min_capacity       = 2
  resource_id        = "service/${aws_ecs_cluster.production.name}/${aws_ecs_service.web-production.name}"
  scalable_dimension = "ecs:service:DesiredCount"
  service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "production-ecs-cpu" {
  name               = "production-ecs-autoscaling-policy-cpu"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.production-ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.production-ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.production-ecs.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    target_value = 80
  }
  depends_on = [aws_appautoscaling_target.production-ecs]
}
resource "aws_appautoscaling_policy" "production-ecs-memory" {
  name               = "production-ecs-autoscaling-policy-memory"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.production-ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.production-ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.production-ecs.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
    target_value = 80
  }
  depends_on = [aws_appautoscaling_target.production-ecs]
}

```
