# AWS Practice Questions 📝

Easy questions to test your understanding. Answers are at the bottom — no peeking!

---

## Compute

### EC2
1. You need a server running 24/7 to host a website. Which service do you use?
2. Who is responsible for patching and updating the operating system on an EC2 instance — you or AWS?
3. Can you change the size (CPU/RAM) of an EC2 instance after creating it?

### Lambda
1. You want to resize an image every time it's uploaded to S3, but you don't want to manage any servers. What do you use?
2. If your Lambda function runs 0 times this month, how much do you pay?
3. What's the difference between Lambda and EC2 in terms of when they run?

### Elastic Beanstalk
1. You have a web app and don't want to deal with setting up servers, load balancers, or scaling. What service helps?
2. Does Elastic Beanstalk create EC2 instances behind the scenes?
3. True or false: Elastic Beanstalk only works with Java applications.

### ECS
1. You have a Docker container and want AWS to run it for you using AWS's own system. What service do you use?
2. What's the difference between ECS and just running Docker on an EC2 instance yourself?
3. Can ECS automatically restart a container if it crashes?

### EKS
1. Your team already uses Kubernetes. Which AWS service lets you run Kubernetes without managing the control plane?
2. What's the main difference between ECS and EKS?
3. Put these in order from smallest to biggest: EKS cluster → Container → Pod

---

## Storage

### S3
1. You need to store 10,000 photos for a website. Which service is the best fit?
2. What is an S3 "bucket"?
3. True or false: S3 has a storage limit of 5 TB.

### EBS
1. Your EC2 instance needs a hard drive. What service provides that?
2. Can two EC2 instances share the same EBS volume at the same time?
3. What happens to an EBS volume if you terminate the EC2 instance it's attached to? (Trick question — it depends!)

### EFS
1. Three EC2 instances need to read and write the same files. S3 won't work because the app needs a regular file system. What do you use?
2. What's the key difference between EBS and EFS?
3. True or false: EFS automatically grows and shrinks as you add or remove files.

---

## Database

### RDS
1. You need a MySQL database but don't want to handle backups, patching, or replication. What do you use?
2. Name two database engines that RDS supports.
3. True or false: With RDS, you still have to manually install the database software.

### DynamoDB
1. You need a database that can handle millions of reads per second with single-digit millisecond response times. SQL or DynamoDB?
2. Is DynamoDB a SQL (relational) or NoSQL database?
3. Do you need to manage servers when using DynamoDB?

### ElastiCache
1. Your database is getting slow because the same data is being queried over and over. What can you put in front of it to speed things up?
2. Name the two engines ElastiCache supports.
3. Where does ElastiCache store data — on disk or in memory?

---

## Networking & Content Delivery

### VPC
1. You want to isolate your production servers from your development servers in AWS. What do you use?
2. What are subnets in a VPC?
3. True or false: Every AWS account comes with a default VPC.

### Route 53
1. You bought the domain "myapp.com" and need to point it to your EC2 instance. What AWS service handles this?
2. What does DNS stand for?
3. Why is it called "Route 53"? (Hint: it's a port number.)

### CloudFront
1. Your website is hosted in the US, but users in India say it's slow. What service can help?
2. What is an "edge location"?
3. True or false: CloudFront can only serve static files like images and CSS.

### ELB
1. You have 5 EC2 instances running your app. How do you make sure traffic is spread evenly across all 5?
2. If one of the 5 instances goes down, what does the load balancer do?
3. True or false: A load balancer can also handle HTTPS certificates.

### API Gateway
1. You built a Lambda function and want to expose it as a REST API. What service sits in front of it?
2. Can API Gateway limit how many requests a user can make per second?
3. Name two things API Gateway handles besides routing.

---

## Security & Identity

### IAM
1. A new developer joins your team. They should only be able to read S3 buckets, nothing else. How do you set this up?
2. What's the difference between an IAM user and an IAM role?
3. True or false: The root account should be used for everyday tasks.

### KMS
1. You want to encrypt data stored in S3. What service manages the encryption keys?
2. Who controls who can use a KMS key — KMS itself or IAM policies?
3. What's the difference between encryption at rest and encryption in transit?

### WAF
1. Your website is getting hit with SQL injection attacks. What AWS service can block them?
2. Does WAF protect against DDoS attacks?
3. True or false: WAF works with CloudFront and API Gateway.

### Shield
1. Your website is being flooded with millions of fake requests trying to take it down. What's this attack called, and what AWS service protects against it?
2. What's the difference between Shield Standard and Shield Advanced?
3. True or false: Shield Standard is free and automatically enabled.

---

## Messaging & Integration

### SQS
1. Service A produces tasks faster than Service B can process them. What do you put between them?
2. What happens to a message in SQS if nobody picks it up?
3. What's the difference between a standard queue and a FIFO queue?

### SNS
1. When a new user signs up, you want to send them an email AND trigger a Lambda function AND add a message to an SQS queue. What service can do all three with one publish?
2. What is an SNS "topic"?
3. True or false: SNS guarantees messages are delivered in order.

### SES
1. Your app needs to send password reset emails to users. What AWS service do you use?
2. Can SES also receive emails, or only send them?
3. Name two types of emails SES is commonly used for.

---

## Monitoring & Management

### CloudWatch
1. You want to get an alert when your EC2 instance's CPU goes above 90%. What service do you use?
2. What's the difference between CloudWatch Logs and CloudWatch Metrics?
3. Can CloudWatch trigger a Lambda function when an alarm goes off?

### CloudTrail
1. Someone deleted an S3 bucket and you need to find out who. Where do you look?
2. What's the difference between CloudWatch and CloudTrail?
3. True or false: CloudTrail is enabled by default in every AWS account.

### CloudFormation
1. You need to create the exact same infrastructure in 3 different AWS regions. What service helps you do this repeatably?
2. What format are CloudFormation templates written in?
3. If you delete a CloudFormation stack, what happens to the resources it created?

---

## Scenario Questions 🍕

These combine multiple services — just like real life!

1. A user uploads a photo to S3. You want to automatically create a thumbnail. Which two services work together here?

2. Your app has a web frontend, an API, and a database. The frontend should be fast globally, the API should scale automatically, and the database should be managed. Pick one service for each.

3. You're deploying a new version of your app and want to make sure no alerts fire during the 30-minute maintenance window. What do you do?

4. A hacker is sending thousands of requests with malicious SQL in the URL. Which two services help protect you?

5. Your team wants to know every time someone changes an IAM policy. How do you set this up?

---

# Answers 🔑

<details>
<summary>Click to reveal answers</summary>

## Compute

**EC2:** 1) EC2. 2) You — EC2 gives you the server, you manage the OS. 3) Yes — stop the instance, change the instance type, start it again.

**Lambda:** 1) Lambda. 2) Nothing — $0. You only pay when it runs. 3) EC2 runs all the time; Lambda runs only when triggered.

**Elastic Beanstalk:** 1) Elastic Beanstalk. 2) Yes — it provisions EC2, ELB, and other resources for you. 3) False — it supports many languages (Python, Node.js, Java, Go, .NET, etc.).

**ECS:** 1) ECS. 2) ECS handles scheduling, scaling, and health checks automatically; doing it yourself means manual management. 3) Yes — ECS monitors containers and restarts failed ones.

**EKS:** 1) EKS. 2) ECS is AWS-native and simpler; EKS runs standard Kubernetes. 3) Container (your app) → Pod (one or more containers) → EKS cluster (the whole system).

## Storage

**S3:** 1) S3. 2) A container (folder) that holds your files (objects). 3) False — S3 has virtually unlimited storage. Individual objects max at 5 TB.

**EBS:** 1) EBS. 2) No — standard EBS attaches to one instance at a time (multi-attach exists for specific types but is rare). 3) It depends on the "Delete on Termination" setting — it can be kept or deleted.

**EFS:** 1) EFS. 2) EBS attaches to one instance; EFS is shared across many instances. 3) True.

## Database

**RDS:** 1) RDS. 2) Any two of: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora. 3) False — RDS handles installation and management for you.

**DynamoDB:** 1) DynamoDB. 2) NoSQL. 3) No — DynamoDB is fully serverless.

**ElastiCache:** 1) ElastiCache. 2) Redis and Memcached. 3) In memory — that's why it's fast.

## Networking & Content Delivery

**VPC:** 1) Separate VPCs (or subnets within a VPC). 2) Smaller network segments inside a VPC — like rooms inside a building. 3) True.

**Route 53:** 1) Route 53. 2) Domain Name System. 3) DNS uses port 53.

**CloudFront:** 1) CloudFront. 2) A data center close to users where cached content is served from. 3) False — it can also serve dynamic content and stream video.

**ELB:** 1) Put an ELB in front of them. 2) It stops sending traffic to the unhealthy instance and routes to the healthy ones. 3) True — ELB can terminate SSL/TLS.

**API Gateway:** 1) API Gateway. 2) Yes — this is called throttling. 3) Any two of: authentication, caching, request validation, CORS, monitoring.

## Security & Identity

**IAM:** 1) Create an IAM user, attach a policy that allows s3:Get* and s3:List* only. 2) A user is a person with permanent credentials; a role is temporary permissions assumed by users or services. 3) False — use the root account only for initial setup; use IAM users for daily work.

**KMS:** 1) KMS. 2) IAM policies (and KMS key policies) control access. 3) At rest = data sitting on disk (encrypted by KMS). In transit = data moving over the network (encrypted by TLS/HTTPS).

**WAF:** 1) WAF. 2) WAF helps with layer 7 (application) attacks but Shield is specifically for DDoS. 3) True.

**Shield:** 1) DDoS (Distributed Denial of Service); Shield. 2) Standard is free and automatic; Advanced costs more but gives 24/7 support, advanced detection, and cost protection. 3) True.

## Messaging & Integration

**SQS:** 1) An SQS queue. 2) It stays in the queue until it expires (default: 4 days, max: 14 days). 3) Standard = high throughput, possible duplicates, no guaranteed order. FIFO = exactly-once, guaranteed order, lower throughput.

**SNS:** 1) SNS — publish once to a topic, all subscribers get it. 2) A channel that subscribers listen to. 3) False — standard SNS doesn't guarantee order (FIFO topics do).

**SES:** 1) SES. 2) Both — SES can send and receive. 3) Any two of: transactional emails (password resets, confirmations), marketing emails (newsletters), notification emails.

## Monitoring & Management

**CloudWatch:** 1) CloudWatch Alarms. 2) Logs = text records (application output, errors). Metrics = numbers over time (CPU %, request count). 3) Yes — alarms can trigger Lambda, SNS, or Auto Scaling actions.

**CloudTrail:** 1) CloudTrail. 2) CloudWatch monitors performance (metrics, logs, alarms). CloudTrail records API activity (who did what). 3) True.

**CloudFormation:** 1) CloudFormation. 2) YAML or JSON. 3) By default, they get deleted too (you can protect specific resources).

## Scenario Questions

1) **S3 + Lambda** — S3 triggers a Lambda function when a file is uploaded, Lambda creates the thumbnail.

2) **CloudFront** (fast frontend) + **Lambda or API Gateway** (scalable API) + **RDS or DynamoDB** (managed database).

3) Use a **maintenance window/downtime** in your monitoring tool, or **mute the relevant CloudWatch alarms** during deployment.

4) **WAF** (blocks SQL injection) + **Shield** (blocks DDoS).

5) **CloudTrail** logs the IAM change → **CloudWatch Events/EventBridge** detects it → **SNS** sends a notification to the team.

</details>

---

*Test yourself, then check the answers. Repeat until it's second nature! 💪*
