# Day 2: Temporary User Setup with Expiry

## 🎯 Complete Coding Walkthrough (SSH to User Creation with Expiry)

### Step 1: SSH into App Server 1

```bash
# Basic SSH connection
ssh username@app_server_1

# Example:
ssh tony@stapp01

# SSH with specific port
ssh -p 22 username@app_server_1

# SSH with private key
ssh -i /path/to/private_key.pem username@app_server_1
```

**Expected Output:**

```
The authenticity of host 'stapp01 (172.16.238.10)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'stapp01' (ECDSA) to the list of known hosts.
username@stapp01's password: 
[username@stapp01 ~]$ 
```

---

### Step 2: Verify Current User and Permissions

```bash
# Check current user
whoami

# Check if you have sudo privileges
sudo -l
```

**Expected Output:**

```
tony
# OR
User tony may run the following commands on stapp01:
    (ALL) ALL
```

---

### Step 3: Create User with Expiry Date

```bash
# Create user 'mark' with expiry date 2024-03-28
sudo useradd -e 2024-03-28 mark
```

**Flags Used:**

| Flag | Purpose |
|------|---------|
| `-e` | Set account expiry date (YYYY-MM-DD format) |

**Note:** User name must be lowercase as per standard protocol.

---

### Step 4: Verify User Creation

```bash
# Check if user exists in /etc/passwd
grep mark /etc/passwd

# View user ID information
id mark

# Check if home directory was created
ls -la /home/ | grep mark
```

**Expected Output:**

```
mark:x:1002:1002::/home/mark:/bin/bash

uid=1002(mark) gid=1002(mark) groups=1002(mark)

drwx------  2 mark mark 4096 Dec 28 23:50 mark
```

---

### Step 5: Verify Account Expiry Date

```bash
# Check account expiry details
sudo chage -l mark
```

**Expected Output:**

```
Last password change                                    : Dec 28, 2024
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Mar 28, 2024
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

**Key Line to Verify:**

```
Account expires : Mar 28, 2024
```

---

### Step 6: (Optional) Set Password for the User

```bash
# Set password for the user
sudo passwd mark
```

**Expected Output:**

```
Changing password for user mark.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

---

### Step 7: (Optional) Create Home Directory if Not Created

```bash
# If home directory wasn't created automatically, use -m flag
sudo useradd -m -e 2024-03-28 mark

# Or create manually
sudo mkdir /home/mark
sudo chown mark:mark /home/mark
sudo chmod 700 /home/mark
```

---

### Step 8: Test User Login Before Expiry

```bash
# Switch to the user (before expiry date)
su - mark
```

**Expected Output (if password is set):**

```
Password: 
[mark@stapp01 ~]$ 
```

**Exit the user session:**

```bash
exit
```

---

### Step 9: Verify Expiry in /etc/shadow

```bash
# Check shadow file for expiry date
sudo grep mark /etc/shadow
```

**Expected Output:**

```
mark:!!:19716:0:99999:7::19811:
```

**Note:** The number `19811` represents days since epoch (Jan 1, 1970) = March 28, 2024

---

### 📋 Complete Session Example

```bash
# Full walkthrough in one session
$ ssh tony@stapp01
tony@stapp01's password: ********

[tony@stapp01 ~]$ whoami
tony

[tony@stapp01 ~]$ sudo useradd -e 2024-03-28 mark

[tony@stapp01 ~]$ grep mark /etc/passwd
mark:x:1002:1002::/home/mark:/bin/bash

[tony@stapp01 ~]$ id mark
uid=1002(mark) gid=1002(mark) groups=1002(mark)

[tony@stapp01 ~]$ sudo chage -l mark
Last password change                                    : Dec 28, 2024
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Mar 28, 2024
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7

[tony@stapp01 ~]$ sudo passwd mark
Changing password for user mark.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.

[tony@stapp01 ~]$ su - mark
Password: 
[mark@stapp01 ~]$ whoami
mark

[mark@stapp01 ~]$ exit
logout

[tony@stapp01 ~]$ echo "User 'mark' created successfully with expiry date 2024-03-28!"
User 'mark' created successfully with expiry date 2024-03-28!

[tony@stapp01 ~]$ exit
logout
Connection to stapp01 closed.
```

---

## 🔍 What Happens Internally

### User Creation Process

1. **Creates user `mark`**
   - Adds entry to `/etc/passwd`
   - Creates home directory `/home/mark`
   - Assigns default shell `/bin/bash`

2. **Sets account expiration in `/etc/shadow`**
   - Stores expiry date as days since epoch
   - After 2024-03-28, user cannot log in

3. **Important Notes:**
   - ❌ This does NOT delete the user automatically
   - ✅ It only disables login after the expiry date
   - ✅ Files owned by `mark` remain on the system
   - ✅ User can still own processes and files

---

## 📝 Additional Commands Reference

### Modify Expiry Date

```bash
# Extend expiry date
sudo usermod -e 2024-06-30 mark

# Remove expiry date (make permanent)
sudo usermod -e "" mark

# Or using chage
sudo chage -E 2024-06-30 mark
sudo chage -E -1 mark  # Remove expiry
```

### Check Expiry Status

```bash
# View expiry date
sudo chage -l mark | grep "Account expires"

# Check all users with expiry dates
sudo awk -F: '$8 != "" {print $1, $8}' /etc/shadow
```

### Delete User After Expiry

```bash
# Remove user and home directory
sudo userdel -r mark

# Remove user but keep home directory
sudo userdel mark
```

---

## 🎯 One-Line Summary

```bash
sudo useradd -e 2024-03-28 mark
```

---
