# AWS CDK — Infrastructure as Code with Programming Languages ☁️

Write infrastructure in TypeScript, Python, Java — not YAML.

---

## What is CDK?

```
CloudFormation = Infrastructure as YAML/JSON (verbose, hard to read)
Terraform      = Infrastructure as HCL (declarative, multi-cloud)
CDK            = Infrastructure as CODE (TypeScript, Python, Java, Go, C#)

CDK compiles your code → CloudFormation template → deploys to AWS.

# 50 lines of CloudFormation YAML becomes:
const bucket = new s3.Bucket(this, 'MyBucket', {
  versioned: true,
  encryption: s3.BucketEncryption.S3_MANAGED,
});
```

---

## CDK vs Terraform

| Feature | CDK | Terraform |
|---------|-----|-----------|
| Language | TypeScript, Python, Java, Go | HCL |
| Cloud support | AWS only | Multi-cloud |
| State | CloudFormation manages | Terraform state file |
| Loops/conditions | Native programming | HCL (limited) |
| Testing | Jest, pytest | Terratest |
| Learning curve | Know a language? Easy | Learn HCL |
| Community | Growing | Massive |

**Use CDK** if: AWS-only, team knows TypeScript/Python, want real programming.
**Use Terraform** if: Multi-cloud, team knows HCL, want ecosystem.

---

## Quick Example (TypeScript)

```typescript
import * as cdk from 'aws-cdk-lib';
import * as s3 from 'aws-cdk-lib/aws-s3';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as ecs from 'aws-cdk-lib/aws-ecs';

export class MyAppStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string) {
    super(scope, id);

    // VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', { maxAzs: 2 });

    // ECS Cluster
    const cluster = new ecs.Cluster(this, 'MyCluster', { vpc });

    // Fargate Service with ALB — ONE construct!
    new ecs.patterns.ApplicationLoadBalancedFargateService(this, 'MyApp', {
      cluster,
      taskImageOptions: {
        image: ecs.ContainerImage.fromRegistry('nginx'),
      },
      desiredCount: 2,
      publicLoadBalancer: true,
    });
  }
}
```

That single `ApplicationLoadBalancedFargateService` creates: ALB, target group, ECS service, task definition, security groups, IAM roles — all wired together.

---

## Commands

```bash
npm install -g aws-cdk                 # Install
cdk init app --language typescript     # New project
cdk synth                              # Generate CloudFormation
cdk diff                               # Preview changes
cdk deploy                             # Deploy
cdk destroy                            # Tear down
cdk list                               # List stacks
cdk bootstrap                          # One-time setup per account/region
```

---

*CDK = real programming for infrastructure. Powerful if you're AWS-only! ☁️*
