---
layout: post
title: "DNS ssl certificate setting"
description: ""
categories: [devops]
tags: [Terraform, AWS, DNS]
redirect_from:
  - /2019/09/20/
---

acm_certificate.tf
~~~
# Remove from terraform management (Staging tends to be rebuild and destroy)
# resource "aws_acm_certificate" "ninninjob-staging-certificate" {
#   domain_name = var.domain
#   subject_alternative_names = ["*.${var.domain}"]
#   validation_method = "DNS"
#   lifecycle {
#     create_before_destroy = true
#   }
# }

# resource "aws_acm_certificate_validation" "ninninjob-staging-validation" {
#   certificate_arn = aws_acm_certificate.ninninjob-staging-certificate.arn
#   validation_record_fqdns = [
#     aws_route53_record.validation.fqdn,
#   ]
# }

~~~

data.tf
~~~
data "aws_acm_certificate" "ninninjob-certificate" {
  domain = var.domain
}

data "aws_route53_zone" "public" {
  name = var.domain
}
~~~

route53.tf
~~~
# Remove from terraform management (Staging tends to be rebuild and destroy)
# resource "aws_route53_zone" "public" {
#   name = var.domain
# }

# resource "aws_route53_record" "validation" {
#   zone_id = aws_route53_zone.public.zone_id
#   name = aws_acm_certificate.ninninjob-staging-certificate.domain_validation_options.0.resource_record_name
#   type = aws_acm_certificate.ninninjob-staging-certificate.domain_validation_options.0.resource_record_type
#   records = [aws_acm_certificate.ninninjob-staging-certificate.domain_validation_options.0.resource_record_value]
#   ttl = 60
# }

# default ttl is 0
resource "aws_route53_record" "ninninjob-staging-alb" {
  zone_id = data.aws_route53_zone.public.zone_id
  name = "staging.${var.domain}"
  type = "A"
  alias {
    name = aws_lb.ninninjob-staging-alb.dns_name
    zone_id = aws_lb.ninninjob-staging-alb.zone_id
    evaluate_target_health = false
  }
}
~~~

Add a https listener and its rules to lb:

lb.tf
~~~
resource "aws_lb_listener" "https-listener" {
  load_balancer_arn = aws_lb.ninninjob-staging-alb.arn
  port = "443"
  protocol = "HTTPS"
  ssl_policy = "ELBSecurityPolicy-2016-08"
  # certificate_arn = aws_acm_certificate.ninninjob-staging-certificate.arn
  certificate_arn = data.aws_acm_certificate.ninninjob-certificate.arn
  default_action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-frontend-staging.arn
  }
}

resource "aws_lb_listener_rule" "web-https" {
  listener_arn = aws_lb_listener.https-listener.arn
  priority = 100
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
  }
  condition {
    path_pattern {
      values = [
        "/healthcheck",
        "/api/*",
        "/admin_users/*",
        "/users/*",
        "/admins/*",
      ]
    }
  }
  depends_on = [
    aws_lb_listener.https-listener,
    aws_lb_target_group.ninninjob-web-staging,
  ]
}

resource "aws_lb_listener_rule" "web-https2" {
  listener_arn = aws_lb_listener.https-listener.arn
  priority = 200
  action {
    type = "forward"
    target_group_arn = aws_lb_target_group.ninninjob-web-staging.arn
  }
  condition {
    path_pattern {
      values = [
        "/rails/*"
      ]
    }
  }
  depends_on = [
    aws_lb_listener.https-listener,
    aws_lb_target_group.ninninjob-web-staging,
  ]
}

~~~

Check if security groups already allowed 443 port

Because the domain is not purchased from AWS, so will need some setting in the external DNS server admin console:

1. Verify the domain:
Add the validation CNAME record in external DNS server admin console. -> (This seems not necessary?)

2. Change the name server setting in external DNS server admin console.
Check AWS NS record in the target hosted zone for the 4 name servers.
Add them to the external DNS server admin console.

3. Wait for the change to become valid. (Can take a few hours up to 72 hours)


