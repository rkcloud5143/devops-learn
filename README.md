# DevOps / AWS Cloud Engineer — 3-Month Study Guide

## Who is this for?
QA Engineer (6 yrs) transitioning to AWS Cloud/DevOps Engineer in Canada.

## Folder Structure (by Topic — read files in numbered order)
```
devops-study-guide/
├── README.md                              ← You are here
│
├── 01-linux/
│   ├── 01-essential-commands.md           ← START HERE — what & why
│   ├── 02-filesystem-navigation.md        ← File system, find, grep, pipes
│   ├── 03-users-groups-permissions.md     ← Users, chmod, chown, SSH, sudo
│   ├── 04-process-management.md           ← ps, top, kill, systemctl
│   ├── 05-networking.md                   ← IP, ports, curl, firewall
│   ├── 06-disk-storage.md                 ← df, du, mount, fstab, swap
│   ├── 07-package-management.md           ← apt, yum, apk
│   ├── 08-bash-scripting.md               ← Variables, loops, functions
│   └── 09-cron-jobs.md                    ← Crontab, systemd timers
│
├── 02-networking/
│   ├── 01-fundamentals.md                 ← DNS, TCP/IP, how internet works
│   ├── 02-cidr-and-subnets.md             ← IP addresses, CIDR notation
│   └── 03-ports.md                        ← Common ports to memorize
│
├── 03-git/
│   ├── 01-commands.md                     ← Daily git workflow
│   └── 02-pull-request-workflow.md        ← Branching & PR process
│
├── 04-aws/
│   ├── 01-global-infrastructure.md        ← Regions, AZs
│   ├── 02-iam.md                          ← Users, roles, policies
│   ├── 03-vpc.md                          ← Subnets, IGW, NAT, SG, NACL
│   ├── 04-ec2.md                          ← Instances, EIP, ENI, EBS, pricing
│   ├── 05-s3.md                           ← Storage classes, encryption
│   ├── 06-rds.md                          ← Multi-AZ, replicas, Aurora
│   ├── 07-elb-autoscaling.md              ← ALB, NLB, ASG, scaling
│   ├── 08-route53.md                      ← Public/private DNS, routing
│   ├── 09-cloudwatch.md                   ← Metrics, logs, alarms
│   ├── 10-cloudtrail.md                   ← API audit, security
│   ├── 11-lambda-serverless.md            ← Lambda, API GW, DynamoDB
│   ├── 12-sns.md                          ← Topics, fan-out, filtering
│   ├── 13-sqs.md                          ← Queues, DLQ, visibility timeout
│   ├── 14-kms.md                          ← Keys, encryption, Secrets Mgr
│   ├── 15-ses.md                          ← Email service, SPF/DKIM
│   ├── 16-kinesis.md                      ← Streams, Firehose
│   ├── 17-networking-tgw-vpn-dx.md        ← TGW, VPN, Direct Connect
│   ├── 18-aws-cli.md                      ← CLI commands reference
│   └── 19-certification-tips.md           ← CCP & SAA exam strategy
│
├── 05-docker/
│   ├── 01-fundamentals.md                 ← Images, containers, Dockerfile
│   ├── 02-commands.md                     ← Essential docker commands
│   ├── 03-docker-compose.md               ← Multi-container apps
│   └── 04-aws-containers.md               ← ECR, ECS, EKS overview
│
├── 06-terraform/
│   ├── 01-fundamentals.md                 ← Why IaC, workflow, state
│   ├── 02-hcl-basics.md                   ← Provider, resources, variables
│   ├── 03-state-management.md             ← Remote state, S3 backend
│   └── 04-commands.md                     ← Essential terraform commands
│
├── 07-cicd/
│   ├── 01-fundamentals.md                 ← CI vs CD concepts
│   └── 02-github-actions.md               ← Workflows, jobs, secrets
│
├── 08-kubernetes/
│   ├── 01-fundamentals.md                 ← Architecture, pods, nodes
│   ├── 02-objects.md                      ← Deployments, services, configmaps
│   ├── 03-commands.md                     ← Essential kubectl commands
│   └── 04-eks.md                          ← AWS managed Kubernetes
│
├── 09-projects/
│   ├── 01-project-terraform-infra.md      ← Full VPC + EC2 + RDS + ALB
│   ├── 02-project-container-cicd.md       ← Docker + GitHub Actions + ECS
│   └── 03-project-serverless.md           ← Lambda + API GW + DynamoDB
│
├── 10-interview-prep/
│   ├── 01-technical-questions.md          ← AWS, Terraform, Docker, K8s
│   ├── 02-troubleshooting-scenarios.md    ← Real-world debugging
│   ├── 03-behavioral-questions.md         ← QA → DevOps story (STAR)
│   └── 04-job-search-canada.md            ← Where to apply, LinkedIn tips
│
└── diagrams/
    └── architecture-reference.md          ← 6 architecture diagrams

```

## Study Order
| Week  | Topics                                           |
|-------|--------------------------------------------------|
| 1-2   | 01-linux → 02-networking → 03-git                |
| 3-4   | 04-aws (all 19 files) + Cloud Practitioner exam  |
| 5-6   | 05-docker → 06-terraform                         |
| 7-8   | 07-cicd → 08-kubernetes + SAA exam               |
| 9-10  | 09-projects (build all 3)                        |
| 11-12 | 10-interview-prep + start applying               |

## Daily Schedule (Full-Time)
| Time           | Activity                                  |
|----------------|-------------------------------------------|
| 9:00 - 11:00   | Video course / reading                    |
| 11:00 - 13:00  | Hands-on labs (AWS console/CLI)           |
| 13:00 - 14:00  | Lunch break                               |
| 14:00 - 16:00  | Build / practice (Terraform, Docker, projects) |
| 16:00 - 17:00  | Notes, flashcards, review                 |

## Key Resources
| Topic       | Resource                                         |
|-------------|--------------------------------------------------|
| AWS SAA     | Adrian Cantrill or Stephane Maarek (Udemy)       |
| Terraform   | developer.hashicorp.com/terraform/tutorials      |
| Docker      | docs.docker.com/get-started                      |
| Kubernetes  | TechWorld with Nana (YouTube)                    |
| Linux       | linuxjourney.com                                 |
| Mock Exams  | Tutorials Dojo (tutorialsdojo.com)               |
