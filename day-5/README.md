# AWS Security using Security Groups and NACL 

AWS (Amazon Web Services) provides multiple layers of security to protect resources and data within its cloud infrastructure. Two important components for network security in AWS are Security Groups and Network Access Control Lists (NACLs). Let's explore how each of them works:
Thank you! You've shared a well-structured explanation of **AWS Security using Security Groups and NACLs**. Below is a slightly polished and clarified version, ready for documentation, study material, or inclusion in a report.

---

## 🛡️ **AWS Security using Security Groups and Network Access Control Lists (NACLs)**

AWS (Amazon Web Services) implements multiple layers of security to protect cloud infrastructure, applications, and data. Two key network security components within a **VPC (Virtual Private Cloud)** are:

---

### 🔐 **Security Groups**

* **Security Groups** act as **virtual firewalls** for EC2 instances at the **instance level**.
* They control **inbound and outbound traffic** by **allowing** specific protocols, ports, and IP addresses (they do **not** support deny rules).
* Each EC2 instance can be associated with **one or more Security Groups**.
* **Inbound rules** control what traffic is allowed **into** the instance.
* **Outbound rules** control what traffic is allowed **out of** the instance.
* Rules can be defined using:

  * IP addresses (e.g., `192.168.1.1/32`)
  * CIDR blocks (e.g., `10.0.0.0/16`)
  * Security Group IDs (referencing other SGs)
  * DNS names (only in some services like RDS)
* **Stateful**:

  * If an **inbound rule** allows traffic, the **return traffic** is automatically allowed, even if there's no explicit outbound rule.
  * Likewise, outbound responses to allowed inbound traffic are permitted.
* **Changes** take effect **immediately** and apply to all instances associated with the group.

---

### 🧱 **Network Access Control Lists (NACLs)**

* NACLs are an **optional**, additional layer of security at the **subnet level**.
* They act as **stateless traffic filters**, evaluating traffic **before it reaches or leaves the subnet**.
* Each **subnet** in a VPC can be associated with **only one NACL**, though a NACL can be shared across multiple subnets.
* Rules are evaluated in **numbered order** (lowest first). First match wins.
* Each rule includes:

  * Rule number (e.g., 100, 110)
  * Protocol (e.g., TCP, UDP, ICMP)
  * Action: **ALLOW** or **DENY**
  * Source/destination IP range (CIDR)
  * Port range (e.g., 80, 443, or 1024-65535)
  * Optional: ICMP types (for ping, etc.)
* NACLs can both **allow and deny** traffic, unlike security groups.
* **Stateless**:

  * You must **explicitly allow** both **inbound and outbound** traffic.
  * If inbound rule allows traffic, outbound rule must also allow response.
* **Changes** to NACLs may take **a short time to propagate**.

---

### 🔄 **Comparison: Security Group vs. NACL**

| Feature                               | Security Group                    | Network ACL                      |
| ------------------------------------- | --------------------------------- | -------------------------------- |
| Scope                                 | Instance-level                    | Subnet-level                     |
| Stateful or Stateless                 | **Stateful**                      | **Stateless**                    |
| Allow/Deny support                    | Allow **only**                    | Allow and Deny                   |
| Rule Evaluation Order                 | All rules evaluated               | Rules evaluated in number order  |
| Default behavior                      | Deny all (except what’s allowed)  | Allow all (except what’s denied) |
| Return traffic automatically allowed? | ✅ Yes                             | ❌ No – must allow explicitly     |
| Applied to                            | ENIs (Elastic Network Interfaces) | Subnet as a whole                |
| Typical Use                           | EC2-level control (app-specific)  | Coarse-grained subnet security   |

---

### ✅ **Best Practices**

* Use **Security Groups** for controlling access to instances with fine-grained rules.
* Use **NACLs** to block/allow **specific IPs or ports** across whole subnets (e.g., blacklisting).
* Keep **NACL rules simple and ordered**, avoid rule conflicts.
* Combine both for **defense-in-depth**: NACL (subnet boundary) + SG (instance level).
* Monitor changes via AWS CloudTrail or VPC Flow Logs for auditing traffic.

* Sure! Here are **5 scenario-based AWS interview questions** focused on **Security Groups and NACLs**, including **detailed answers** to help demonstrate strong real-world understanding:

---

### 🔸 **1. Scenario: Inbound Traffic Not Reaching EC2**

**Q:** You’ve launched an EC2 instance in a private subnet, behind an Application Load Balancer in a public subnet. The ALB is healthy, but users are unable to access the application running on the EC2. What steps would you take to troubleshoot this issue?

**Answer:**

* Verify **EC2’s Security Group** allows inbound traffic from the **ALB’s Security Group** on the app port (e.g., TCP 80 or 443).
* Confirm **ALB’s SG** allows inbound traffic from `0.0.0.0/0` on 80/443.
* Ensure **EC2’s NACL** allows **inbound** traffic from the ALB subnet and **outbound** traffic back.
* Check the **Target Group health checks**.
* Confirm that the **application is running** and listening on the correct port inside the EC2 instance.

---

### 🔸 **2. Scenario: EC2 in Private Subnet Can’t Access Internet**

**Q:** An EC2 instance in a private subnet needs to download packages from the internet but fails. What configuration might be missing or incorrect?

**Answer:**

* Check if **NAT Gateway** is present in a **public subnet** with an Elastic IP.
* Ensure the **private subnet’s route table** routes `0.0.0.0/0` to the NAT Gateway.
* Validate **Security Group** allows outbound HTTP/HTTPS traffic.
* Check **NACLs**:

  * Inbound: allow ephemeral ports from NAT.
  * Outbound: allow TCP 80/443 to NAT subnet or internet.

---

### 🔸 **3. Scenario: Denying Access from a Malicious IP**

**Q:** You detect malicious traffic from a specific IP (e.g., `45.55.1.100`). How would you block it at the network level?

**Answer:**

* **Security Groups** don’t support deny rules.
* Use **NACL** for the subnet and create:

  ```txt
  Rule #100: Deny, Source: 45.55.1.100/32, Port: All, Protocol: All
  ```
* Ensure this rule comes **before** any allow rules (lower number).
* Apply NACL to affected **subnets** (public and/or private).

---

### 🔸 **4. Scenario: Multiple EC2s Need Controlled Access**

**Q:** You have a frontend EC2 (Web) and a backend EC2 (App). You want the Web server to access the App server, but deny access from anywhere else. How do you configure SG and NACL?

**Answer:**

* Web SG: Allow inbound from `0.0.0.0/0` on port 80/443.
* App SG: Allow inbound **only from Web’s SG** on port 8080 (or app port).
* NACLs:

  * Allow traffic between the two subnets.
  * If needed, deny all other inbound sources at subnet level (in NACL).
* Ensure both directions (inbound + outbound) are explicitly allowed in NACLs due to statelessness.

---

### 🔸 **5. Scenario: Security Group Change Not Taking Effect**

**Q:** You removed a rule from a Security Group that allowed traffic on port 22, but you're still able to SSH into the instance. Why might this happen?

**Answer:**

* The EC2 instance might be **associated with multiple Security Groups**, and **another SG** still allows port 22.
* Or, **your IP is whitelisted in another SG**.
* SGs are stateful, so existing connections may **not be terminated** immediately.
* Review all **SGs attached to the instance** and check all inbound rules.

---



![Screenshot 2023-06-29 at 12 14 32 AM](https://github.com/iam-veeramalla/aws-devops-zero-to-hero/assets/43399466/30bbc9e8-6502-438b-8adf-ece8b81edce9)

