# AWS EFS Deployment Guide
 
This guide explains how to deploy Amazon Elastic File System (EFS) with the correct VPC and subnet configurations, and how to set up CI/CD.
 
## What is Amazon EFS?
Amazon EFS (Elastic File System) is like a shared network drive in the cloud that:
- Grows and shrinks automatically
- Can be accessed by multiple EC2 instances at the same time
- Works like a regular file system (NFS)
- Stores data across multiple availability zones
 
## Simple Example with EC2
 
### Basic Setup
```
                   ┌─── AZ-1 ────┐      ┌─── AZ-2 ────┐
                   │             │      │             │
                   │  EC2        │      │  EC2        │
                   │  Instance 1 │      │  Instance 2 │
                   │   └──────┐  │      │  ┌──────┘  │
                   │          │  │      │  │         │
                   └──────────┼──┘      └──┼─────────┘
                              │           │
                              │           │
                              ▼           ▼
                         ┌─────────────────────┐
                         │        EFS          │
                         │   (Shared Drive)    │
                         └─────────────────────┘
```
 
## Prerequisites
 
- AWS Account
- GitHub Account
- AWS CLI configured locally
- Terraform installed
 
## VPC and Subnet Configuration
 
1. **Find Your VPC ID**
   ```bash
   aws ec2 describe-vpcs --query 'Vpcs[*].{VpcId:VpcId,Name:Tags[?Key==`Name`].Value|[0],CIDR:CidrBlock}'
   ```
 
2. **Find Subnet IDs in your VPC**
   ```bash
   aws ec2 describe-subnets --filters "Name=vpc-id,Values=YOUR-VPC-ID" --query 'Subnets[*].{SubnetId:SubnetId,AZ:AvailabilityZone,CIDR:CidrBlock}'
   ```
 
3. **Update terraform.tfvars**
   ```hcl
   region     = "us-east-1"              # Your preferred region
   vpc_id     = "vpc-XXXXXXXXXXXXX"      # Your VPC ID
   subnet_ids = [
     "subnet-XXXXXXXXXXXXX",             # Subnet in AZ-1
     "subnet-XXXXXXXXXXXXX"              # Subnet in AZ-2
   ]
   efs_name    = "MyEFS"                 # Your EFS name
   efs_sg_name = "efs-sg"               # Security group name
   ```
 
## Setting up CI/CD with GitHub Actions
 
1. **Create GitHub Repository Secrets**
   - Go to your GitHub repository
   - Navigate to Settings → Secrets and variables → Actions
   - Add these secrets:
     ```
     AWS_ACCESS_KEY_ID        # Your AWS access key
     AWS_SECRET_ACCESS_KEY    # Your AWS secret key
     ```
 
## Deployment Process
 
1. **Clone Repository**
   ```bash
   git clone https://github.com/yourusername/your-repo.git
   cd your-repo
   ```
 
2. **Update Configuration**
   - Update VPC ID and subnet IDs in `terraform.tfvars`
   - Commit and push changes:
   ```bash
   git add .
   git commit -m "Updated VPC and subnet configuration"
   git push origin main
   ```
 
3. **Monitor Deployment**
   - Go to GitHub repository
   - Click "Actions" tab
   - Watch the deployment progress
 
## Best Practices
 
1. **VPC Selection**
   - Choose a VPC with proper internet connectivity
   - Ensure VPC has DNS hostnames enabled
   - Verify CIDR ranges don't conflict
 
2. **Subnet Selection**
   - Choose subnets in different Availability Zones
   - Ensure subnets have proper routing
   - Verify subnet has enough IP addresses
 
3. **Security**
   - Use minimal IAM permissions
   - Rotate AWS access keys regularly
   - Review security group rules
 
## Troubleshooting
 
1. **VPC Issues**
   ```bash
   # Check VPC details
   aws ec2 describe-vpc-attribute --vpc-id vpc-XXXXXXXXXXXXX --attribute enableDnsHostnames
   ```
 
2. **Subnet Issues**
   ```bash
   # Check subnet details
   aws ec2 describe-subnets --subnet-ids subnet-XXXXXXXXXXXXX
   ```
 
3. **Common Issues**
   - VPC not found: Verify VPC ID
   - Subnet not found: Verify subnet IDs
   - Permission denied: Check IAM roles
   - Deployment failure: Check GitHub Actions logs
