# AWS VPC Connectivity Project

A comprehensive guide to setting up a secure AWS VPC with public and private subnets, demonstrating proper network isolation and connectivity patterns.

## 🎯 Project Goal

This project demonstrates how to create a secure AWS VPC architecture where:
- **Public server** (EC2 in public subnet) has internet access
- **Private server** (EC2 in private subnet) is isolated from internet but accessible from public server
- Proper security controls using NACLs and Security Groups

## 🏗️ Architecture Overview

```
Internet
    ↓
Internet Gateway
    ↓
Public Subnet (10.0.0.0/24)
    ↓
Public Server ←→ Private Server
                    ↓
            Private Subnet (10.0.1.0/24)
```

## 🚀 Quick Start

1. **Create VPC** - Set up VPC with public/private subnets
2. **Configure Network ACLs** - Set subnet-level security rules
3. **Setup Security Groups** - Configure instance-level firewall rules
4. **Launch EC2 Instances** - Deploy servers in respective subnets
5. **Test Connectivity** - Verify internet and inter-subnet communication

## 📋 Prerequisites

- AWS Account with appropriate permissions
- Basic understanding of VPC, subnets, and security groups
- AWS CLI or Console access

## 🛠️ Implementation

For detailed step-by-step instructions, see [IMPLEMENTATION.md](./IMPLEMENTATION.md)

## 🔧 Troubleshooting

For common issues and solutions, see [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)

## 📸 Screenshots

All implementation screenshots are available in the [images/](./images/) directory.

## 🔒 Security Considerations

- **Principle of Least Privilege**: Only necessary ports are opened
- **Network Segmentation**: Public and private subnets provide isolation
- **Defense in Depth**: Both NACLs and Security Groups provide layered security
- **No Internet Access for Private Resources**: Private subnet has no route to Internet Gateway

## 🧪 Testing Results

- ✅ Public server can access internet
- ✅ Public server can communicate with private server
- ✅ Private server is isolated from internet
- ✅ SSH access works through proper security group configuration

## 📚 Resources

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [EC2 Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)

## 🤝 Contributing

Feel free to open issues or submit pull requests if you find improvements or have questions!

## 📄 License

This project is for educational purposes and is provided as-is.
