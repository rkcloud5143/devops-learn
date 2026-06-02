# AWS — CLI Commands Reference

## Setup
```bash
aws configure
# Enter: Access Key, Secret Key, Region (ca-central-1), Output (json)
```

## EC2
```bash
aws ec2 describe-instances
aws ec2 start-instances --instance-ids i-1234567890
aws ec2 stop-instances --instance-ids i-1234567890
```

## S3
```bash
aws s3 ls                          # List buckets
aws s3 cp file.txt s3://my-bucket/ # Upload
aws s3 sync ./folder s3://my-bucket/folder  # Sync directory
```

## IAM
```bash
aws iam list-users
aws iam list-roles
```

## CloudWatch
```bash
aws cloudwatch list-metrics --namespace AWS/EC2
```

## Checklist
- [ ] Install AWS CLI, practice 10+ commands
