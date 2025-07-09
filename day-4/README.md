# VPC

Imagine you want to set up a private, secure, and isolated area in the cloud where you can run your applications and store your data. This is where a VPC comes into play.

A VPC is a virtual network that you create in the cloud. It allows you to have your own private section of the internet, just like having your own network within a larger network. Within this VPC, you can create and manage various resources, such as servers, databases, and storage.

Think of it as having your own little "internet" within the bigger internet. This virtual network is completely isolated from other users' networks, so your data and applications are secure and protected.

Just like a physical network, a VPC has its own set of rules and configurations. You can define the IP address range for your VPC and create smaller subnetworks within it called subnets. These subnets help you organize your resources and control how they communicate with each other.

To connect your VPC to the internet or other networks, you can set up gateways or routers. These act as entry and exit points for traffic going in and out of your VPC. You can control the flow of traffic and set up security measures to protect your resources from unauthorized access.

With a VPC, you have control over your network environment. You can define access rules, set up firewalls, and configure security groups to regulate who can access your resources and how they can communicate.

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/12cc10b6-724c-42c9-b07b-d8a7ce124e24)

By default, when you create an AWS account, AWS will create a default VPC for you but this default VPC is just to get started with AWS. You should create VPCs for applications or projects. 

## VPC components 

The following features help you configure a VPC to provide the connectivity that your applications need:
| Component                  | Description                                                               |
| -------------------------- | ------------------------------------------------------------------------- |
| **VPC**                    | Your isolated cloud network environment.                                  |
| **Public Subnet**          | Subnet with a route to the internet via the Internet Gateway.             |
| **Private Subnet**         | No direct internet access. Typically used for backend servers.            |
| **Internet Gateway (IGW)** | Enables public subnet instances to access the internet.                   |
| **NAT Gateway (Optional)** | Allows private subnet instances to access the internet for updates, etc.  |
| **Route Tables**           | Define how traffic is routed between subnets and outside networks.        |
| **Load Balancer**          | Distributes incoming traffic to instances. Supports Public/Private types. |



Virtual private clouds (VPC)

    A VPC is a virtual network that closely resembles a traditional network that you'd operate in your own data center. After you create a VPC, you can add subnets.
Subnets

    A subnet is a range of IP addresses in your VPC. A subnet must reside in a single Availability Zone. After you add subnets, you can deploy AWS resources in your VPC.
IP addressing

    You can assign IP addresses, both IPv4 and IPv6, to your VPCs and subnets. You can also bring your public IPv4 and IPv6 GUA addresses to AWS and allocate them to resources in your VPC, such as EC2 instances, NAT gateways, and Network Load Balancers.

Network Access Control List (NACL)

    A Network Access Control List is a stateless firewall that controls inbound and outbound traffic at the subnet level. It operates at the IP address level and can allow or deny traffic based on rules that you define. NACLs provide an additional layer of network security for your VPC.
   
Security Group

    A security group acts as a virtual firewall for instances (EC2 instances or other resources) within a VPC. It controls inbound and outbound traffic at the instance level. Security groups allow you to define rules that permit or restrict traffic based on protocols, ports, and IP addresses.  

Routing

    Use route tables to determine where network traffic from your subnet or gateway is directed.
Gateways and endpoints

    A gateway connects your VPC to another network. For example, use an internet gateway to connect your VPC to the internet. Use a VPC endpoint to connect to AWS services privately, without the use of an internet gateway or NAT device.
Peering connections

    Use a VPC peering connection to route traffic between the resources in two VPCs.
Traffic Mirroring

    Copy network traffic from network interfaces and send it to security and monitoring appliances for deep packet inspection.
Transit gateways

    Use a transit gateway, which acts as a central hub, to route traffic between your VPCs, VPN connections, and AWS Direct Connect connections.
VPC Flow Logs

    A flow log captures information about the IP traffic going to and from network interfaces in your VPC.
VPN connections

    Connect your VPCs to your on-premises networks using AWS Virtual Private Network (AWS VPN).


## Resources 

VPC with servers in private subnets and NAT

https://docs.aws.amazon.com/vpc/latest/userguide/vpc-example-private-subnets-nat.html

![image](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/89d8316e-7b70-4821-a6bf-67d1dcc4d2fb)

![image](https://github.com/user-attachments/assets/a4d12ba8-0ecc-4bda-9848-3f94e3da24ce)


You're looking for a **complete AWS VPC architecture** covering:

> **Public & Private Subnets, Internet Gateway, Elastic Load Balancer, Route Tables, NAT Gateway, Target Group, Security Groups, and NACLs**, both for:

* Inbound → Load Balancer → Private Subnet EC2 (via Target Group)
* Outbound → Private Subnet EC2 → Internet (via NAT Gateway)

---

## ✅ **High-Level Architecture:**

```
           +----------------------+
           |   Internet Gateway   |
           +----------+-----------+
                      |
              +-------v--------+                   << PUBLIC SIDE >>
              | Route Table 1  |----+
              +-------+--------+    |
                      |             |
       +--------------+-------------+--------------+
       |              |                            |
+------v----+   +-----v-----+              +-------v--------+
| PubSubnet |   | PubSubnet |              | Elastic Load   |
| (NAT-GW)  |   | (ALB)     |              | Balancer (ALB) |
+-----------+   +-----------+              +----------------+
     |              |                             |
     |              |                      Forwards to Target Group
     |              |                             |
     |              |                      +------v-------+
     |              +--------------------->| Pvt Subnet  |
     |                                     | EC2 (App)    |
     |                                     +--------------+
     |
     |       << OUTBOUND FLOW >>
     |                             +--------------+
     +-----> NAT Gateway ----------> Internet GW  |
                                   +--------------+
```

---

## 🏗️ **Step-by-Step Setup**

---

### 🔹 **1. VPC**

* CIDR: `10.0.0.0/16`

---

### 🔹 **2. Subnets**

| Subnet Name      | CIDR Block   | AZ  | Type    | Use                     |
| ---------------- | ------------ | --- | ------- | ----------------------- |
| Public Subnet 1  | 10.0.1.0/24  | `a` | Public  | ALB, NAT Gateway        |
| Public Subnet 2  | 10.0.2.0/24  | `b` | Public  | Redundancy              |
| Private Subnet 1 | 10.0.10.0/24 | `a` | Private | App EC2s (Target Group) |
| Private Subnet 2 | 10.0.11.0/24 | `b` | Private | App EC2s                |

---

### 🔹 **3. Internet Gateway**

* Attach to VPC

---

### 🔹 **4. NAT Gateway**

* In **Public Subnet**
* Associate Elastic IP
* Used by **Private Subnet EC2s** for **outbound internet access**

---

### 🔹 **5. Route Tables**

#### 🔸 Public Route Table (attached to Public Subnets)

| Destination | Target           |
| ----------- | ---------------- |
| `0.0.0.0/0` | Internet Gateway |

#### 🔸 Private Route Table (attached to Private Subnets)

| Destination | Target      |
| ----------- | ----------- |
| `0.0.0.0/0` | NAT Gateway |

---

### 🔹 **6. Load Balancer (ALB - Application Load Balancer)**

* Deployed in **Public Subnets**
* Forwards traffic to **Target Group** with EC2s in **Private Subnet**
* Listener: HTTP (80) or HTTPS (443)

---

### 🔹 **7. Target Group**

* Targets: EC2 instances in **Private Subnets**
* Health checks: e.g., `/health`

---

## 🔐 **8. Security Groups**

### ALB Security Group (Public)

| Type     | Protocol | Port | Source      |
| -------- | -------- | ---- | ----------- |
| Inbound  | TCP      | 80   | `0.0.0.0/0` |
| Outbound | ALL      | ALL  | `0.0.0.0/0` |

### EC2 Security Group (Private)

| Type     | Protocol | Port | Source (SG)              |
| -------- | -------- | ---- | ------------------------ |
| Inbound  | TCP      | 80   | ALB Security Group       |
| Outbound | ALL      | ALL  | `0.0.0.0/0` (via NAT GW) |

---

## 🔒 **9. Network ACLs (NACLs)**

> **NACLs are stateless**, so you must allow both inbound and outbound rules.

### Public Subnet NACL (for ALB & NAT GW)

**Inbound Rules:**

| Rule # | Protocol | Port Range | Source    | Allow/Deny |
| ------ | -------- | ---------- | --------- | ---------- |
| 100    | TCP      | 80         | 0.0.0.0/0 | ALLOW      |
| 110    | TCP      | 443        | 0.0.0.0/0 | ALLOW      |

**Outbound Rules:**

| Rule # | Protocol | Port Range | Destination | Allow/Deny |
| ------ | -------- | ---------- | ----------- | ---------- |
| 100    | TCP      | 1024-65535 | 0.0.0.0/0   | ALLOW      |

---

### Private Subnet NACL (for EC2s)

**Inbound Rules:**

| Rule # | Protocol | Port Range | Source             | Allow/Deny |
| ------ | -------- | ---------- | ------------------ | ---------- |
| 100    | TCP      | 80         | Public Subnet CIDR | ALLOW      |
| 110    | TCP      | 443        | Public Subnet CIDR | ALLOW      |

**Outbound Rules:**

| Rule # | Protocol | Port Range | Destination | Allow/Deny |
| ------ | -------- | ---------- | ----------- | ---------- |
| 100    | TCP      | 80,443     | 0.0.0.0/0   | ALLOW      |

---

## 🔁 **Traffic Flow Summary**

### 🔸 Inbound (Client → App):

1. **Client** hits ALB DNS (via Internet).
2. ALB (in Public Subnet) forwards traffic to **Target Group** (EC2 in Private Subnet).
3. EC2 responds back via ALB → Internet Gateway.

### 🔸 Outbound (App → Internet):

1. Private EC2 makes HTTP request.
2. Route table directs traffic to **NAT Gateway** (in Public Subnet).
3. NAT Gateway → Internet Gateway → Internet.

---

Would you like:

* **Architecture diagram?**
* **Terraform or CloudFormation templates?**
* **Step-by-step AWS Console screenshots/guide?**

Let me know your preferred format.



