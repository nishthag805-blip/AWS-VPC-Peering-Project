# AWS VPC Peering and Secure EC2 Connectivity

## 📌 Project Overview

This project demonstrates how to establish private network communication between two Amazon Virtual Private Clouds (VPCs) using **AWS VPC Peering**. It focuses on connecting EC2 instances located in different VPCs while maintaining network isolation and secure access.

The project uses a public bastion host in VPC A to provide administrative SSH access to private EC2 instances. VPC peering enables communication between private workloads in VPC A and VPC B without requiring public IP addresses on the application instances.

**Project Type:** Cloud Computing / AWS Networking  
**Platform:** Amazon Web Services (AWS)  
**Services:** Amazon VPC, Amazon EC2, Internet Gateway, Route Tables, Security Groups, VPC Peering

## 🎯 Project Objectives

- Understand AWS VPC and subnet architecture.
- Create public and private subnets.
- Configure an Internet Gateway for controlled external access.
- Establish a VPC peering connection between two VPCs.
- Configure route tables for communication between VPCs.
- Use a bastion host to access private EC2 instances securely.
- Test connectivity between instances using private IP addresses.
- Understand network security, troubleshooting, and AWS cost management.

## 🏗️ Architecture Overview

The architecture contains two VPCs connected through a VPC peering connection.

### VPC A — Primary VPC

- **CIDR Block:** `10.10.0.0/16`
- **Public Subnet:** `10.10.1.0/24`
- **Private Subnet:** `10.10.2.0/24`
- **Bastion Host:** `10.10.1.10`
- **Private Instance App-A:** `10.10.2.10`
- **Internet Gateway:** Attached to VPC A

### VPC B — Secondary VPC

- **CIDR Block:** `10.20.0.0/16`
- **Private Subnet:** `10.20.1.0/24`
- **Private Instance App-B:** `10.20.1.10`
- **Internet Gateway:** None in this design

### Network Design

| Component | CIDR / Private IP | Purpose |
|---|---|---|
| VPC A | `10.10.0.0/16` | Primary network |
| Public subnet A | `10.10.1.0/24` | Bastion host |
| Private subnet A | `10.10.2.0/24` | App-A |
| VPC B | `10.20.0.0/16` | Secondary network |
| Private subnet B | `10.20.1.0/24` | App-B |
| VPC Peering | Between VPC A and VPC B | Private communication |

> **Note:** These CIDR ranges are the consistent addressing plan used throughout this README. The accompanying illustrative diagram may show different `28.x.x.x` ranges. Update the diagram to match this plan before publishing it.

## 🔄 How VPC Peering Works

VPC peering creates a private network connection between two VPCs using their private IP addresses.

Communication requires the following configuration:

1. Create the two VPCs and their subnets.
2. Launch the EC2 instances in the appropriate subnets.
3. Create and accept the VPC peering connection.
4. Add routes to the peer VPC CIDR in the relevant route tables.
5. Configure security groups to allow only the required traffic.
6. Connect to private instances through the bastion host.
7. Test connectivity between App-A and App-B.

VPC peering is not automatically transitive. A peered VPC also does not automatically gain access to the other VPC's Internet Gateway.

## 🛣️ Route Table Configuration

The following table describes the intended routing configuration.

| Route Table | Destination | Target |
|---|---|---|
| VPC A public | `10.10.0.0/16` | Local |
| VPC A public | `0.0.0.0/0` | Internet Gateway |
| VPC A private | `10.10.0.0/16` | Local |
| VPC A private | `10.20.0.0/16` | VPC Peering |
| VPC B private | `10.20.0.0/16` | Local |
| VPC B private | `10.10.0.0/16` | VPC Peering |

The public route table provides internet routing for the bastion host, subject to its subnet configuration and security controls. Private route tables direct traffic for the other VPC through the peering connection.

## 🔐 Security Configuration

Security groups should follow the principle of least privilege.

- Allow SSH access to the bastion host only from your trusted public IP address.
- Allow SSH from the bastion host to the private EC2 instances.
- Allow required application traffic between the private instances.
- Restrict ICMP traffic to the source addresses needed for connectivity testing.
- Avoid allowing SSH or ICMP from `0.0.0.0/0`.
- Do not assign public IP addresses to App-A or App-B in this design.
- Never upload private SSH keys, AWS credentials, access tokens, or other secrets to GitHub.

The exact security group rules depend on the ports and protocols used in the lab.

## 💻 Accessing Private EC2 Instances

Private EC2 instances can be accessed through the bastion host using SSH ProxyJump.

Example command:

```bash
ssh -J ec2-user@BASTION_PUBLIC_IP ec2-user@10.10.2.10
```

Replace `BASTION_PUBLIC_IP` with the bastion host's actual public IP address. The command assumes the appropriate SSH key and username are configured.

To connect to App-B, use the same approach with the appropriate network path and target private IP:

```bash
ssh -J ec2-user@BASTION_PUBLIC_IP ec2-user@10.20.1.10
```

These commands are examples, not proof that the connection has been configured or tested. Ensure that routing, security groups, SSH keys, and host access are configured correctly.

## 🧪 Testing and Validation

The following tests can be performed after the infrastructure is deployed.

| Test | Expected Result |
|---|---|
| Verify VPC peering status | Active |
| Check route table entries | Required local and peering routes exist |
| SSH from an authorized client to the bastion | Successful |
| SSH from the bastion to App-A | Successful when permitted |
| Test private connectivity between App-A and App-B | Successful when routes and security rules permit it |
| Test ICMP between private instances | Successful if ICMP is allowed |
| Remove a required peering route temporarily | Cross-VPC connectivity should fail |
| Restore the route | Cross-VPC connectivity should recover |

**Testing status:** Pending deployment and validation. Expected results are listed above; no successful execution is claimed by this documentation.

## 🔍 Troubleshooting

### 1. VPC Peering Is Not Active
- Verify the peering request has been accepted.
- Confirm the peering connection is active.
- Check that both VPCs are in the intended AWS Regions and accounts.

### 2. EC2 Instances Cannot Communicate
- Verify routes to the peer VPC CIDR in both VPCs.
- Confirm the subnet route tables are correctly associated.
- Review security group inbound and outbound rules.
- Check network ACL rules and instance-level firewall settings.
- Confirm the destination instance is running and listening on the required port.

### 3. SSH Connection Fails
- Verify the bastion's public IP and security group.
- Confirm that the SSH private key is available locally and has appropriate permissions.
- Check the username for the chosen EC2 operating system.
- Verify that the bastion can reach the target's private IP on TCP port 22.

### 4. Ping Fails
- ICMP may be blocked by security groups, network ACLs, or host firewalls.
- A failed ping does not necessarily mean that TCP application traffic is unavailable.
- Test the actual application port where appropriate.

## 💰 Cost Considerations

AWS resources may incur charges even when used for learning.

Before deploying this project:

- Review current Amazon EC2 pricing for the selected instance types.
- Check applicable public IPv4 address charges.
- Review any data transfer charges.
- Check whether other billable resources are being created.
- Use AWS Budgets or billing alerts where appropriate.
- Delete resources after testing.

VPC peering itself does not have an hourly connection fee, but applicable data transfer charges may apply. Verify current pricing for your Region before deployment.

## 🧹 Cleanup

After completing the lab:

1. Terminate the EC2 instances.
2. Delete the VPC peering connection.
3. Delete the Internet Gateway after detaching it.
4. Remove associated route tables and security groups as appropriate.
5. Delete the subnets.
6. Delete the VPCs after all dependent resources have been removed.
7. Review the AWS billing console to check for remaining resources.

Do not delete resources used by other projects or workloads.

## 📚 Key Learnings

This project is designed to develop practical knowledge of:

- AWS VPC architecture and CIDR planning.
- Public and private subnet configuration.
- Route tables and destination-based routing.
- VPC peering and private IP communication.
- Bastion-host-based administrative access.
- Security groups and network access control.
- Connectivity testing and troubleshooting.
- Cloud cost awareness and resource cleanup.

## 🚀 Future Improvements

- Automate the infrastructure using Terraform or AWS CloudFormation.
- Add a private application service and test TCP connectivity.
- Use AWS Systems Manager Session Manager where appropriate to reduce reliance on inbound SSH.
- Add automated connectivity checks.
- Create a deployment checklist and test-results report.
- Improve the architecture diagram and document verified deployment evidence.

## 👨‍💻 Author

**Nishtha Gupta**

Cloud Computing | AWS Networking | EC2 | VPC | Cloud Security

---

**Disclaimer:** This repository documents a learning project and proposed implementation. Actual deployment, test results, and costs must be verified in the AWS account used for the lab.
