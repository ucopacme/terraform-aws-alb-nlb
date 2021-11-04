# terraform-aws-alb-nlb
Terraform AWS alb/nbl Module


-->

Terraform module to provision AWS [`lb`]



## Introduction

The module will create:

* ALB
* NLB
* LB target Groups
* LB listeners
* LB listeners rule
* LB listeners condition



## Listeners rules action

|listners rules  |
|----------------|
| Redirect       |
| fixed-response |
| forward        | 

## Listeners rule condations

|listners condations |
|--------------------|
| Path Pattern       |
| Host header        |
| Http header        | 
| Http request method| 
| Query string       | 
| Source IP address  | 

## Usage 

Complete example of ALB creation with all Rules/condation

Create terragrunt.hcl config file, copy/past the following configuration.


```hcl

#
# Include all settings from root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}

inputs = {
  enabled            = "true"
  load_balancer_type = "application"
  vpc_id             = "vpc-xxxx"
  subnets            = ["subnet-xxxxxx", "subnet-xxxxx"]
  security_groups    = ["sg-group", "sg-group"]
  access_logs = {
    bucket = "kk-test-alb"
  }
  # Creates Target group
  target_groups = [
    {
      backend_protocol = "HTTPS"
      backend_port     = 443
      target_type      = "instance"
      stickiness = {
        enabled         = "true"
        cookie_duration = "86400"
        type            = "app_cookie" # change to "lb_cookie" for lb cookies type
        cookie_name     = "Cookie"

      }
      health_check = {
        enabled             = "true"
        interval            = 30
        path                = "/index.html"
        port                = "traffic-port"
        healthy_threshold   = 5
        unhealthy_threshold = 2
        timeout             = 5
        protocol            = "HTTPS"
      }
      # Add instances to target groups
      targets = [
        {
          target_id = "Instance ID"
          port      = 80
        },
        {
          target_id = "Instance ID"
          port      = 8080
        }
      ]
    },
    {
      backend_protocol = "HTTP"
      backend_port     = 80
      target_type      = "instance"
      stickiness = {
        enabled         = "true"
        cookie_duration = "172800"
        type            = "app_cookie" # change to "lb_cookie" for lb cookies type
        cookie_name     = "Cookie"
      }
      # Add instances to target groups
      targets = [
        {
          target_id = "Instance ID"
          port      = 80
        },
        {
          target_id = "Instance ID"
          port      = 80
        }
      ]
    }
  ]
  # Create HTTPs listners
  https_listeners = [
    {
      port               = 443
      protocol           = "HTTPS"
      certificate_arn    = "arn:aws:iam::xxxxxxx:server-certificate/my-server-test"
      target_group_index = 0
      ssl_policy         = "ELBSecurityPolicy-TLS-1-2-Ext-2018-06"
    }
  ]
  # Create HTTPs Rules
  https_listener_rules = [
    {
      https_listener_index = 0 # changed the index number to point the listener to different target group.
      priority             = 5000

      actions = [{
        type        = "redirect"
        status_code = "HTTP_302"
        host        = "www.youtube.com"
        path        = "/watch"
        query       = "v=dQw4w9WgXcQ"
        protocol    = "HTTPS"
      }]
  # Create HTTPS rule conditions
      conditions = [{
        path_patterns = ["/onboarding", "/docs"]
      }]
    },
    {
      https_listener_index = 0
      priority             = 5
      actions = [
        {
          type               = "forward"
          target_group_index = 0 # change the index number to point the action to different target group
        }
      ]
      conditions = [{
        source_ips = ["129.48.0.0/16"]
      }]

    },
   {
      https_listener_index = 0
      priority             = 9
      actions = [
        {
          type               = "forward"
          target_group_index = 0 # change the index number to point the action to different target group
        }
      ]

      conditions = [{
        http_headers = [{
          http_header_name = "X-Forwarded-For"
          values           = ["10.0.0.1"]
        }]
      }]
    },
    {
      https_listener_index = 0
      priority             = 5009

      actions = [{
        type        = "redirect"
        status_code = "HTTP_302"
        # host        = "www.youtube.com"
        # path        = "/watch"
        # query       = "v=dQw4w9WgXcQ"
        protocol    = "HTTPS"
        port = "8080"
      }]

      conditions = [{
        http_headers = [{
          http_header_name = "X-Forwarded-For"
          values           = ["10.0.0.1","129.48.0.0/16"]
        }]
      }]
    },
    {
      https_listener_index = 0
      priority             = 3
      actions = [{
        type         = "fixed-response"
        content_type = "text/plain"
        status_code  = 200
        message_body = "This is a fixed response"
      }]

      conditions = [{
        http_headers = [{
          http_header_name = "Fixed-Response"
          values           = ["10.0.0.1", "please"]
        }]
      }]
    },

  ]

# Create http listeners
  http_tcp_listeners = [
    {
      port               = 80
      protocol           = "HTTP"
      target_group_index = 0
    }

  ]

  name = join("-", [local.application, local.environment
  ])
  tags = {
    "ucop:application" = local.application
    "ucop:createdBy"   = local.createdBy
    "ucop:environment" = local.environment
    "ucop:group"       = local.group
    "ucop:source"      = local.source
  }
}


locals {
  application = "kkapp"
  createdBy   = "terraform"
  environment = "dev"
  group       = "chs"
  source      = join("/", ["https://github.com/ucopacme/ucop-terraform-config/tree/master/terraform/its-chs-dev/us-west-2", path_relative_to_include()])

}


terraform {
  source = "git::https://git@github.com/ucopacme/terraform-aws-alb-nlb//?ref=v0.0.4"
}

```

 
 
## Usage

Complete example of NLB creation

Create terragrunt.hcl config file, copy/past the following configuration.


```hcl


#
# Include all settings from root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}

inputs = {
  enabled            = "true"
  load_balancer_type = "network"
  vpc_id             = "vpc-xxxx"
  subnets            = ["subnet-xxx", "subnet-xxx"]
  # security_groups    = ["sg-xxxxx", "sg-xxxx"]

  access_logs = {
    bucket = "my-nlb-logs"
  }

  target_groups = [
    {
      backend_protocol = "TCP"
      backend_port     = 80
      target_type      = "ip"
    }
  ]

  https_listeners = [
    {
      port               = 443
      protocol           = "TLS"
      certificate_arn    = "arn:aws:iam::xxxxxxx:server-certificate/my-server-test"
      target_group_index = 0
    }
  ]

  http_tcp_listeners = [
    {
      port               = 80
      protocol           = "TCP"
      target_group_index = 0
    }
  ]


  name = join("-", [local.application, local.environment
  ])
  tags = {
    "ucop:application" = local.application
    "ucop:createdBy"   = local.createdBy
    "ucop:environment" = local.environment
    "ucop:group"       = local.group
    "ucop:source"      = local.source
  }
}


locals {
  application = "kkapp-nlb"
  createdBy   = "terraform"
  environment = "dev"
  group       = "chs"
  source      = join("/", ["https://github.com/ucopacme/ucop-terraform-config/tree/master/terraform/its-chs-dev/us-west-2", path_relative_to_include()])

}


terraform {
  source = "git::https://git@github.com/ucopacme/terraform-aws-alb-nlb//?ref=v0.0.4"
}
