# Day 1: Create AWS EC2 Key Pair

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

### Step 5: Create the Key Pair (RSA)

```bash
aws ec2 create-key-pair \
  --key-name nautilus-kp \
  --key-type rsa \
  --query 'KeyMaterial' \
  --output text > nautilus-kp.pem
```

---

### Step 6: Secure the Key File

```bash
chmod 400 nautilus-kp.pem
```

---

### Step 7: Verify the Key Pair

```bash
aws ec2 describe-key-pairs --key-names nautilus-kp
```

**Expected Output:**

```json
{
    "KeyPairs": [
        {
            "KeyPairId": "key-0123456789abcdef0",
            "KeyFingerprint": "xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx",
            "KeyName": "nautilus-kp",
            "KeyType": "rsa",
            "Tags": []
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

$ aws ec2 create-key-pair \
    --key-name nautilus-kp \
    --key-type rsa \
    --query 'KeyMaterial' \
    --output text > nautilus-kp.pem

$ chmod 400 nautilus-kp.pem

$ aws ec2 describe-key-pairs --key-names nautilus-kp
{
    "KeyPairs": [
        {
            "KeyPairId": "key-0123456789abcdef0",
            "KeyFingerprint": "xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx",
            "KeyName": "nautilus-kp",
            "KeyType": "rsa",
            "Tags": []
        }
    ]
}

$ echo "Key pair created successfully!"
Key pair created successfully!
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
|----------|--------------------|
| Username | kk_labs_user_219308 |
| Password | A!nDqmz15aEw       |

---

### Step 2: Navigate to EC2

1. Search **EC2** in the AWS search bar
2. Click on **EC2** service
3. Verify region is **us-east-1 (N. Virginia)** in the top-right corner

---

### Step 3: Go to Key Pairs

1. In the left sidebar, scroll down to **Network & Security**
2. Click on **Key Pairs**

---

### Step 4: Create Key Pair

1. Click **Create key pair** button

2. Fill in the details:

| Field                     | Value        |
|---------------------------|--------------|
| Name                      | nautilus-kp  |
| Key pair type             | RSA          |
| Private key file format   | .pem         |

1. Click **Create key pair**

2. The `.pem` file will **auto-download** to your computer

---

## ✅ Final Checklist

| Requirement          | Value       |
|----------------------|-------------|
| Key pair name        | nautilus-kp |
| Key type             | RSA         |
| Region               | us-east-1   |
| Created successfully | ✔           |

---
