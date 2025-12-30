# Day 3: Secure Root SSH Access - Stratos Datacenter

## 📋 Task Overview

Following security audits, the xFusionCorp Industries security team has rolled out new protocols, including the restriction of direct root SSH login.

**Objective:** Disable direct SSH root login on all app servers within the Stratos Datacenter.

---

## 🔐 Security Context

**Why disable root SSH login?**

- Prevents brute-force attacks targeting the root account
- Enforces accountability (users must log in with their own accounts)
- Follows security best practices and compliance requirements
- Reduces attack surface for unauthorized access

---

## ✅ Manual Steps (Per Server)

### 1️⃣ Connect to the Server

```bash
# Log in as a sudo-capable user (NOT root)
ssh user@app-server-ip
```

### 2️⃣ Edit SSH Configuration

```bash
sudo vi /etc/ssh/sshd_config
# Or use nano if preferred
sudo nano /etc/ssh/sshd_config
```

### 3️⃣ Update PermitRootLogin Setting

Find the line containing `PermitRootLogin` and change it:

**From:**

```
PermitRootLogin yes
# OR
#PermitRootLogin yes
```

**To:**

```
PermitRootLogin no
```

⚠️ **Important:** If the line is commented out (starts with `#`), uncomment it first.

### 4️⃣ Restart SSH Service

```bash
# For systemd-based systems (RHEL/CentOS 7+, Ubuntu 16+)
sudo systemctl restart sshd

# Alternative for some Ubuntu systems
sudo systemctl restart ssh

# For older init.d systems
sudo service sshd restart
```

### 5️⃣ Verify the Change

```bash
# Check the configuration
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config

# Should output: PermitRootLogin no
```

### 6️⃣ Test (Optional but Recommended)

```bash
# From another terminal, try to SSH as root
ssh root@app-server-ip

# Expected result: Permission denied
```

---

## 🚀 Automated Solution

### Script 1: Single Server Automation

Save as `disable-root-ssh.sh`:

```bash
#!/bin/bash
# Script to disable root SSH login on a single server

set -e

echo "🔒 Disabling root SSH login..."

# Backup the original configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)
echo "✅ Backup created"

# Update PermitRootLogin setting
if sudo grep -q "^PermitRootLogin" /etc/ssh/sshd_config; then
    # Setting exists and is uncommented
    sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
    echo "✅ Updated existing PermitRootLogin setting"
elif sudo grep -q "^#PermitRootLogin" /etc/ssh/sshd_config; then
    # Setting exists but is commented
    sudo sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
    echo "✅ Uncommented and updated PermitRootLogin setting"
else
    # Setting doesn't exist, add it
    echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config > /dev/null
    echo "✅ Added PermitRootLogin no to config"
fi

# Verify the change
if sudo grep -q "^PermitRootLogin no" /etc/ssh/sshd_config; then
    echo "✅ Configuration verified"
else
    echo "❌ Configuration verification failed!"
    exit 1
fi

# Restart SSH service
if sudo systemctl restart sshd 2>/dev/null || sudo systemctl restart ssh 2>/dev/null; then
    echo "✅ SSH service restarted successfully"
else
    echo "❌ Failed to restart SSH service"
    exit 1
fi

echo "🎉 Root SSH login has been disabled successfully!"
```

**Usage:**

```bash
chmod +x disable-root-ssh.sh
./disable-root-ssh.sh
```

### Script 2: Multi-Server Deployment

Save as `disable-root-ssh-all-servers.sh`:

```bash
#!/bin/bash
# Script to disable root SSH login across multiple app servers

# Define your app servers
APP_SERVERS=(
    "user@app-server-1"
    "user@app-server-2"
    "user@app-server-3"
)

# Color codes for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

echo "🚀 Starting deployment to ${#APP_SERVERS[@]} servers..."
echo ""

for server in "${APP_SERVERS[@]}"; do
    echo -e "${YELLOW}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    echo -e "${YELLOW}Processing: $server${NC}"
    echo -e "${YELLOW}━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━${NC}"
    
    # Execute commands on remote server
    ssh -t "$server" << 'ENDSSH'
        set -e
        
        # Backup configuration
        sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup.$(date +%Y%m%d_%H%M%S)
        
        # Update PermitRootLogin setting
        if sudo grep -q "^PermitRootLogin" /etc/ssh/sshd_config; then
            sudo sed -i 's/^PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
        elif sudo grep -q "^#PermitRootLogin" /etc/ssh/sshd_config; then
            sudo sed -i 's/^#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
        else
            echo "PermitRootLogin no" | sudo tee -a /etc/ssh/sshd_config > /dev/null
        fi
        
        # Restart SSH service
        sudo systemctl restart sshd 2>/dev/null || sudo systemctl restart ssh 2>/dev/null
        
        # Verify
        sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
ENDSSH
    
    if [ $? -eq 0 ]; then
        echo -e "${GREEN}✅ $server: SUCCESS${NC}"
    else
        echo -e "${RED}❌ $server: FAILED${NC}"
    fi
    echo ""
done

echo -e "${GREEN}🎉 Deployment complete!${NC}"
```

**Usage:**

```bash
# Edit the script and update APP_SERVERS array with your actual servers
chmod +x disable-root-ssh-all-servers.sh
./disable-root-ssh-all-servers.sh
```

### Script 3: Compliance Verification

Save as `verify-root-ssh-disabled.sh`:

```bash
#!/bin/bash
# Script to verify root SSH login is disabled across all servers

# Define your app servers
APP_SERVERS=(
    "user@app-server-1"
    "user@app-server-2"
    "user@app-server-3"
)

# Color codes
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

echo -e "${BLUE}🔍 Verifying SSH root login status across servers...${NC}"
echo ""

COMPLIANT=0
NON_COMPLIANT=0

for server in "${APP_SERVERS[@]}"; do
    echo -n "Checking $server ... "
    
    RESULT=$(ssh "$server" "sudo grep '^PermitRootLogin' /etc/ssh/sshd_config" 2>/dev/null)
    
    if [[ "$RESULT" == "PermitRootLogin no" ]]; then
        echo -e "${GREEN}✅ COMPLIANT${NC}"
        ((COMPLIANT++))
    else
        echo -e "${RED}❌ NON-COMPLIANT${NC} (Current: $RESULT)"
        ((NON_COMPLIANT++))
    fi
done

echo ""
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo -e "${BLUE}Summary:${NC}"
echo -e "${GREEN}Compliant servers: $COMPLIANT${NC}"
echo -e "${RED}Non-compliant servers: $NON_COMPLIANT${NC}"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ $NON_COMPLIANT -eq 0 ]; then
    echo -e "${GREEN}🎉 All servers are compliant!${NC}"
    exit 0
else
    echo -e "${RED}⚠️  Some servers are not compliant. Please investigate.${NC}"
    exit 1
fi
```

**Usage:**

```bash
chmod +x verify-root-ssh-disabled.sh
./verify-root-ssh-disabled.sh
```

---

## 🛡️ Pre-Flight Checklist

Before disabling root SSH login, ensure:

- [ ] At least one non-root user with sudo privileges exists
- [ ] You have tested sudo access with the non-root user
- [ ] You have an alternative access method (console access, etc.)
- [ ] You have backed up `/etc/ssh/sshd_config`
- [ ] You understand the current authentication method

---

## 🔄 Rollback Procedure

If you need to restore root SSH access:

```bash
# Restore from backup
sudo cp /etc/ssh/sshd_config.backup.YYYYMMDD_HHMMSS /etc/ssh/sshd_config

# OR manually edit
sudo sed -i 's/^PermitRootLogin no/PermitRootLogin yes/' /etc/ssh/sshd_config

# Restart SSH service
sudo systemctl restart sshd
```

---

## 📊 Verification Commands

```bash
# Check current setting
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config

# Check SSH service status
sudo systemctl status sshd

# View SSH configuration test
sudo sshd -t

# View recent SSH authentication logs
sudo tail -f /var/log/auth.log        # Debian/Ubuntu
sudo tail -f /var/log/secure          # RHEL/CentOS
```

---

## 🎯 KodeKloud Environment Specifics

For typical KodeKloud Stratos Datacenter setup:

**App Servers:**

- App Server 1 (`stapp01`)
- App Server 2 (`stapp02`)
- App Server 3 (`stapp03`)

**Common Credentials:**

- Jump Host/Thor: `thor` (password usually provided)
- App Servers: User credentials vary (check task description)

**Example workflow:**

```bash
# From Jump Host (Thor)
ssh tony@stapp01    # or banner/steve for stapp02/stapp03

# Once connected to app server
sudo vi /etc/ssh/sshd_config
# Make changes
sudo systemctl restart sshd

# Repeat for other app servers
```

---
