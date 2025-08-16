# Troubleshooting Guide

This document covers common issues you might encounter while implementing the AWS VPC connectivity project and their solutions.

## Table of Contents
- [EC2 Instance Connect Issues](#ec2-instance-connect-issues)
- [Connectivity Problems](#connectivity-problems)
- [Security Group Issues](#security-group-issues)
- [Network ACL Problems](#network-acl-problems)
- [Internet Gateway Issues](#internet-gateway-issues)
- [General Debugging Steps](#general-debugging-steps)

---

## EC2 Instance Connect Issues

### Problem: "Failed to connect to your instance"
**Symptoms:**
- EC2 Instance Connect shows connection error
- Browser-based SSH fails to establish connection

**Common Causes & Solutions:**

#### 1. Missing SSH Rule in Security Group
**Solution:**
1. Go to EC2 Console → Security Groups
2. Select the security group attached to your instance
3. Add inbound rule:
   - Type: SSH
   - Port: 22
   - Source: 0.0.0.0/0 (for testing) or your IP range

#### 2. Instance Not in Public Subnet
**Solution:**
- Verify instance is launched in public subnet
- Check if subnet has route to Internet Gateway (0.0.0.0/0 → igw-xxxxx)

#### 3. No Public IP Assigned
**Solution:**
- Terminate and relaunch instance with "Auto-assign Public IP" enabled
- Or allocate and associate an Elastic IP

---

## Connectivity Problems

### Problem: Cannot ping between public and private servers
**Symptoms:**
- `ping 10.0.1.x` times out from public server
- No response from private server

**Diagnostic Steps:**
1. Check Security Groups
2. Check Network ACLs
3. Verify subnet routes

#### Solution 1: Security Group Configuration
**Private Server Security Group should have:**
```
Inbound Rules:
- Type: All ICMP - IPv4
- Source: [Public Security Group ID]
```

#### Solution 2: Network ACL Configuration
**Private Subnet NACL should have:**
```
Inbound Rules:
- Rule #: 100
- Type: All ICMP - IPv4
- Source: 10.0.0.0/24

Outbound Rules:
- Rule #: 100
- Type: All ICMP - IPv4
- Destination: 10.0.0.0/24
```

### Problem: SSH from public to private server fails
**Symptoms:**
- `ssh ec2-user@10.0.1.x` hangs or times out
- Connection refused error

**Solution:**
1. **Security Group**: Ensure private server's security group allows SSH from public security group
2. **Network ACL**: Add ephemeral port rules (1024-65535) for return traffic
3. **Key Pair**: Ensure same key pair is used or copy public key to private server

---

## Security Group Issues

### Problem: Rules not taking effect immediately
**Solution:**
- Security group changes are applied immediately
- If still not working, check if you're modifying the correct security group
- Verify the security group is actually attached to the instance

### Problem: Confusion between source types
**Common Mistakes:**
- Using IP address instead of security group ID for internal communication
- Using security group ID instead of CIDR for internet access

**Best Practices:**
- Use Security Group IDs for internal AWS resource communication
- Use CIDR blocks (0.0.0.0/0) for internet access
- Use specific IP ranges for known external sources

---

## Network ACL Problems

### Problem: Stateless nature causing connection issues
**Understanding NACLs:**
- NACLs are stateless (unlike security groups)
- Both inbound AND outbound rules needed
- Rules processed in order (lowest number first)
- Explicit DENY overrides ALLOW

**Common Fix:**
```
For SSH (port 22):
Inbound: Allow SSH (22) from source
Outbound: Allow Ephemeral Ports (1024-65535) to source

For HTTP/HTTPS:
Inbound: Allow HTTP/HTTPS (80/443) from 0.0.0.0/0
Outbound: Allow Ephemeral Ports (1024-65535) to 0.0.0.0/0
```

### Problem: NACL rules conflict
**Solution:**
1. Check rule numbers - lower numbers process first
2. Ensure no explicit DENY rules blocking traffic
3. Remember: if no rule matches, traffic is denied by default

---

## Internet Gateway Issues

### Problem: Public server cannot reach internet
**Diagnostic Steps:**
1. **Internet Gateway**: Ensure IGW is attached to VPC
2. **Route Table**: Public subnet should have route 0.0.0.0/0 → igw-xxxxx
3. **Security Group**: Outbound rules should allow HTTP/HTTPS
4. **Public IP**: Instance must have public IP or Elastic IP

**Solution Checklist:**
```bash
# Test from public server
curl -I https://www.google.com
nslookup google.com

# Check route table
# Should show: 0.0.0.0/0 → igw-xxxxx for public subnet
```

### Problem: Private server accidentally has internet access
**This should NOT happen in our setup. If it does:**
1. Check if private subnet route table has IGW route
2. Verify instance is actually in private subnet
3. Check if NAT Gateway was accidentally created

---

## General Debugging Steps

### 1. Connectivity Testing Commands
```bash
# Test internet connectivity
curl -I https://www.google.com
ping 8.8.8.8

# Test internal connectivity
ping 10.0.1.x  # from public to private
ping 10.0.0.x  # from private to public

# Test DNS resolution
nslookup google.com

# Check network configuration
ip route
ip addr show
```

### 2. AWS CLI Debugging Commands
```bash
# Check security groups
aws ec2 describe-security-groups --group-ids sg-xxxxx

# Check NACLs
aws ec2 describe-network-acls --network-acl-ids acl-xxxxx

# Check route tables
aws ec2 describe-route-tables --route-table-ids rtb-xxxxx

# Check instance details
aws ec2 describe-instances --instance-ids i-xxxxx
```

### 3. Systematic Troubleshooting Approach

#### Step 1: Verify Instance Details
- Instance is running
- Correct subnet assignment
- Security group attachment
- Public IP (for public instance)

#### Step 2: Check Security Groups
- Correct inbound/outbound rules
- Source/destination properly configured
- Rules applied to correct instances

#### Step 3: Check Network ACLs
- Associated with correct subnet
- Inbound AND outbound rules present
- Rule numbers in correct order
- No conflicting DENY rules

#### Step 4: Check Route Tables
- Subnet associations correct
- Routes pointing to correct targets
- IGW attached and routed (for public)

#### Step 5: Check VPC Configuration
- DNS resolution enabled
- DNS hostnames enabled
- IGW attached to VPC

---

## Common Error Messages and Solutions

### "Connection timed out"
- Usually security group or NACL blocking traffic
- Check if ports are open in both security groups and NACLs

### "Network is unreachable"
- Route table issue
- No route to destination network

### "Connection refused"
- Service not running on target port
- Firewall (security group) blocking specific port

### "Permission denied (publickey)"
- SSH key issue
- Wrong username (use `ec2-user` for Amazon Linux)
- Key not properly configured

---

## Getting Help

If you're still experiencing issues:

1. **AWS Documentation**: [VPC Troubleshooting Guide](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-troubleshooting.html)
2. **AWS Support**: Consider AWS Support if you have a support plan
3. **AWS Forums**: [AWS Developer Forums](https://forums.aws.amazon.com/)
4. **VPC Flow Logs**: Enable for detailed network traffic analysis

## Prevention Tips

1. **Document Changes**: Keep track of security group and NACL modifications
2. **Test Incrementally**: Test connectivity after each configuration change
3. **Use Descriptive Names**: Name security groups and NACLs clearly
4. **Follow Principle of Least Privilege**: Only open ports that are absolutely necessary
5. **Regular Audits**: Periodically review security group rules and remove unused ones
