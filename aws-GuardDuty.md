# AWS GuardDuty Interview Questions and Answers

This document contains common interview questions related to AWS GuardDuty and their answers.

### 1. What is Amazon GuardDuty?
**Answer:**
Amazon GuardDuty is a threat detection service that continuously monitors AWS accounts, workloads, and data using machine learning, anomaly detection, and threat intelligence.

### 2. GuardDuty flags an EC2 instance for "UnauthorizedAccess:EC2/MaliciousIPCaller." Walk me through an automated remediation workflow.
**Answer:**
**Trigger**: GuardDuty generates a finding.
**Event:** An EventBridge rule matches that finding type.
**Action:** Trigger an AWS Lambda function.
**Remediation:** The Lambda function could:
1. Modify the EC2 Security Group to block the malicious IP.
2. Isolate the instance by attaching a "Quarantine" SG.
3. Take a snapshot of the EBS volume for forensic analysis via Amazon Detective.

### 3. How does GuardDuty process data without affecting the performance of your EC2 instances?
**Answer:**
GuardDuty is a managed, non-invasive service. It pulls data directly from AWS infrastructure logs (VPC Flow Logs, DNS logs, and CloudTrail) rather than using an agent. Because it consumes these logs independently of your compute resources, it has zero impact on CPU/Memory or network latency. GuardDuty is reactive/behavioral.

### 4. What is GuardDuty Malware Protection?
**Answer:**
Amazon GuardDuty Malware Protection is a managed threat detection capability that automatically detects malware on Amazon EC2 instances and container workloads without requiring agents to be installed.

### 5. What triggers Malware Protection scans?
**Answer:**
1. Suspicious network activity
2. Crypto mining behavior
3. Credential compromise signals
4. Backdoor or rootkit indicators
5. Unexpected outbound connections

### 6. GuardDuty flags an EBS volumes for suspicious behavior (e.g., crypto mining, C2 traffic, credential abuse), Walk me through an automated remediation workflow.
**Answer:**
1. Creates a snapshot of the EBS volume
2. Scans the snapshot using malware detection engines
3. Generates GuardDuty findings if malware is detected

No performance impact on running workloads because snapshots are scanned, not live volumes.

### 7. Follow up question: How will Pricing work for workflow you explained (Important for interviews)
**Answer:**
*   It is charged per GB of EBS volume scanned.
*   No upfront cost.
*   Snapshot storage costs may apply temporarily.

### 8. Is GuardDuty agent-based?
**Answer:**
No. GuardDuty is agentless and requires no software installation.

### 9. What is a GuardDuty Finding?
**Answer:**
A finding is a structured security alert describing:
1. What happened
2. Which resource is affected
3. Severity (Low / Medium / High)
4. Recommended remediation

### 10. Can GuardDuty block attacks automatically?
**Answer:**
No. GuardDuty is a detection service, not prevention.
However, you can automate response using:
*   EventBridge
*   Lambda
*   Security Hub
*   Step Functions

### 11. How do you respond automatically to GuardDuty findings?
**Answer:**
1. Send findings to Amazon EventBridge
2. Trigger Lambda functions
3. Actions may include:
    *   Isolate EC2 (security group update)
    *   Disable IAM user
    *   Stop compromised instance

### 12. What is GuardDuty for EKS?
**Answer:**
GuardDuty monitors EKS audit logs.
**Detects:**
*   Privilege escalation
*   Suspicious pod execution
*   Malicious container behavior

### 13. What is the severity classification in GuardDuty?
**Answer:**
*   **Low** – Recon or benign anomaly
*   **Medium** – Suspicious activity
*   **High** – Confirmed malicious behavior

### 14. Does GuardDuty store logs?
**Answer:**
No, GuardDuty analyzes logs but does not store them.
Logs remain in:
*   CloudTrail
*   VPC Flow Logs
*   DNS logs

### 15. Can GuardDuty scan S3 for malware?
**Answer:**
No, GuardDuty:
*   Detects suspicious S3 access
*   Does not scan file contents for malware
(Use Amazon Macie or third-party tools for that.)
