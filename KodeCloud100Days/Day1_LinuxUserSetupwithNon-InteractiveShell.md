# Day 1: Linux User Setup with Non-Interactive Shell

## ️ Complete Coding Walkthrough (SSH to User Creation)

### Step 1: SSH into the Remote Server

```bash
# Basic SSH connection
ssh username@server_ip

# Example:
ssh root@192.168.1.100

# SSH with specific port
ssh -p 22 username@server_ip

# SSH with private key
ssh -i /path/to/private_key.pem username@server_ip
```

**Expected Output:**

```
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
ECDSA key fingerprint is SHA256:xxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.100' (ECDSA) to the list of known hosts.
username@192.168.1.100's password: 
[username@server ~]$ 
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
root
# OR
username
User username may run the following commands on server:
    (ALL) ALL
```

---

### Step 3: Check Available Shells

```bash
# List all valid shells on the system
cat /etc/shells
```

**Expected Output:**

```
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
/sbin/nologin
/usr/sbin/nologin
```

---

### Step 4: Create the User with Non-Interactive Shell

```bash
# Create user with non-interactive shell
sudo useradd -m -c "Application Service Account" -s /sbin/nologin appuser
```

**Flags Used:**

| Flag | Purpose |
|------|---------|
| `-m` | Create home directory `/home/appuser` |
| `-c` | Add comment/description |
| `-s` | Set login shell to `/sbin/nologin` |

---

### Step 5: Verify User Creation

```bash
# Check if user exists in /etc/passwd
grep appuser /etc/passwd

# Check if home directory was created
ls -la /home/ | grep appuser

# View user ID information
id appuser
```

**Expected Output:**

```
appuser:x:1001:1001:Application Service Account:/home/appuser:/sbin/nologin

drwx------  2 appuser appuser 4096 Dec 25 15:10 appuser

uid=1001(appuser) gid=1001(appuser) groups=1001(appuser)
```

---

### Step 6: (Optional) Set Password for the User

```bash
# Set password for the user
sudo passwd appuser
```

**Expected Output:**

```
Changing password for user appuser.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
```

---

### Step 7: Test Non-Interactive Shell Restriction

```bash
# Attempt to switch to the user
su - appuser
```

**Expected Output:**

```
This account is currently not available.
```

---

### Step 8: Verify User Cannot SSH

```bash
# Exit and try to SSH as the new user (from another terminal)
ssh appuser@localhost
```

**Expected Output:**

```
appuser@localhost's password: 
This account is currently not available.
Connection to localhost closed.
```

---

### 📋 Complete Session Example

```bash
# Full walkthrough in one session
$ ssh root@192.168.1.100
root@192.168.1.100's password: ********

[root@server ~]# whoami
root

[root@server ~]# cat /etc/shells | grep nologin
/sbin/nologin
/usr/sbin/nologin

[root@server ~]# useradd -m -c "Application Service Account" -s /sbin/nologin appuser

[root@server ~]# grep appuser /etc/passwd
appuser:x:1001:1001:Application Service Account:/home/appuser:/sbin/nologin

[root@server ~]# id appuser
uid=1001(appuser) gid=1001(appuser) groups=1001(appuser)

[root@server ~]# ls -la /home/appuser/
total 12
drwx------ 2 appuser appuser 4096 Dec 25 15:10 .
drwxr-xr-x 4 root    root    4096 Dec 25 15:10 ..
-rw-r--r-- 1 appuser appuser  220 Dec 25 15:10 .bash_logout
-rw-r--r-- 1 appuser appuser  807 Dec 25 15:10 .profile

[root@server ~]# su - appuser
This account is currently not available.

[root@server ~]# echo "User created successfully with non-interactive shell!"
User created successfully with non-interactive shell!

[root@server ~]# exit
logout
Connection to 192.168.1.100 closed.
```

---
