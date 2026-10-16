# AWS E-commerce Ordering Platform — Architecture Case Study

> **What this repo is:** the architecture, infrastructure walkthrough, and evidence for a cloud-native ordering platform I designed and deployed on AWS as the capstone for my B.S. in Cloud Computing (Purdue Global, 2026). It is a documentation-and-evidence repo: the AWS environment was built and configured by hand in the console and verified end to end. The application container source is not included here.

## What was built

A food-distribution ordering system: customers place orders, choose delivery preferences, and track status. The backend API runs as a container on **Amazon ECS** behind an **Application Load Balancer**, stores orders in **Amazon RDS (MySQL)** inside a private subnet, and publishes order events to **Amazon SQS** so downstream systems (inventory, delivery) consume them asynchronously instead of being called directly.

```
Customer → ALB (HTTP→HTTPS redirect) → ECS service (API container)
                                             │
                                             ├─→ RDS MySQL (private subnet, SG-restricted)
                                             │     secrets via AWS Secrets Manager
                                             └─→ SQS queue ──→ inventory / delivery consumers
Observability: CloudWatch log groups + alarm · CloudTrail (multi-region, S3)
```

## Design decisions

- **Private data tier.** RDS sits in private subnets behind a dedicated database security group; a NAT gateway gives private resources outbound access without inbound exposure.
- **Event-driven integration.** The API never calls inventory or delivery systems directly. It publishes an order event to SQS, which decouples the systems, absorbs bursts, and lets consumers fail and retry without affecting order placement.
- **No secrets in config.** Database credentials are stored in AWS Secrets Manager rather than in the container image or environment.
- **Managed everything.** ECS (not self-managed EC2 hosts), RDS with automated backups, ALB health checks, so the operational surface is small for a single builder.
- **Logging first.** CloudTrail delivering to S3 was enabled before the first service deployment; CloudWatch log groups and an alarm cover the running service.

## Evidence

Six screenshots that show the system working, in deployment order. The full set (47) is in [`screenshots /`](screenshots%20/).

| Step | Evidence |
|---|---|
| 1. VPC with public/private subnets, IGW, NAT, route tables | ![VPC](screenshots%20/U3-Step4-VPC.png) |
| 2. ECS cluster created | ![ECS cluster](screenshots%20/U4-01-ECS-Cluster-Created.png) |
| 3. ALB active with listener and target group | ![ALB](screenshots%20/Unit4_ALB_Active_Listener_TargetGroup.png) |
| 4. Orders schema and sample queries (customers, orders, order_items) | ![Schema](screenshots%20/U5_databaseTables.png) |
| 5. Order events received on the SQS queue | ![SQS](screenshots%20/U6_SQSMsgsReceivedSeccuess.png) |
| 6. HTTP→HTTPS redirect rule on the listener | ![Redirect](screenshots%20/U7%20http%20to%20https%20redirect.png) |

## Documentation

- [`architecture/`](architecture/README.md) — layers, components, and how they communicate
- [`infrastructure/`](infrastructure/README.md) — VPC, security groups, ECS, RDS, SQS configuration and the deployment order
- [`api/`](api/README.md) — REST endpoints (create order, get order, update delivery preference, order status)
- [`integrations/`](integrations/README.md) — the SQS event flow and the simulated inventory consumer

## What I would do differently

- **Infrastructure as code.** Everything here was clicked together in the console. The next iteration is a CloudFormation or Terraform definition so the environment is reproducible and reviewable.
- **A build pipeline.** Image builds, the push to ECR, and ECS service rollouts were manual; a CI/CD pipeline would make deployments repeatable.
- **A dead-letter queue.** Failed inventory events currently have nowhere to go.
- **Automated tests against the API** rather than manual Postman checks.

## What went wrong and how it was fixed

The main problems were VPC routing (Internet Gateway attachment and route-table configuration), ECS service connectivity to the database, and security-group rules blocking API traffic. Each was resolved by correcting the network and security-group configuration and re-verifying connectivity from an EC2 instance inside the VPC before retesting the API.
