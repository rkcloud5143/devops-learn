# AWS Q&A Part 2: Compute, Networking & Security (200+ Questions)

Continuing AWS coverage with Lambda, ELB, Route 53, CloudFront, and Security services.

---

## Lambda & Serverless

**Q141: What is AWS Lambda?**
A: Serverless compute. Run code without managing servers. Pay only for execution time.

**Q142: What triggers Lambda?**
A: API Gateway, S3, DynamoDB Streams, SQS, SNS, CloudWatch Events, Kinesis, and many more.

**Q143: What is the maximum Lambda execution time?**
A: 15 minutes.

**Q144: What is Lambda memory configuration?**
A: 128 MB to 10,240 MB. CPU scales proportionally with memory.

**Q145: What is a cold start?**
A: Delay when Lambda creates a new execution environment. Happens on first invocation or after idle period.

**Q146: How do you reduce cold starts?**
A: Provisioned Concurrency (keeps instances warm), smaller deployment packages, choose faster runtimes.

**Q147: What is Provisioned Concurrency?**
A: Pre-initialized Lambda instances. Eliminates cold starts. You pay for provisioned capacity.

**Q148: What is Lambda Layers?**
A: Shared code/libraries across functions. Reduces deployment package size.

**Q149: What is Lambda@Edge?**
A: Run Lambda at CloudFront edge locations. For request/response manipulation.

**Q150: How do you give Lambda access to VPC resources?**
A: Configure VPC, subnets, and security groups. Lambda creates ENIs. Needs NAT Gateway for internet.

**Q151: What is the Lambda execution role?**
A: IAM role that Lambda assumes. Grants permissions to access AWS services.

**Q152: What is Lambda concurrency?**
A: Number of simultaneous executions. Default account limit: 1000. Can reserve per function.

**Q153: What is reserved concurrency?**
A: Guarantees capacity for a function. Also acts as a limit.

**Q154: What are Lambda destinations?**
A: Route async invocation results to SQS, SNS, Lambda, or EventBridge based on success/failure.

**Q155: What is Lambda container image support?**
A: Package Lambda as Docker image up to 10GB. Use AWS base images or custom.

**Q156: What is Step Functions?**
A: Orchestrate Lambda functions and AWS services into workflows. Visual workflow designer.

**Q157: What is API Gateway?**
A: Managed API service. Create REST, HTTP, and WebSocket APIs. Handles auth, throttling, caching.

**Q158: What's the difference between REST API and HTTP API in API Gateway?**
A: HTTP API is cheaper, faster, simpler. REST API has more features (caching, request validation, WAF).

**Q159: What is API Gateway throttling?**
A: Limits requests per second. Default: 10,000 RPS. Prevents abuse.

**Q160: What is API Gateway caching?**
A: Caches responses to reduce backend calls. Configurable TTL.

**Q161: What is a Lambda authorizer?**
A: Custom authorization logic in Lambda. Returns IAM policy. For token validation.

**Q162: What is Amazon EventBridge?**
A: Serverless event bus. Routes events between AWS services, SaaS apps, and custom apps.

**Q163: What is SQS?**
A: Simple Queue Service. Managed message queue. Decouples producers and consumers.

**Q164: What's the difference between Standard and FIFO queues?**
A: Standard = unlimited throughput, at-least-once, best-effort ordering. FIFO = exactly-once, strict ordering, 300 TPS (3000 with batching).

**Q165: What is SQS visibility timeout?**
A: Time a message is hidden after being read. If not deleted, becomes visible again. Default: 30 seconds.

**Q166: What is SQS dead-letter queue?**
A: Queue for messages that fail processing repeatedly. For debugging and reprocessing.

**Q167: What is SQS long polling?**
A: Waits for messages instead of returning immediately. Reduces empty responses and cost.

**Q168: What is SNS?**
A: Simple Notification Service. Pub/sub messaging. One message to many subscribers.

**Q169: What can subscribe to an SNS topic?**
A: SQS, Lambda, HTTP/HTTPS endpoints, email, SMS, mobile push.

**Q170: What is SNS FIFO?**
A: Ordered, exactly-once delivery. Works with SQS FIFO queues.

**Q171: What is the fanout pattern?**
A: SNS topic with multiple SQS queues subscribed. One publish, multiple parallel consumers.

**Q172: What is Amazon MQ?**
A: Managed message broker. Apache ActiveMQ and RabbitMQ. For migrating existing apps.

**Q173: What is Kinesis?**
A: Real-time streaming data. Kinesis Data Streams (collect), Kinesis Firehose (load), Kinesis Analytics (analyze).

**Q174: What's the difference between Kinesis and SQS?**
A: Kinesis = real-time streaming, multiple consumers, replay. SQS = message queue, single consumer per message, no replay.

**Q175: What is a Kinesis shard?**
A: Unit of capacity. 1 MB/s in, 2 MB/s out. Scale by adding shards.

---

## ELB & Auto Scaling

**Q176: What is Elastic Load Balancing?**
A: Distributes traffic across targets. Three types: ALB, NLB, CLB (legacy).

**Q177: What is an Application Load Balancer (ALB)?**
A: Layer 7 (HTTP/HTTPS). Path-based routing, host-based routing, WebSocket support.

**Q178: What is a Network Load Balancer (NLB)?**
A: Layer 4 (TCP/UDP). Ultra-low latency, millions of requests per second. Static IP support.

**Q179: When would you use ALB vs NLB?**
A: ALB for HTTP/HTTPS apps with routing needs. NLB for extreme performance, TCP/UDP, static IPs.

**Q180: What is a target group?**
A: Group of targets (EC2, IP, Lambda) that receive traffic from load balancer.

**Q181: What are health checks?**
A: Load balancer checks target health. Unhealthy targets don't receive traffic.

**Q182: What is connection draining?**
A: Allows in-flight requests to complete before deregistering target. Default: 300 seconds.

**Q183: What is sticky sessions?**
A: Routes user to same target. Uses cookies. Can cause uneven load.

**Q184: What is cross-zone load balancing?**
A: Distributes traffic evenly across all targets in all AZs. Enabled by default for ALB.

**Q185: What is SSL termination?**
A: Load balancer decrypts HTTPS, sends HTTP to targets. Offloads SSL processing.

**Q186: What is end-to-end encryption?**
A: HTTPS from client to load balancer to target. More secure but more overhead.

**Q187: What is Auto Scaling?**
A: Automatically adjusts capacity based on demand. Maintains availability and optimizes cost.

**Q188: What is a launch template?**
A: Defines instance configuration for Auto Scaling. AMI, instance type, security groups, etc.

**Q189: What is desired capacity?**
A: Target number of instances. Auto Scaling maintains this unless scaling policy changes it.

**Q190: What is a scaling policy?**
A: Rules for when to scale. Target tracking, step scaling, or scheduled.

**Q191: What is scale-in protection?**
A: Prevents specific instances from being terminated during scale-in.

**Q192: What is a cooldown period?**
A: Time after scaling activity before another can start. Prevents thrashing.

**Q193: What is predictive scaling?**
A: Uses ML to predict demand and scale proactively.

**Q194: What is instance refresh?**
A: Rolling replacement of instances. For deploying new AMI or launch template.

---

## Route 53

**Q195: What is Route 53?**
A: Managed DNS service. Domain registration, DNS routing, health checks.

**Q196: What is a hosted zone?**
A: Container for DNS records for a domain. Public (internet) or private (VPC).

**Q197: What DNS record types does Route 53 support?**
A: A, AAAA, CNAME, MX, TXT, NS, SOA, PTR, SRV, CAA, ALIAS.

**Q198: What is an ALIAS record?**
A: Route 53-specific. Points to AWS resources (ELB, CloudFront, S3). Works at zone apex.

**Q199: What's the difference between ALIAS and CNAME?**
A: ALIAS works at zone apex (example.com), no charge for queries. CNAME can't be at apex, charged.

**Q200: What routing policies does Route 53 have?**
A: Simple, Weighted, Latency, Failover, Geolocation, Geoproximity, Multivalue Answer.

**Q201: What is weighted routing?**
A: Distribute traffic by percentage. Example: 70% to v1, 30% to v2.

**Q202: What is latency-based routing?**
A: Routes to region with lowest latency for the user.

**Q203: What is failover routing?**
A: Active-passive. Routes to secondary if primary health check fails.

**Q204: What is geolocation routing?**
A: Routes based on user's location. For localized content or restrictions.

**Q205: What are Route 53 health checks?**
A: Monitor endpoint health. Can trigger failover or remove from DNS.

**Q206: What is Route 53 Resolver?**
A: DNS resolution for hybrid environments. Inbound/outbound endpoints for VPC and on-premises.

---

## CloudFront

**Q207: What is CloudFront?**
A: Content Delivery Network (CDN). Caches content at edge locations globally.

**Q208: What is an edge location?**
A: Data center where CloudFront caches content. 400+ locations worldwide.

**Q209: What is an origin?**
A: Source of content. S3 bucket, ALB, EC2, or custom HTTP server.

**Q210: What is a distribution?**
A: CloudFront configuration. Defines origins, behaviors, and settings.

**Q211: What is TTL in CloudFront?**
A: Time To Live. How long content is cached before checking origin.

**Q212: How do you invalidate CloudFront cache?**
A: Create invalidation for specific paths. Or use versioned file names.

**Q213: What is Origin Access Control (OAC)?**
A: Restricts S3 access to CloudFront only. Replaces Origin Access Identity (OAI).

**Q214: What is CloudFront signed URLs/cookies?**
A: Restrict access to content. For paid content or time-limited access.

**Q215: What is CloudFront Functions?**
A: Lightweight JavaScript functions at edge. For simple request/response manipulation.

**Q216: What's the difference between CloudFront Functions and Lambda@Edge?**
A: CloudFront Functions = simpler, cheaper, sub-millisecond. Lambda@Edge = more powerful, longer execution.

**Q217: What is field-level encryption?**
A: Encrypts specific form fields at edge. Only your application can decrypt.

---

## Security Services

**Q218: What is KMS?**
A: Key Management Service. Create and manage encryption keys.

**Q219: What is a CMK?**
A: Customer Master Key. The primary resource in KMS. Can be AWS-managed or customer-managed.

**Q220: What's the difference between AWS-managed and customer-managed keys?**
A: AWS-managed = AWS creates and manages, auto-rotates. Customer-managed = you control, can set policies, manual rotation.

**Q221: What is envelope encryption?**
A: Encrypt data with data key, encrypt data key with master key. Efficient for large data.

**Q222: What is key rotation?**
A: Automatically create new key material yearly. Old material kept for decryption.

**Q223: What is AWS Secrets Manager?**
A: Store and rotate secrets (passwords, API keys). Automatic rotation for RDS, Redshift, DocumentDB.

**Q224: What's the difference between Secrets Manager and Parameter Store?**
A: Secrets Manager = automatic rotation, higher cost. Parameter Store = simpler, cheaper, manual rotation.

**Q225: What is AWS Certificate Manager (ACM)?**
A: Provision and manage SSL/TLS certificates. Free for AWS services. Auto-renewal.

**Q226: What is AWS WAF?**
A: Web Application Firewall. Protects against SQL injection, XSS, and other attacks.

**Q227: What are WAF rules?**
A: Conditions that match requests. Allow, block, or count. Managed rules available.

**Q228: What is AWS Shield?**
A: DDoS protection. Standard (free, automatic) and Advanced (paid, 24/7 support).

**Q229: What is AWS Firewall Manager?**
A: Centrally manage WAF, Shield, security groups across accounts.

**Q230: What is Amazon GuardDuty?**
A: Threat detection. Analyzes CloudTrail, VPC Flow Logs, DNS logs. ML-based.

**Q231: What is Amazon Inspector?**
A: Vulnerability assessment. Scans EC2 and container images for vulnerabilities.

**Q232: What is Amazon Macie?**
A: Discovers and protects sensitive data in S3. Uses ML to identify PII.

**Q233: What is AWS Security Hub?**
A: Centralized security view. Aggregates findings from GuardDuty, Inspector, Macie, etc.

**Q234: What is Amazon Detective?**
A: Investigate security findings. Visualizes relationships and timelines.

**Q235: What is AWS Config?**
A: Tracks resource configuration changes. Evaluates compliance against rules.

**Q236: What is a Config rule?**
A: Evaluates resource configuration. AWS-managed or custom Lambda.

**Q237: What is AWS CloudTrail?**
A: Logs all API calls. Who did what, when, from where. Essential for auditing.

**Q238: What is a CloudTrail trail?**
A: Configuration for logging. Can be single-region or all-regions.

**Q239: What is CloudTrail Insights?**
A: Detects unusual API activity. Identifies anomalies.

**Q240: What is AWS Trusted Advisor?**
A: Best practice recommendations. Cost optimization, security, performance, fault tolerance.

---

## Monitoring & Management

**Q241: What is CloudWatch?**
A: Monitoring service. Metrics, logs, alarms, dashboards.

**Q242: What is a CloudWatch metric?**
A: Time-ordered data points. CPU utilization, request count, etc.

**Q243: What is a CloudWatch alarm?**
A: Watches a metric and triggers actions. OK, ALARM, INSUFFICIENT_DATA states.

**Q244: What actions can CloudWatch alarms trigger?**
A: SNS notification, Auto Scaling action, EC2 action, Systems Manager action.

**Q245: What is CloudWatch Logs?**
A: Collect, store, and analyze log data. From EC2, Lambda, containers, etc.

**Q246: What is a log group?**
A: Collection of log streams with same retention and permissions.

**Q247: What is CloudWatch Logs Insights?**
A: Query and analyze logs with SQL-like syntax.

**Q248: What is CloudWatch Container Insights?**
A: Monitoring for ECS, EKS, Kubernetes. Collects metrics and logs.

**Q249: What is CloudWatch Synthetics?**
A: Canary scripts that monitor endpoints. Simulates user behavior.

**Q250: What is AWS X-Ray?**
A: Distributed tracing. Visualizes request flow through microservices.

**Q251: What is a trace in X-Ray?**
A: End-to-end request path. Contains segments from each service.

**Q252: What is AWS Systems Manager?**
A: Operations hub. Patch management, run commands, parameter store, session manager.

**Q253: What is SSM Session Manager?**
A: Secure shell access without SSH keys or bastion hosts. Audited.

**Q254: What is SSM Parameter Store?**
A: Hierarchical storage for configuration and secrets. Free tier available.

**Q255: What is SSM Run Command?**
A: Execute commands on EC2 instances remotely. No SSH needed.

**Q256: What is SSM Patch Manager?**
A: Automate OS and application patching.

**Q257: What is AWS Organizations?**
A: Manage multiple AWS accounts. Consolidated billing, SCPs.

**Q258: What is AWS Control Tower?**
A: Set up and govern multi-account environment. Landing zone with guardrails.

**Q259: What is a landing zone?**
A: Well-architected multi-account baseline. Includes logging, security accounts.

**Q260: What is AWS Service Catalog?**
A: Create and manage approved products. Self-service for users.

---

*Continue to Part 3 for Kubernetes, Docker, Terraform, and CI/CD...*
