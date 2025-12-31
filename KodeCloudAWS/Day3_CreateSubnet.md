# Day 3: Create AWS Subnet

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
    "Account": "xxxxxxxxxxxx",
    "Arn": "arn:aws:sts::xxxxxxxxxxxx:assumed-role/..."
}
```

---

### Step 4: Set the Region

```bash
aws configure set region us-east-1
```

---

### Step 5: Identify the Default VPC

We need to find the VPC ID of the default VPC to create our subnet in it.

```bash
aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text
```

**Save the VPC ID in a variable:**

```bash
VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)
echo $VPC_ID
```

---

### Step 6: Calculate a Valid CIDR Block

Before creating a subnet, we must ensure the CIDR block doesn't overlap with existing subnets.

**1. List existing subnets and their CIDR blocks:**

```bash
aws ec2 describe-subnets --filters "Name=vpc-id,Values=$VPC_ID" --query "Subnets[*].CidrBlock" --output text
```

**2. Analyze the Output:**

* **Existing Blocks:** `172.31.0.0/20`, `172.31.16.0/20`, `172.31.32.0/20`, `172.31.48.0/20`, `172.31.64.0/20`, `172.31.80.0/20`.
* **Pattern:** The default VPC uses `/20` blocks, which increment the third octet by 16 (0, 16, 32, 48, 64, 80).
* **Next Available Block:** To maintain the pattern, add 16 to the last block (80 + 16 = 96).
  * **Chosen CIDR:** `172.31.96.0/20`

**Set your chosen CIDR in a variable:**

```bash
SUBNET_CIDR="172.31.96.0/20"
```

---

### Step 7: Create the Subnet

We will create a subnet named `xfusion-subnet` with the chosen CIDR.

```bash
aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block $SUBNET_CIDR \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-subnet}]'
```

*If you receive an overlap error, repeat the calculation step and pick a different third octet (e.g., 101, 102).*

---

### Step 8: Verify the Subnet

```bash
aws ec2 describe-subnets --filters "Name=tag:Name,Values=xfusion-subnet"
```

**Expected Output:**

```json
{
    "Subnets": [
        {
            "CidrBlock": "172.31.96.0/20",
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "xfusion-subnet"
                }
            ]
        }
    ]
}
```

**Expected Output:**

```json
{
    "Subnets": [
        {
            "AvailabilityZone": "us-east-1a",
            "AvailabilityZoneId": "use1-az1",
            "CidrBlock": "172.31.96.0/20",
            "DefaultForAz": false,
            "MapPublicIpOnLaunch": false,
            "State": "available",
            "SubnetId": "subnet-0123456789abcdef0",
            "VpcId": "vpc-xxxxxxxx",
            "OwnerId": "xxxxxxxxxxxx",
            "AssignIpv6AddressOnCreation": false,
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "xfusion-subnet"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-1:xxxxxxxxxxxx:subnet/subnet-0123456789abcdef0"
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

$ aws configure set region us-east-1

$ VPC_ID=$(aws ec2 describe-vpcs --filters "Name=isDefault,Values=true" --query "Vpcs[0].VpcId" --output text)
$ echo $VPC_ID
vpc-0abcd1234efgh5678

$ aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 172.31.100.0/24 \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=xfusion-subnet}]'
{
    "Subnet": {
        "SubnetId": "subnet-0987654321fedcba0",
        "CidrBlock": "172.31.100.0/24",
        "VpcId": "vpc-0abcd1234efgh5678",
        # ... additional output ...
        "Tags": [
            {
                "Key": "Name",
                "Value": "xfusion-subnet"
            }
        ]
    }
}

$ echo "Subnet created successfully!"
Subnet created successfully!
```

---

## 🖱️ Option 2: GUI Walkthrough (AWS Console)

### Step 1: Log in to AWS Console

**Open URL:**

```
https://console.aws.amazon.com/console/home?region=us-east-1
```

*(Use the provided credentials to log in)*

---

### Step 2: Navigate to VPC

1. Search **VPC** in the AWS search bar
2. Click on **VPC** service
3. Verify region is **us-east-1 (N. Virginia)** in the top-right corner

---

### Step 3: Go to Subnets

1. In the left sidebar, click on **Subnets**
2. Click **Create subnet** button

---

### Step 4: Configure Subnet

1. **VPC ID**: Select the **Default VPC** from the dropdown menu.
2. **Subnet settings**:
   * **Subnet name**: `xfusion-subnet`
   * **Availability Zone**: No preference (or select any e.g., `us-east-1a`)
   * **IPv4 CIDR block**: `172.31.100.0/24` (or any available block like `172.31.101.0/24`)

3. Click **Create subnet**

---

### Step 5: Verify

1. You should see `Successfully created subnet: xfusion-subnet`
2. Search for `xfusion-subnet` in the subnets list to confirm it exists and is available.

---

## ✅ Final Checklist

| Requirement          | Value            |
|----------------------|------------------|
| Subnet Name          | xfusion-subnet   |
| VPC                  | Default VPC      |
| Region               | us-east-1        |
| Created successfully | ✔                |

---
