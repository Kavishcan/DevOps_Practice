# Day 4: Enable Versioning for S3 Bucket

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

### Step 5: Enable Bucket Versioning

We need to enable versioning for the bucket named `datacenter-s3-9644`.

```bash
aws s3api put-bucket-versioning \
    --bucket datacenter-s3-9644 \
    --versioning-configuration Status=Enabled
```

_(Note: This command produces no output if successful)_

---

### Step 6: Verify Versioning is Enabled

To confirm that versioning has been enabled, run the following command:

```bash
aws s3api get-bucket-versioning --bucket datacenter-s3-9644
```

**Expected Output:**

```json
{
    "Status": "Enabled"
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

$ aws s3api put-bucket-versioning --bucket datacenter-s3-9644 --versioning-configuration Status=Enabled

$ aws s3api get-bucket-versioning --bucket datacenter-s3-9644
{
    "Status": "Enabled"
}

$ echo "Versioning enabled successfully!"
Versioning enabled successfully!
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

### Step 2: Navigate to S3

1. Search **S3** in the AWS search bar.
2. Click on **S3** service.
3. Verify region is **us-east-1 (N. Virginia)** if applicable (S3 is global, but buckets are regional).

---

### Step 3: Find the Bucket

1. In the **Buckets** list, locate the bucket named `datacenter-s3-9644`.
2. Click on the bucket name to open its details.

---

### Step 4: Enable Versioning

1. Click on the **Properties** tab.
2. Scroll down to the **Bucket Versioning** section.
3. Click **Edit**.
4. Select **Enable**.
5. Click **Save changes**.

---

### Step 5: Verify

1. You should see a success banner: `Successfully updated bucket versioning`.
2. In the **Properties** tab, under **Bucket Versioning**, it should now say **Bucket Versioning: Enabled**.

---