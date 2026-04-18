# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- Customer-managed KMS keys add ~$1/key/month plus $0.03 per 10,000 API calls — trivial at low volume but can become a surprise line item when CloudTrail, Config, and S3 all generate high call rates at scale [inferred]
- AWS Config continuous recording costs ~$0.003 per configuration item; recording every resource change in a busy account with hundreds of EC2s and frequent deployments can push Config costs past $50-100/month before you notice [inferred]
- Placing all EC2 instances in private subnets with no NAT Gateway saves NAT processing costs but requires VPC endpoints for every AWS service the instances call — missed endpoints cause silent connectivity failures that are hard to debug [from-code]
- WAF Web ACLs on a regional basis cost $5/month per ACL plus $1 per rule group and $0.60 per million requests — attaching WAF to every ALB in a multi-environment setup can add $30-60/month per environment [inferred]

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
