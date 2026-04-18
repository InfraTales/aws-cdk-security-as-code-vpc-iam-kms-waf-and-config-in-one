# Architecture Notes

## Overview

The stack provisions a multi-AZ VPC with strict subnet isolation — public, private, and isolated tiers — then places EC2 workloads exclusively in private subnets behind security groups generated from a central SecurityConfig class. KMS customer-managed keys encrypt S3, Secrets Manager, and CloudTrail at rest, with CloudTrail writing to a locked-down S3 bucket and shipping to CloudWatch Logs for near-real-time alerting. AWS Config records resource-level changes and enforces managed rules, WAF protects any public-facing endpoints, and all credentials are vended through Secrets Manager rather than environment variables — every one of these controls is expressed as CDK TypeScript constructs so the security posture is reproducible, diffable, and PR-reviewable.

## Key Decisions

- Customer-managed KMS keys add ~$1/key/month plus $0.03 per 10,000 API calls — trivial at low volume but can become a surprise line item when CloudTrail, Config, and S3 all generate high call rates at scale [inferred]
- AWS Config continuous recording costs ~$0.003 per configuration item; recording every resource change in a busy account with hundreds of EC2s and frequent deployments can push Config costs past $50-100/month before you notice [inferred]
- Placing all EC2 instances in private subnets with no NAT Gateway saves NAT processing costs but requires VPC endpoints for every AWS service the instances call — missed endpoints cause silent connectivity failures that are hard to debug [from-code]
- Centralising security config in a single SecurityConfig class makes the design consistent but also means a wrong CIDR or prefix there propagates to every subnet, security group, and KMS key policy simultaneously — a high-blast-radius single point of change [from-code]
- WAF Web ACLs on a regional basis cost $5/month per ACL plus $1 per rule group and $0.60 per million requests — attaching WAF to every ALB in a multi-environment setup can add $30-60/month per environment [inferred]