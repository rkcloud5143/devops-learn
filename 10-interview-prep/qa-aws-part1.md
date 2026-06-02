# AWS Q&A Part 1: Core Services (200+ Questions)

Basic, Advanced, and Scenario-based questions for AWS.

---

## IAM (Identity & Access Management)

**Q1: What is IAM?**
A: AWS Identity and Access Management. Controls who (authentication) can do what (authorization) in your AWS account.

**Q2: What are the main IAM components?**
A: Users, Groups, Roles, and Policies.

**Q3: What is an IAM user?**
A: An identity representing a person or application. Has permanent credentials (password, access keys).

**Q4: What is an IAM group?**
A: A collection of users. Attach policies to groups to manage permissions for multiple users at once.

**Q5: What is an IAM role?**
A: An identity with permissions that can be assumed temporarily. Used by services, applications, or users. No permanent credentials.

**Q6: What is an IAM policy?**
A: A JSON document defining permissions. Specifies what actions are allowed/denied on which resources.

**Q7: What's the difference between identity-based and resource-based policies?**
A: Identity-based attach to users/groups/roles. Resource-based attach to resources (S3 bucket policy, SQS policy).

**Q8: What is the principle of least privilege?**
A: Grant only the minimum permissions needed to perform a task. Security best practice.

**Q9: What is the root account?**
A: The account created when you first set up AWS. Has unrestricted access. Should be secured with MFA and rarely used.

**Q10: How should you secure the root account?**
A: Enable MFA, don't create access keys, use it only for tasks that require it (billing, account settings).

**Q11: What is MFA?**
A: Multi-Factor Authentication. Requires a second factor (phone, hardware token) in addition to password.

**Q12: What are access keys?**
A: Credentials for programmatic access (CLI, SDK). Consist of Access Key ID and Secret Access Key.

**Q13: How often should you rotate access keys?**
A: Regularly (every 90 days is common). Use IAM credential report to audit.

**Q14: What is STS?**
A: Security Token Service. Provides temporary credentials for assuming roles.

**Q15: What is AssumeRole?**
A: STS API to get temporary credentials for a role. Used for cross-account access, federation, etc.

**Q16: What is a trust policy?**
A: Defines who can assume a role. Attached to the role itself.

**Q17: What is a permissions boundary?**
A: Sets maximum permissions a user/role can have. Even if policy grants more, boundary limits it.

**Q18: What is the IAM policy evaluation logic?**
A: Explicit Deny > Explicit Allow > Implicit Deny. If no policy allows, it's denied.

**Q19: What is a service-linked role?**
A: A role predefined by an AWS service with permissions that service needs. You can't modify its permissions.

**Q20: What is instance profile?**
A: A container for an IAM role that you attach to EC2 instances. Allows EC2 to assume the role.

**Q21: What is ABAC?**
A: Attribute-Based Access Control. Permissions based on tags (e.g., allow access if user's department tag matches resource's department tag).

**Q22: What is the IAM credential report?**
A: CSV report of all users and their credential status (password age, MFA, access keys).

**Q23: What is IAM Access Analyzer?**
A: Identifies resources shared with external entities. Helps find unintended public access.

**Q24: What is AWS Organizations?**
A: Manages multiple AWS accounts. Consolidated billing, SCPs for governance.

**Q25: What is an SCP?**
A: Service Control Policy. Sets permission guardrails for accounts in an Organization. Doesn't grant permissions, only restricts.

---

## EC2 (Elastic Compute Cloud)

**Q26: What is EC2?**
A: Virtual servers in the cloud. You choose instance type, OS, and configuration.

**Q27: What is an AMI?**
A: Amazon Machine Image. Template containing OS, applications, and configuration to launch instances.

**Q28: What are EC2 instance types?**
A: Different combinations of CPU, memory, storage, and networking. Families: General (t3, m5), Compute (c5), Memory (r5), Storage (i3), GPU (p3).

**Q29: What does the instance type name mean (e.g., t3.medium)?**
A: Family (t3) + size (medium). t = burstable, 3 = generation, medium = size.

**Q30: What are burstable instances (T series)?**
A: Accumulate CPU credits when idle, use them when busy. Good for variable workloads. Unlimited mode available.

**Q31: What is an EC2 key pair?**
A: SSH key for Linux instances. AWS stores public key, you keep private key.

**Q32: What is a security group?**
A: Virtual firewall for EC2. Controls inbound and outbound traffic. Stateful (return traffic automatically allowed).

**Q33: What's the difference between security groups and NACLs?**
A: Security groups are stateful, instance-level, allow rules only. NACLs are stateless, subnet-level, allow and deny rules.

**Q34: What is user data?**
A: Script that runs when instance first launches. Used for bootstrapping (install software, configure settings).

**Q35: What is instance metadata?**
A: Information about the instance available at `http://169.254.169.254/latest/meta-data/`. Includes instance ID, IP, IAM role credentials.

**Q36: What is IMDSv2?**
A: Instance Metadata Service version 2. Requires session token, more secure than v1.

**Q37: What are EC2 pricing models?**
A: On-Demand (pay per hour/second), Reserved (1-3 year commitment, up to 72% off), Spot (up to 90% off, can be interrupted), Savings Plans (flexible commitment).

**Q38: What are Spot Instances?**
A: Unused EC2 capacity at steep discount. Can be interrupted with 2-minute warning. Good for fault-tolerant workloads.

**Q39: What is a Spot Fleet?**
A: Collection of Spot and optionally On-Demand instances. Automatically maintains target capacity.

**Q40: What are Reserved Instances?**
A: Commitment to use specific instance type in a region for 1-3 years. Significant discount.

**Q41: What's the difference between Standard and Convertible RIs?**
A: Standard = bigger discount, can't change instance family. Convertible = smaller discount, can change instance family.

**Q42: What are Savings Plans?**
A: Commitment to spend $/hour for 1-3 years. More flexible than RIs — applies across instance families, regions, services.

**Q43: What is a placement group?**
A: Controls how instances are placed on hardware. Cluster (low latency), Spread (high availability), Partition (large distributed workloads).

**Q44: What is an Elastic IP?**
A: Static public IPv4 address. Persists across instance stop/start. Charged when not attached to running instance.

**Q45: What is EC2 hibernation?**
A: Saves RAM contents to EBS, stops instance. On start, RAM is restored. Faster than cold boot.

**Q46: What is a launch template?**
A: Versioned template for launching instances. Includes AMI, instance type, security groups, etc. Used by Auto Scaling.

**Q47: What is EC2 Auto Scaling?**
A: Automatically adjusts number of instances based on demand. Uses launch templates and scaling policies.

**Q48: What are scaling policies?**
A: Rules for when to scale. Target tracking (maintain metric at target), Step (scale based on alarm thresholds), Scheduled (scale at specific times).

**Q49: What is a target tracking scaling policy?**
A: Maintains a metric at a target value. Example: keep average CPU at 50%.

**Q50: What is the difference between horizontal and vertical scaling?**
A: Horizontal = add more instances. Vertical = increase instance size. Horizontal is preferred for availability.

---

## VPC (Virtual Private Cloud)

**Q51: What is a VPC?**
A: Your private network in AWS. You control IP ranges, subnets, routing, and gateways.

**Q52: What is the default VPC?**
A: AWS creates one per region with public subnets, internet gateway, and default security group. Ready to use.

**Q53: What is a CIDR block?**
A: IP address range for your VPC. Example: 10.0.0.0/16 gives you 65,536 IPs.

**Q54: What is a subnet?**
A: A range of IPs within a VPC. Can be public (has route to internet gateway) or private.

**Q55: What makes a subnet public vs private?**
A: Public subnet has a route to an Internet Gateway. Private subnet doesn't.

**Q56: What is an Internet Gateway?**
A: Allows communication between VPC and the internet. Attach to VPC, add route in route table.

**Q57: What is a NAT Gateway?**
A: Allows private subnet instances to access the internet (outbound only). Managed by AWS.

**Q58: What is a NAT Instance?**
A: EC2 instance acting as NAT. Self-managed. Cheaper but less reliable than NAT Gateway.

**Q59: What is a route table?**
A: Rules determining where network traffic goes. Each subnet is associated with one route table.

**Q60: What is the local route?**
A: Automatic route for VPC CIDR. Allows communication within VPC.

**Q61: What is a NACL?**
A: Network Access Control List. Stateless firewall at subnet level. Has allow and deny rules.

**Q62: What is the default NACL?**
A: Allows all inbound and outbound traffic. Custom NACLs deny all by default.

**Q63: What is VPC peering?**
A: Connects two VPCs privately. Traffic stays on AWS network. No transitive peering.

**Q64: What is AWS Transit Gateway?**
A: Hub to connect multiple VPCs and on-premises networks. Simplifies network architecture.

**Q65: What is a VPC endpoint?**
A: Private connection to AWS services without using internet. Gateway endpoints (S3, DynamoDB) or Interface endpoints (most services).

**Q66: What is AWS PrivateLink?**
A: Exposes services privately to other VPCs. Uses interface endpoints.

**Q67: What is a VPN connection?**
A: Encrypted tunnel between your data center and AWS VPC over the internet.

**Q68: What is AWS Direct Connect?**
A: Dedicated private connection from your data center to AWS. More consistent than VPN.

**Q69: What is a bastion host?**
A: EC2 instance in public subnet used to SSH into private instances. Also called jump box.

**Q70: What is VPC Flow Logs?**
A: Captures IP traffic information for VPC, subnet, or ENI. Useful for troubleshooting and security.

**Q71: What is an ENI?**
A: Elastic Network Interface. Virtual network card. Can have multiple IPs, security groups.

**Q72: What is an EFA?**
A: Elastic Fabric Adapter. High-performance network interface for HPC and ML workloads.

**Q73: What are security group best practices?**
A: Least privilege, reference other security groups instead of IPs, use descriptive names, audit regularly.

**Q74: Can a VPC span multiple regions?**
A: No. VPC is regional. Subnets are AZ-specific.

**Q75: What is the maximum CIDR block size for a VPC?**
A: /16 (65,536 IPs). Minimum is /28 (16 IPs).

---

## S3 (Simple Storage Service)

**Q76: What is S3?**
A: Object storage service. Store unlimited data as objects in buckets. Highly durable (99.999999999%).

**Q77: What is an S3 bucket?**
A: Container for objects. Name must be globally unique. Created in a specific region.

**Q78: What is an S3 object?**
A: A file plus metadata. Identified by key (path). Max size 5TB.

**Q79: What are S3 storage classes?**
A: Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Flexible Retrieval, Glacier Deep Archive.

**Q80: When would you use S3 Standard-IA?**
A: Infrequently accessed data that needs immediate access. Lower storage cost, higher retrieval cost.

**Q81: What is S3 Glacier?**
A: Archive storage. Very cheap storage, retrieval takes minutes to hours. Good for backups, compliance.

**Q82: What is S3 Intelligent-Tiering?**
A: Automatically moves objects between tiers based on access patterns. Small monitoring fee.

**Q83: What is S3 versioning?**
A: Keeps multiple versions of an object. Protects against accidental deletion. Can be enabled per bucket.

**Q84: What is an S3 lifecycle policy?**
A: Rules to automatically transition or delete objects. Example: move to Glacier after 90 days.

**Q85: What is S3 replication?**
A: Automatically copies objects to another bucket. Same-Region (SRR) or Cross-Region (CRR).

**Q86: What is S3 Transfer Acceleration?**
A: Uses CloudFront edge locations to speed up uploads. Good for global users.

**Q87: What is S3 Select?**
A: Query data inside objects using SQL. Retrieve only needed data, reducing transfer.

**Q88: What is a presigned URL?**
A: Temporary URL granting access to a private object. Expires after specified time.

**Q89: How do you secure an S3 bucket?**
A: Block public access, bucket policies, ACLs (legacy), encryption, VPC endpoints, access logging.

**Q90: What is S3 Block Public Access?**
A: Account or bucket-level settings to prevent public access. Overrides policies and ACLs.

**Q91: What encryption options does S3 have?**
A: SSE-S3 (AWS managed keys), SSE-KMS (KMS keys), SSE-C (customer-provided keys), client-side encryption.

**Q92: What is S3 Object Lock?**
A: WORM (Write Once Read Many) protection. Prevents deletion for a retention period. For compliance.

**Q93: What is S3 event notification?**
A: Triggers Lambda, SQS, or SNS when objects are created, deleted, etc.

**Q94: What is S3 static website hosting?**
A: Serve static content directly from S3. Configure index and error documents.

**Q95: What is the S3 consistency model?**
A: Strong read-after-write consistency for all operations (since December 2020).

**Q96: What is multipart upload?**
A: Upload large objects in parts. Required for objects > 5GB. Recommended for > 100MB.

**Q97: What is S3 Access Points?**
A: Named network endpoints with distinct permissions. Simplifies managing access for shared datasets.

**Q98: What is S3 Batch Operations?**
A: Perform operations on billions of objects. Copy, invoke Lambda, restore from Glacier.

**Q99: How do you optimize S3 costs?**
A: Right storage class, lifecycle policies, analyze with S3 Storage Lens, delete incomplete multipart uploads.

**Q100: What is S3 Storage Lens?**
A: Analytics dashboard for storage usage and activity across accounts and regions.

---

## EBS & EFS

**Q101: What is EBS?**
A: Elastic Block Store. Block storage volumes for EC2. Like a hard drive.

**Q102: What are EBS volume types?**
A: gp3/gp2 (general SSD), io2/io1 (high-performance SSD), st1 (throughput HDD), sc1 (cold HDD).

**Q103: What's the difference between gp2 and gp3?**
A: gp3 has baseline 3000 IOPS regardless of size, can provision IOPS/throughput independently. gp2 IOPS scales with size.

**Q104: When would you use io2?**
A: Databases, latency-sensitive workloads needing high IOPS (up to 64,000) and durability.

**Q105: Can EBS volumes be attached to multiple instances?**
A: io1/io2 support Multi-Attach (up to 16 instances in same AZ). Others don't.

**Q106: What is an EBS snapshot?**
A: Point-in-time backup of a volume. Stored in S3. Incremental (only changed blocks).

**Q107: Can you resize an EBS volume?**
A: Yes, increase size without detaching. May need to extend filesystem inside OS.

**Q108: What is EBS encryption?**
A: Encrypts data at rest, in transit, and snapshots. Uses KMS. Minimal performance impact.

**Q109: What is EFS?**
A: Elastic File System. Managed NFS file system. Multiple EC2 instances can access simultaneously.

**Q110: What's the difference between EBS and EFS?**
A: EBS = block storage, single instance (usually), AZ-specific. EFS = file storage, shared, regional.

**Q111: What are EFS storage classes?**
A: Standard, Infrequent Access (IA). Use lifecycle policies to move files.

**Q112: What is EFS throughput mode?**
A: Bursting (scales with size) or Provisioned (set specific throughput).

**Q113: What is EFS performance mode?**
A: General Purpose (low latency) or Max I/O (higher throughput, higher latency).

**Q114: What is FSx?**
A: Managed file systems. FSx for Windows (SMB), FSx for Lustre (HPC), FSx for NetApp ONTAP, FSx for OpenZFS.

**Q115: When would you use FSx for Lustre?**
A: High-performance computing, ML training, video processing. Can integrate with S3.

---

## RDS & Databases

**Q116: What is RDS?**
A: Relational Database Service. Managed databases: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, Aurora.

**Q117: What does "managed" mean for RDS?**
A: AWS handles provisioning, patching, backups, recovery, scaling. You manage data and optimization.

**Q118: What is a DB instance?**
A: An isolated database environment. Has compute and storage resources.

**Q119: What is a Multi-AZ deployment?**
A: Synchronous standby replica in another AZ. Automatic failover. For high availability, not read scaling.

**Q120: What are Read Replicas?**
A: Asynchronous copies for read scaling. Can be in different regions. Can be promoted to standalone.

**Q121: What's the difference between Multi-AZ and Read Replicas?**
A: Multi-AZ = HA, automatic failover, same region. Read Replicas = read scaling, manual promotion, can be cross-region.

**Q122: What is Amazon Aurora?**
A: AWS-built MySQL/PostgreSQL-compatible database. 5x faster than MySQL, 3x faster than PostgreSQL. Auto-scaling storage.

**Q123: What is Aurora Serverless?**
A: Auto-scaling Aurora. Scales capacity based on demand. Pay per second. Good for variable workloads.

**Q124: What is Aurora Global Database?**
A: Aurora spanning multiple regions. Sub-second replication. For disaster recovery and global reads.

**Q125: What is RDS Proxy?**
A: Managed connection pooler. Reduces database connections, improves failover time. Great for Lambda.

**Q126: How do you encrypt RDS?**
A: Enable encryption at creation (can't encrypt existing). Uses KMS. Encrypts storage, backups, replicas.

**Q127: What is DynamoDB?**
A: Managed NoSQL database. Key-value and document. Single-digit millisecond latency at any scale.

**Q128: What is a DynamoDB partition key?**
A: Primary key attribute. Determines which partition stores the item. Should have high cardinality.

**Q129: What is a sort key?**
A: Optional second part of primary key. Allows range queries within a partition.

**Q130: What are DynamoDB capacity modes?**
A: Provisioned (set RCU/WCU) or On-Demand (pay per request). On-Demand good for unpredictable workloads.

**Q131: What is a DynamoDB GSI?**
A: Global Secondary Index. Alternate partition/sort key. Query on non-primary attributes.

**Q132: What is a DynamoDB LSI?**
A: Local Secondary Index. Same partition key, different sort key. Must be created at table creation.

**Q133: What is DynamoDB Streams?**
A: Captures item-level changes. Triggers Lambda for real-time processing.

**Q134: What is DynamoDB Global Tables?**
A: Multi-region, multi-active replication. Automatic conflict resolution.

**Q135: What is DAX?**
A: DynamoDB Accelerator. In-memory cache for DynamoDB. Microsecond latency.

**Q136: What is ElastiCache?**
A: Managed in-memory cache. Redis or Memcached. Reduces database load.

**Q137: When would you use Redis vs Memcached?**
A: Redis = persistence, replication, complex data types, pub/sub. Memcached = simple caching, multi-threaded.

**Q138: What is Amazon Redshift?**
A: Data warehouse. Columnar storage, SQL queries. For analytics on large datasets.

**Q139: What is Amazon DocumentDB?**
A: Managed MongoDB-compatible database.

**Q140: What is Amazon Neptune?**
A: Managed graph database. For highly connected data (social networks, recommendations).

---

*Continue to Part 2 for more AWS services...*
