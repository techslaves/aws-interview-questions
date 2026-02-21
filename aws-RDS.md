# AWS RDS Interview Questions and Answers

This document contains common interview questions related to AWS RDS and their answers.

### 1. What is Multi-AZ deployment in RDS?
**Answer:**
Multi-AZ provides high availability by automatically replicating data to a standby instance in another Availability Zone.
*   **Purpose:** Used for disaster recovery and high availability.
*   **Note:** It is **not** used for read scaling (the standby instance is not accessible for reads).

### 2. What are Read Replicas?
**Answer:**
Read replicas are read-only copies of the primary database.
**Use Cases:**
*   Read scaling (offloading read traffic from the primary).
*   Analytics workloads.
*   Reducing load on the primary DB.
*   Cross-region disaster recovery (can be promoted to primary).

### 3. What backup options are available in RDS?
**Answer:**
*   **Automated Backups:** Enables point-in-time recovery within a retention period (1-35 days).
*   **Manual Snapshots:** User-initiated backups that persist even after the DB instance is deleted.

### 4. What is a DB snapshot?
**Answer:**
A DB snapshot is a user-initiated, static backup of the entire DB instance stored in Amazon S3. Unlike automated backups, snapshots are kept until you explicitly delete them.

### 5. How does RDS handle security?
**Answer:**
*   **Network Isolation:** Deployed within a VPC.
*   **Access Control:** Security Groups (firewall rules).
*   **Authentication:** IAM database authentication.
*   **Encryption at Rest:** Using AWS KMS.
*   **Encryption in Transit:** Using SSL/TLS.

### 6. What is point-in-time recovery?
**Answer:**
It allows you to restore a database to any specific second within the backup retention period (up to 35 days). It uses automated backups and transaction logs.

### 7. How do you scale RDS?
**Answer:**
*   **Vertical Scaling:** Changing the instance class (e.g., `t3.micro` to `m5.large`) for more CPU/RAM.
*   **Horizontal Scaling:** Adding Read Replicas for read-heavy workloads.
*   **Storage Scaling:** Increasing the allocated storage size (with zero downtime).

### 8. Can RDS be stopped?
**Answer:**
*   **Yes**, but typically for non-production environments to save costs.
*   **Limit:** Maximum stop duration is 7 days (it auto-starts after that).
*   **Restriction:** Not supported for Multi-AZ deployments or Read Replicas in some older configurations (though newer versions support stopping Multi-AZ).

### 9. How would you design a highly available database architecture?
**Answer:**
*   Enable **Multi-AZ** deployment for automatic failover.
*   Configure **Automated Backups** with a sufficient retention period.
*   Use **Read Replicas** for scaling and regional DR.
*   Enable **CloudWatch Monitoring** and alerts.

### 10. How do you migrate an on-prem DB to RDS?
**Answer:**
*   **AWS Database Migration Service (DMS):** For continuous replication and minimal downtime.
*   **AWS Schema Conversion Tool (SCT):** To convert schema if engines differ (e.g., Oracle to Aurora).
*   **Native Tools:** `mysqldump`/`pg_dump` and restore (for offline migration).

### 11. What happens during a Multi-AZ failover?
**Answer:**
1.  The primary instance fails or becomes unreachable.
2.  RDS automatically flips the CNAME record (DNS endpoint) to point to the standby instance.
3.  The standby instance becomes the new primary.
4.  **Impact:** Applications reconnect automatically without manual intervention (usually within 60-120 seconds).

### 12. How to implement Disaster Recovery (DR) in RDS?
**Answer:**
To implement DR in RDS, I use Multi-AZ for high availability within a region and cross-region read replicas for regional DR. Automated backups and snapshots handle data corruption scenarios.

**DR Strategies in Amazon RDS:**

**1️⃣ Multi-AZ Deployment (Primary DR Method)**
*   **What it is:** Synchronous replication to a standby instance in another AZ.
*   **How it helps:** Protects against AZ failure.
*   **RTO:** Minutes (Automatic failover).
*   **RPO:** Zero or near-zero (Synchronous).
*   **Use case:** Mission-critical production databases.

**2️⃣ Read Replicas (Cross-Region DR)**
*   **What it is:** Asynchronous replication to another region.
*   **How it helps:** Protects against regional failures. Can be promoted to primary DB.
*   **RTO:** Minutes (Manual promotion).
*   **RPO:** Seconds to minutes (Replication lag).

**3️⃣ Automated Backups & Point-in-Time Recovery**
*   **What it is:** Continuous backups stored in S3.
*   **How it helps:** Protects against data corruption and accidental deletes.
*   **RTO:** Hours (Restoration time depends on DB size).
*   **RPO:** Minutes (typically 5 minutes).

**4️⃣ Manual Snapshots (Long-Term & Compliance DR)**
*   **What it is:** User-initiated full backups.
*   **How it helps:** Cross-account and cross-region recovery, long-term retention.
*   **RTO:** Hours.
*   **RPO:** Depends on snapshot frequency.

### 13. When we promote a cross-regional RDS replica as the primary replica, how will a pod running in EKS in another region communicate to that?
**Answer:**
Pods in EKS should **NOT** connect using a hard-coded RDS endpoint. They should connect via a DNS abstraction (Route 53 or similar) that is updated during promotion.

**Failover Workflow:**
1.  Disaster occurs in Region A.
2.  Promote read replica in Region B.
3.  Update Route 53 record (automatic or manual).
4.  DNS propagates.
5.  Pods in EKS Region B reconnect to the new primary.

**Important:** We use a **Private Hosted Zone** in Route 53 instead of Public Hosted Zones.

**Why do we use Private Hosted Zones?**
*   **Private Endpoint:** RDS is in a VPC, and its endpoints are not publicly routable.
*   **Internal Resolution:** A Private Hosted Zone (PHZ) resolves DNS inside the VPC only.
*   **Security:** The database hostname is not exposed to the internet, preventing accidental public access.
*   **Best Practice:** Aligns with least-privilege & zero-trust principles.

### 14. What is the impact of DNS TTL on failover?
**Answer:**
DNS TTL (Time To Live) has a direct and critical impact on failover speed when you use Route 53 for RDS DR with EKS.
*   **Definition:** TTL defines how long DNS resolvers cache a record before querying Route 53 again.
*   **Impact:**
    *   **High TTL (e.g., 300s):** Slower failover. Pods will keep trying to connect to the old (failed) IP until the cache expires.
    *   **Low TTL (e.g., 60s):** Faster failover, but higher query volume to Route 53.
*   **Recommendation:** Use a low TTL (e.g., 60 seconds) for DR-related DNS records to minimize downtime during a switchover.
