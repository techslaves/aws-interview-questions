# AWS GuardDuty Interview Questions and Answers

This document contains common interview questions related to Kubernetes Security and their answers.

### 1. GuardDuty flags an EC2 instance for "UnauthorizedAccess:EC2/MaliciousIPCaller." Walk me through an automated remediation workflow.
**Answer:**
**Trigger**: GuardDuty generates a finding.

**Event:** An EventBridge rule matches that finding type.

**Action:** Trigger an AWS Lambda function.

**Remediation:** The Lambda function could:

1. Modify the EC2 Security Group to block the malicious IP.

2. Isolate the instance by attaching a "Quarantine" SG.

3. Take a snapshot of the EBS volume for forensic analysis via Amazon Detective.
