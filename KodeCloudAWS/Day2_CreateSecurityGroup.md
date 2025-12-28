# Day 2: Create AWS Security Group with Inbound Rules

---

## 🖥️ Option 1: CLI Walkthrough (Recommended)

### Step 1: Open the AWS Client Terminal

```bash
# Click the expand/toggle button to open the terminal of the aws-client host
```

---

### Step 2: Get AWS Credentials

```bash
showcreds
```

**Expected Output:**

```
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
```

---

### Step 3: Export the Credentials

```bash
# Copy and paste all three export commands into the terminal
export AWS_ACCESS_KEY_ID=XXXXX
export AWS_SECRET_ACCESS_KEY=XXXXX
export AWS_SESSION_TOKEN=XXXXX
```

**Verify credentials are active:**

```bash
aws sts get-caller-identity
```

**Expected Output:**

```json
{
    "UserId": "AROAXXXXXXXXXX:user",
    "Account": "726556490828",
    "Arn": "arn:aws:sts::726556490828:assumed-role/..."
}
```

---

### Step 4: Set the Region

```bash
aws configure set region us-east-1
```

---

### Step 5: Get the Default VPC ID

```bash
# Get the default VPC ID
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text
```

**Expected Output:**

```
vpc-0a1b2c3d4e5f6g7h8
```

**Save it to a variable:**

```bash
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)
echo $VPC_ID
```

---

### Step 6: Create the Security Group

```bash
aws ec2 create-security-group \
  --group-name nautilus-sg \
  --description "Security group for Nautilus project with HTTP and SSH access" \
  --vpc-id $VPC_ID
```

**Expected Output:**

```json
{
    "GroupId": "sg-0123456789abcdef0"
}
```

**Save the Security Group ID:**

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name nautilus-sg \
  --description "Security group for Nautilus project with HTTP and SSH access" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)
echo $SG_ID
```

---

### Step 7: Add SSH Inbound Rule (Port 22)

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0
```

**Expected Output:**

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0123456789abcdef0",
            "GroupId": "sg-0123456789abcdef0",
            "GroupOwnerId": "726556490828",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

---

### Step 8: Add HTTP Inbound Rule (Port 80)

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

**Expected Output:**

```json
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-0987654321fedcba0",
            "GroupId": "sg-0123456789abcdef0",
            "GroupOwnerId": "726556490828",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0"
        }
    ]
}
```

---

### Step 9: Verify the Security Group

```bash
aws ec2 describe-security-groups --group-ids $SG_ID
```

**Expected Output:**

```json
{
    "SecurityGroups": [
        {
            "Description": "Security group for Nautilus project with HTTP and SSH access",
            "GroupName": "nautilus-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "ToPort": 22
                },
                {
                    "FromPort": 80,
                    "IpProtocol": "tcp",
                    "IpRanges": [
                        {
                            "CidrIp": "0.0.0.0/0"
                        }
                    ],
                    "ToPort": 80
                }
            ],
            "OwnerId": "726556490828",
            "GroupId": "sg-0123456789abcdef0",
            "VpcId": "vpc-0a1b2c3d4e5f6g7h8"
        }
    ]
}
```

---

### 📋 Complete CLI Session Example

```bash
# Full walkthrough in one session
$ showcreds
export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
export AWS_SESSION_TOKEN=FwoGZXIvYXdzEBYaD...

$ export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
$ export AWS_SECRET_ACCESS_KEY=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
$ export AWS_SESSION_TOKEN=FwoGZXIvYXdzEBYaD...

$ aws sts get-caller-identity
{
    "UserId": "AROAXXXXXXXXXX:user",
    "Account": "726556490828",
    "Arn": "arn:aws:sts::726556490828:assumed-role/..."
}

$ aws configure set region us-east-1

$ VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)
$ echo $VPC_ID
vpc-0a1b2c3d4e5f6g7h8

$ SG_ID=$(aws ec2 create-security-group \
    --group-name nautilus-sg \
    --description "Security group for Nautilus project with HTTP and SSH access" \
    --vpc-id $VPC_ID \
    --query 'GroupId' \
    --output text)
$ echo $SG_ID
sg-0123456789abcdef0

$ aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [...]
}

$ aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [...]
}

$ aws ec2 describe-security-groups --group-ids $SG_ID
{
    "SecurityGroups": [
        {
            "Description": "Security group for Nautilus project with HTTP and SSH access",
            "GroupName": "nautilus-sg",
            "IpPermissions": [
                {
                    "FromPort": 22,
                    "IpProtocol": "tcp",
                    "IpRanges": [{"CidrIp": "0.0.0.0/0"}],
                    "ToPort": 22
                },
                {
                    "FromPort": 80,
                    "IpProtocol": "tcp",
                    "IpRanges": [{"CidrIp": "0.0.0.0/0"}],
                    "ToPort": 80
                }
            ],
            "GroupId": "sg-0123456789abcdef0",
            "VpcId": "vpc-0a1b2c3d4e5f6g7h8"
        }
    ]
}

$ echo "Security group created successfully with SSH and HTTP access!"
Security group created successfully with SSH and HTTP access!
```

---

## 🖱️ Option 2: GUI Walkthrough (AWS Console)

### Step 1: Log in to AWS Console

**Open URL:**

```
https://726556490828.signin.aws.amazon.com/console?region=us-east-1
```

**Login Credentials:**

| Field    | Value              |
|----------|-------------------|
| Username | kk_labs_user_219308 |
| Password | A!nDqmz15aEw       |

---

### Step 2: Navigate to EC2

1. Search **EC2** in the AWS search bar
2. Click on **EC2** service
3. Verify region is **us-east-1 (N. Virginia)** in the top-right corner

---

### Step 3: Go to Security Groups

1. In the left sidebar, scroll down to **Network & Security**
2. Click on **Security Groups**

---

### Step 4: Create Security Group

1. Click **Create security group** button

2. Fill in the basic details:

| Field                     | Value                                                      |
|---------------------------|------------------------------------------------------------|
| Security group name       | nautilus-sg                                                |
| Description               | Security group for Nautilus project with HTTP and SSH access |
| VPC                       | (Select default VPC)                                       |

---

### Step 5: Add Inbound Rules

**Click "Add rule" twice to add both SSH and HTTP rules:**

#### Rule 1: SSH

| Field       | Value       |
|-------------|-------------|
| Type        | SSH         |
| Protocol    | TCP         |
| Port range  | 22          |
| Source      | 0.0.0.0/0   |
| Description | SSH access  |

#### Rule 2: HTTP

| Field       | Value       |
|-------------|-------------|
| Type        | HTTP        |
| Protocol    | TCP         |
| Port range  | 80          |
| Source      | 0.0.0.0/0   |
| Description | HTTP access |

---

### Step 6: Review Outbound Rules

**Default outbound rule (leave as is):**

| Type        | Protocol | Port range | Destination |
|-------------|----------|------------|-------------|
| All traffic | All      | All        | 0.0.0.0/0   |

---

### Step 7: Create the Security Group

1. Scroll down and click **Create security group**
2. You should see a success message with the Security Group ID

---

## 🔍 Understanding the Configuration

### CIDR Block: 0.0.0.0/0

| CIDR          | Meaning                          | Use Case                    |
|---------------|----------------------------------|-----------------------------|
| `0.0.0.0/0`   | All IPv4 addresses (anywhere)    | Public web servers, testing |
| `10.0.0.0/16` | Specific VPC range               | Internal services           |
| `203.0.113.5/32` | Single specific IP            | Restricted admin access     |

**⚠️ Security Note:** Using `0.0.0.0/0` allows access from anywhere on the internet. For production environments, restrict to specific IP ranges.

---

### Port Reference

| Service | Port | Protocol | Common Use                |
|---------|------|----------|---------------------------|
| SSH     | 22   | TCP      | Remote server access      |
| HTTP    | 80   | TCP      | Web traffic (unencrypted) |
| HTTPS   | 443  | TCP      | Web traffic (encrypted)   |
| RDP     | 3389 | TCP      | Windows remote desktop    |
| MySQL   | 3306 | TCP      | Database access           |

---

## 📝 Additional Commands Reference

### Add HTTPS Rule (Port 443)

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

---

### Add Custom Port Range

```bash
aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 8000-9000 \
  --cidr 0.0.0.0/0
```

---

### Remove a Rule

```bash
aws ec2 revoke-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

---

### Delete Security Group

```bash
aws ec2 delete-security-group --group-id $SG_ID
```

---

### List All Security Groups

```bash
aws ec2 describe-security-groups --query "SecurityGroups[*].[GroupId,GroupName,Description]" --output table
```

---

## ✅ Final Checklist

| Requirement                  | Value                                                      |
|------------------------------|------------------------------------------------------------|
| Security group name          | nautilus-sg                                                |
| VPC                          | Default VPC                                                |
| SSH inbound rule (Port 22)   | ✔ Source: 0.0.0.0/0                                        |
| HTTP inbound rule (Port 80)  | ✔ Source: 0.0.0.0/0                                        |
| Region                       | us-east-1                                                  |
| Created successfully         | ✔                                                          |

---
