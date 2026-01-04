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

### 16. What are "Trusted IP" lists?
**Answer:**
Trusted IP lists contain white-listed IPs (like your corporate VPN) that GuardDuty will stop generating findings for.

### 17. GuardDuty is costing too much due to high VPC Flow Log volume. How do you optimize it?
**Answer:**
The Key: Unlike manual log analysis, you don't pay for the storage of logs used by GuardDuty—you pay for the analysis. To optimize:
1. Identify the specific account or resource driving the cost using the GuardDuty "Usage" tab.
2. Consider if certain high-traffic, low-risk VPCs can be excluded (though usually not recommended).
3. Use S3 Protection or EKS Runtime Monitoring selectively if certain workloads don't require deep inspection.

### 18. Scenario 1: Crypto-mining detected on EC2
**Question:**
GuardDuty reports a High severity finding: `CryptoCurrency:EC2/BitcoinTool.B!DNS`. What would you do?

**Answer:**
1. **Validate the finding:**
    *   Check affected EC2 instance ID.
    *   Review VPC Flow Logs and DNS queries.
2. **Contain:**
    *   Isolate instance using a quarantine security group.
    *   Block outbound internet access.
3. **Investigate:**
    *   Enable Malware Protection scan.
    *   Review running processes and user data.
4. **Eradicate:**
    *   Terminate instance if immutable infrastructure.
    *   Rotate IAM credentials.
5. **Prevent:**
    *   Harden AMIs.
    *   Enable Inspector + GuardDuty auto-response.

### 19. Scenario 2: Compromised IAM credentials
**Question:**
GuardDuty detects `UnauthorizedAccess:IAMUser/TorIPCaller`. What actions do you take?

**Answer:**
1. Immediately disable the IAM access key.
2. Rotate credentials.
3. Review CloudTrail logs for:
    *   Unauthorized resource creation.
    *   Privilege escalation attempts.
4. Apply least privilege IAM policies.
5. Enable MFA.
6. Automate via EventBridge → Lambda.

### 20. Scenario 3: GuardDuty detects suspicious S3 access
**Question:**
Finding: `Policy:S3/BucketPublicAccessGranted`. What does GuardDuty detect and what does it NOT do?

**Answer:**
*   **Detects:** Anomalous or risky S3 access patterns and public exposure events.
*   **Does NOT:** Scan objects for malware.
*   **Remediation:**
    *   Block public access.
    *   Review bucket policy.
    *   Enable S3 Access Analyzer.

### 21. Scenario 4: Malware Protection triggered
**Question:**
How does GuardDuty Malware Protection work during an incident?

**Answer:**
1. GuardDuty detects suspicious behavior.
2. Automatically creates EBS snapshot.
3. Snapshot is scanned offline.
4. Findings identify:
    *   Malware family.
    *   Impacted files.
5. **Note:** No runtime performance impact.

### 22. Scenario 5: GuardDuty finding storm (too many alerts)
**Question:**
GuardDuty is generating too many findings. How do you manage noise?

**Answer:**
1. Use Finding suppression rules.
2. Adjust trusted IP lists.
3. Suppress low-severity findings.
4. Aggregate in Security Hub.
5. Create severity-based response automation.

### 23. Scenario 6: EKS cluster compromised
**Question:**
GuardDuty reports `Execution:Kubernetes/ExecInPod`. What does this mean?

**Answer:**
*   **Meaning:** Someone executed a command inside a running pod.
*   **Possible risks:** Lateral movement, Privilege escalation.
*   **Actions:**
    1. Review Kubernetes audit logs.
    2. Identify pod, namespace, service account.
    3. Rotate secrets.
    4. Apply Pod Security Standards.

### 24. Scenario 7: GuardDuty vs Inspector confusion
**Question:**
An EC2 instance is vulnerable but not exploited yet. Which service detects this?

**Answer:**
*   **Amazon Inspector:** Detects CVEs and misconfigurations.
*   **GuardDuty:** Detects active exploitation.
*   **Best practice:** Enable both.

### 25. Scenario 8: Multi-account security monitoring
**Question:**
How would you deploy GuardDuty across 100 AWS accounts?

**Answer:**
1. Use AWS Organizations.
2. Enable GuardDuty in management account.
3. Configure delegated administrator.
4. Aggregate findings centrally.
5. Automate onboarding for new accounts.

### 26. Scenario 9: GuardDuty cannot block traffic
**Question:**
Why doesn’t GuardDuty stop attacks automatically?

**Answer:**
*   GuardDuty is a detection service.
*   Separation of detection and response avoids false positives.
*   Blocking is handled via:
    *   Security Groups
    *   NACLs
    *   AWS WAF
    *   Automation workflows

### 27. Scenario 10: Data exfiltration detected
**Question:**
GuardDuty reports unusual outbound traffic from EC2. How do you investigate?

**Answer:**
1. Inspect VPC Flow Logs.
2. Identify destination IP.
3. Check DNS logs.
4. Validate data transfer volume.
5. Isolate instance.
6. Review application logs.

### 28. Scenario 11: GuardDuty disabled accidentally
**Question:**
How do you prevent GuardDuty from being disabled?

**Answer:**
1. Use SCP (Service Control Policy).
2. Deny `guardduty:DeleteDetector`.
3. Monitor CloudTrail for disable events.

### 29. Scenario 12: GuardDuty + Security Hub
**Question:**
Why integrate GuardDuty with Security Hub?

**Answer:**
1. Centralized visibility.
2. Cross-service correlation.
3. Compliance mapping (CIS, PCI).
4. Single dashboard for SOC teams.

### 30. Scenario 13: False positive crypto mining alert
**Question:**
How would you handle a legitimate high CPU workload flagged as crypto mining?

**Answer:**
1. Validate application behavior.
2. Check destination domains/IPs.
3. Add trusted IPs.
4. Suppress specific findings.
5. Document exception.

### 31. Scenario 14: Incident response automation
**Question:**
Design an automated response for high-severity GuardDuty findings.

**Answer:**
**Flow:**
GuardDuty → EventBridge → Lambda
→ Isolate EC2
→ Disable IAM key
→ Notify SOC (SNS/Slack)

### 32. Scenario 15: Interview “gold answer”
**Question:**
How do you design a production-grade threat detection strategy using GuardDuty?

**Answer:**
1. Enable GuardDuty across all accounts.
2. Integrate with:
    *   Security Hub
    *   Inspector
    *   AWS WAF
3. Automate response.
4. Periodically review findings.
5. Apply least privilege & defense in depth.
