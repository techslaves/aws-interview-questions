# AWS Networking Interview Questions

This document contains common interview questions related to AWS Networking and their answers.

### 1. What is a VPC?
**Answer:**
A **Virtual Private Cloud (VPC)** is a logically isolated virtual network in the AWS cloud where you can launch AWS resources. You have complete control over your virtual networking environment, including selection of your own IP address range, creation of subnets, and configuration of route tables and network gateways.

### 2. What is CIDR and why is it important in VPC?
**Answer:**
**Classless Inter-Domain Routing (CIDR)** is a method for allocating IP addresses and IP routing.
*   **Importance:** It defines the IP address range for your VPC (e.g., `10.0.0.0/16`). It determines the size of your network (how many IPs are available), the size of your subnets, and impacts future scalability.

### 3. Difference between Public and Private Subnet?
**Answer:**
*   **Public Subnet:** Has a route to an **Internet Gateway (IGW)** in its route table. Resources here can communicate directly with the internet.
*   **Private Subnet:** Does **not** have a direct route to the internet. Resources here typically use a NAT Gateway to access the internet securely.

### 4. What makes a subnet public?
**Answer:**
A subnet is considered public if its associated Route Table has a route pointing `0.0.0.0/0` (all traffic) to an **Internet Gateway (IGW)**.

### 5. What is an Internet Gateway?
**Answer:**
An **Internet Gateway (IGW)** is a horizontally scaled, redundant, and highly available VPC component that allows communication between your VPC and the internet. It performs network address translation (NAT) for instances that have been assigned public IPv4 addresses.

### 6. What is a Route Table?
**Answer:**
A **Route Table** contains a set of rules, called routes, that determine where network traffic from your subnet or gateway is directed. Each subnet in your VPC must be associated with a route table.

### 7. How does routing work inside a VPC?
**Answer:**
*   **Local Route:** Every route table has a default `local` route (e.g., `10.0.0.0/16 -> local`) that enables communication between all resources within the VPC. This cannot be deleted.
*   **Longest Prefix Match:** When routing traffic, the most specific route (longest prefix) matches first.

### 8. What is a NAT Gateway?
**Answer:**
A **NAT (Network Address Translation) Gateway** enables instances in a private subnet to connect to the internet or other AWS services, but prevents the internet from initiating a connection with those instances.

### 9. NAT Gateway vs NAT Instance?
**Answer:**

| Feature | NAT Gateway | NAT Instance |
| :--- | :--- | :--- |
| **Management** | Fully Managed by AWS. | Self-managed (EC2 instance). |
| **Scalability** | Scales automatically. | Manual scaling (instance type). |
| **Availability** | Highly available within AZ. | Single point of failure (unless scripted). |
| **Maintenance** | No patching required. | Requires OS patching/updates. |
| **Access** | No SSH access. | Can be used as Bastion/SSH host. |

### 10. Security Group vs NACL?
**Answer:**

| Feature | Security Group (SG) | Network ACL (NACL) |
| :--- | :--- | :--- |
| **Level** | Instance level. | Subnet level. |
| **State** | **Stateful:** Return traffic automatically allowed. | **Stateless:** Return traffic must be explicitly allowed. |
| **Rules** | Allow rules only. | Allow and Deny rules. |
| **Order** | All rules evaluated. | Processed in number order. |

### 11. Why is SG stateful important?
**Answer:**
Being **stateful** means that if you send a request out (e.g., to a web server), the response traffic is automatically allowed back in, regardless of inbound rules. This simplifies configuration as you don't need to manage ephemeral ports for return traffic.

### 12. When would you use NACL deny rules?
**Answer:**
You use NACL deny rules to explicitly block specific IP addresses or ranges (e.g., blocking a known malicious botnet) at the subnet boundary, providing an extra layer of security before traffic even reaches your instances.

### 13. What is Route 53?
**Answer:**
**Amazon Route 53** is a highly available and scalable cloud Domain Name System (DNS) web service. It translates domain names (like `www.example.com`) into numeric IP addresses. It supports public and private DNS, health checks, and traffic routing policies.

### 14. What is a Hosted Zone?
**Answer:**
A **Hosted Zone** is a container for records, and records contain information about how you want to route traffic for a specific domain (e.g., `example.com`) and its subdomains.

### 15. Public vs Private Hosted Zone?
**Answer:**
*   **Public Hosted Zone:** Determines how traffic is routed on the internet. Records are visible publicly.
*   **Private Hosted Zone:** Determines how traffic is routed within an Amazon VPC. Records are only resolvable from within the associated VPCs.

### 16. What is VPC DNS Resolution?
**Answer:**
It is a VPC attribute (`enableDnsSupport`) that determines whether the VPC supports DNS resolution through the Amazon DNS server (`AmazonProvidedDNS`). If enabled, instances can resolve domain names to IP addresses.

### 17. What is VPC Peering?
**Answer:**
**VPC Peering** is a networking connection between two VPCs that enables you to route traffic between them using private IPv4 or IPv6 addresses. Instances in either VPC can communicate with each other as if they are within the same network.

### 18. Limitations of VPC Peering?
**Answer:**
*   **No Transitive Peering:** If A peers with B, and B peers with C, A cannot talk to C via B.
*   **Overlapping CIDRs:** You cannot peer VPCs with matching or overlapping IPv4 CIDR blocks.

### 19. How do you access AWS services privately?
**Answer:**
Using **VPC Endpoints**. They allow you to connect your VPC to supported AWS services (like S3, DynamoDB, SNS) without requiring an Internet Gateway, NAT device, VPN connection, or AWS Direct Connect connection.

### 20. Gateway vs Interface Endpoint?
**Answer:**

| Feature | Gateway Endpoint | Interface Endpoint (PrivateLink) |
| :--- | :--- | :--- |
| **Services** | S3 and DynamoDB only. | Most other AWS services (EC2, SNS, SQS, etc.). |
| **Mechanism** | Uses Route Table entries (Prefix Lists). | Uses ENIs (Elastic Network Interfaces) with private IPs. |
| **Cost** | Free. | Hourly charge + data processing fee. |
| **DNS** | Public DNS resolves to public IP (routed internally). | Private DNS resolves to private IP. |

### 21. What is ENI?
**Answer:**
**Elastic Network Interface (ENI)** is a logical networking component in a VPC that represents a virtual network card. It can include a primary private IPv4 address, one or more secondary private IPv4 addresses, a MAC address, and security groups.

### 22. How does EKS networking work?
**Answer:**
EKS uses the **AWS VPC CNI** plugin. This plugin assigns a native IP address from the VPC subnet to each Pod. This means Pods are first-class citizens on the network and can communicate directly with other resources in the VPC.

### 23. What is MTU and why is it important in AWS?
**Answer:**
**Maximum Transmission Unit (MTU)** is the largest packet size (in bytes) that can be passed over the network.
*   **Default:** 1500 bytes (Ethernet).
*   **Jumbo Frames:** 9001 bytes (supported within VPC).
*   **Importance:** Incorrect MTU settings (e.g., sending jumbo frames over a path that doesn't support them, like the internet) can cause packet fragmentation or drops, leading to connectivity issues.

### 24. How do you design multi-AZ high availability?
**Answer:**
*   **Subnets:** Create subnets in at least two different Availability Zones (AZs).
*   **Load Balancing:** Use an Application Load Balancer (ALB) to distribute traffic across instances in multiple AZs.
*   **Auto Scaling:** Configure Auto Scaling Groups to launch instances across multiple AZs.
*   **NAT:** Deploy a NAT Gateway in each AZ to ensure outbound connectivity resilience.
