# AWS High Availability Interview Questions

This document contains common interview questions related to designing highly available architectures on AWS, focusing on MSK, RDS/Aurora, and EKS.

## 🔹 Amazon MSK (Managed Streaming for Apache Kafka)

### 1. How do you design a highly available MSK cluster?
**Answer:**
A highly available MSK cluster should:
*   Be deployed across multiple **Availability Zones** (minimum 3 AZs).
*   Use **replication factor ≥ 3**.
*   Distribute brokers evenly across AZs.
*   Enable automatic broker recovery.
*   Use **rack awareness** (broker AZ awareness).
*   *Note:* MSK automatically spreads brokers across AZs when configured properly.

### 2. What happens if a broker fails in MSK?
**Answer:**
1.  Kafka elects a new partition leader from the **ISR (In-Sync Replicas)**.
2.  Producers and consumers automatically reconnect to the new leader.
3.  MSK replaces the failed broker automatically (if auto-healing is enabled).
*   **Availability depends on:** Replication factor and `min.insync.replicas` setting.

### 3. How do you prevent data loss in MSK?
**Answer:**
*   Set **replication factor = 3**.
*   Configure producer `acks=all`.
*   Set `min.insync.replicas=2`.
*   Enable `unclean.leader.election=false`.
*   Monitor **ISR shrinkage** via CloudWatch.

### 4. How do you achieve cross-region HA for MSK?
**Answer:**
Use:
*   **MirrorMaker 2** or **MSK Replicator**.
*   Multi-region active-passive architecture.
*   **Example:** Primary MSK in `us-east-1`, Secondary MSK in `us-west-2`, replicating topics using MirrorMaker 2.

### 5. How do you scale MSK without downtime?
**Answer:**
*   **Horizontal Scaling:** Add more brokers.
*   **Vertical Scaling:** Increase broker instance size.
*   **Throughput:** Increase partitions for topics.
*   *Note:* MSK supports rolling updates without downtime.

### 6. What is Multi-AZ in RDS?
**Answer:**
In Amazon RDS, Multi-AZ:
*   **Synchronously replicates** data to a standby instance in another AZ.
*   **Automatically fails over** during instance failure, AZ outage, or maintenance.
*   **Failover time:** Typically 60–120 seconds.

### 7. Difference between Multi-AZ and Read Replicas?
**Answer:**

| Feature | Multi-AZ | Read Replica |
| :--- | :--- | :--- |
| **Focus** | High Availability (HA) | Scaling (Read Performance) |
| **Replication** | Synchronous | Asynchronous |
| **Failover** | Automatic | Manual promotion required |
| **Region** | Same Region | Same or Cross-Region |

### 8. How does Aurora provide higher availability than RDS?
**Answer:**
Amazon Aurora features:
*   **6-way replication** across 3 AZs.
*   **Self-healing storage** system.
*   **Fast failover** (10–30 seconds).
*   **Shared storage architecture** (storage layer handles replication, not the DB engine).

### 9. How do you design cross-region DR for Aurora?
**Answer:**
*   **Aurora Global Database:** Replicates data with < 1 second latency.
*   **Cross-region Read Replica:** Manual promotion.
*   **Automated Backups:** Copy snapshots to S3 in another region.
*   **Best Pattern:** Aurora Global DB with a secondary region for < 1 minute RPO.

### 10. What causes RDS failover?
**Answer:**
*   AZ outage.
*   Instance crash/failure.
*   Storage failure.
*   Manual reboot with failover option.
*   OS patching (during maintenance window).

### 11. How do you design HA for EKS control plane?
**Answer:**
In Amazon EKS, HA is managed by AWS:
*   Control plane is automatically deployed across **3 AZs**.
*   Backed by multi-AZ **etcd**.
*   No manual HA configuration is required for the control plane.

### 12. How do you design HA worker nodes?
**Answer:**
*   Deploy nodes across **multiple AZs**.
*   Use **Auto Scaling Groups (ASG)**.
*   Use multiple node groups.
*   Implement **PodDisruptionBudgets (PDB)**.
*   Enable **Cluster Autoscaler**.

### 13. What happens if a node fails in EKS?
**Answer:**
1.  Kubernetes detects the node as `NotReady`.
2.  Pods are rescheduled onto healthy nodes.
3.  If using ASG, the failed instance is terminated and replaced automatically.

### 14. How do you achieve zero-downtime deployments in EKS?
**Answer:**
*   Use **Rolling Updates**.
*   Configure `maxUnavailable=0` and `maxSurge=1`.
*   Use **Readiness & Liveness Probes**.
*   Configure **PodDisruptionBudgets**.

### 15. How do you make EKS resilient to AZ failure?
**Answer:**
*   Distribute nodes in **≥ 3 AZs**.
*   Use **ALB** (Application Load Balancer) across AZs.
*   Use **Multi-AZ RDS** and **Multi-AZ MSK**.
*   **Avoid** single-AZ EBS volumes (use EFS or ensure app handles volume re-creation).

### 16. Design a highly available streaming platform with MSK + EKS + RDS.
**Sample Answer:**
*   **MSK:** 3 AZ deployment, Replication Factor 3.
*   **EKS:** Multi-AZ node groups, HPA + Cluster Autoscaler.
*   **RDS:** Multi-AZ enabled.
*   **ALB:** Multi-AZ load balancing.
*   **DNS:** Route 53 health checks for failover.

### 17. How do you monitor HA health in AWS?
**Answer:**
*   **CloudWatch Metrics:** CPU, Memory, Disk I/O.
*   **MSK:** ISR (In-Sync Replicas) metrics.
*   **RDS:** Replica lag.
*   **EKS:** Node health status.
*   **ALB:** Unhealthy host count.
*   **AWS Health Dashboard:** For service-wide issues.

### 18. What is RPO and RTO in this architecture?
**Answer:**
*   **RPO (Recovery Point Objective):** Data loss tolerance.
    *   *Example:* Aurora Global DB → RPO < 1 sec.
*   **RTO (Recovery Time Objective):** Recovery time tolerance.
    *   *Example:* Multi-AZ RDS → RTO ~1–2 min.

### 19. Kafka partition leader is in a failed AZ. What happens?
**Answer:**
1.  Kafka elects a new leader from the **ISR**.
2.  Producers reconnect to the new leader.
3.  **No data loss** occurs if `replication factor ≥ 3` and `min.insync.replicas=2`.

### 20. Entire AZ goes down. What survives?
**Answer:**
If properly architected:
*   **MSK:** Survives (brokers in other AZs take over).
*   **RDS Multi-AZ:** Fails over to standby in another AZ.
*   **Aurora:** Survives (6 copies across 3 AZs).
*   **EKS:** Pods are rescheduled to nodes in healthy AZs.
*   **ALB:** Routes traffic only to healthy targets in remaining AZs.

### 21. How do you avoid split-brain in Kafka?
**Answer:**
*   Use an **odd number** of brokers (e.g., 3, 5).
*   Disable `unclean.leader.election`.
*   Ensure proper ISR configuration.

### 22. How do you upgrade MSK/EKS without downtime?
**Answer:**
*   **MSK:** Rolling broker upgrade (one at a time).
*   **EKS:** Blue/Green or Rolling update of node groups. Drain nodes before termination. Upgrade one AZ at a time.

### 23. Design a multi-region active-active architecture.
**Answer Outline:**
*   **MSK:** Deployed in 2 regions with **MirrorMaker** bidirectional replication.
*   **Database:** **Aurora Global Database** (Write in primary, Read in secondary, or DynamoDB Global Tables for active-active).
*   **Compute:** **EKS** clusters running in both regions.
*   **Traffic:** **Route 53** latency-based routing or **AWS Global Accelerator**.
