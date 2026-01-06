# AWS WAF Interview Questions and Answers

This document contains common interview questions related to AWS WAF and their answers.

### 1. What is Amazon WAF?
**Answer:**
AWS WAF (Web Application Firewall) is a web application firewall that helps protect your web applications or APIs against common web exploits and bots that may affect availability, compromise security, or consume excessive resources.

**Key Features:**
*   **Integrates with:**
    *   Amazon CloudFront
    *   Application Load Balancer (ALB)
    *   API Gateway (REST & HTTP APIs)
    *   AWS AppSync
*   **Layer:** WAF is deployed at Layer 7 (HTTP/HTTPS).
*   **Scope:** It can be deployed as Regional WAF (for ALBs, API Gateway) or Global WAF (for CloudFront).

### 2. Difference between AWS WAF (Regional) and CloudFront WAF?
**Answer:**

| Aspect | Regional WAF | CloudFront WAF |
| :--- | :--- | :--- |
| **Scope** | Single region | Global |
| **Latency** | Higher | Lower (edge locations) |
| **Use cases** | ALB, API Gateway | CDN, global apps |

### 3. How would you design WAF protection for a global application?
**Answer:**
*   **CloudFront + AWS WAF:** Deploy WAF at the edge.
*   **AWS Managed Rule Groups:** Use pre-configured rules for common threats.
*   **Geo-restriction:** Block traffic from countries where you don't do business.
*   **Rate-based rules:** Protect against DDoS and brute-force attacks.
*   **Shield Advanced integration:** For enhanced DDoS protection.
*   **Central logging:** Kinesis Firehose → S3 → Athena for analysis.

### 4. What are rule groups and why are they used?
**Answer:**
A rule group is a logical grouping of rules.
**Types:**
*   AWS Managed Rule Groups
*   Marketplace Rule Groups
*   Custom Rule Groups

**Benefits:**
*   Reusability across multiple Web ACLs.
*   Versioning support.
*   Easier management and organization.

### 5. Explain priority and rule evaluation order in AWS WAF.
**Answer:**
*   **Priority Number:** Lower priority number = evaluated first.
*   **Evaluation:** The first matching rule decides the action (ALLOW, BLOCK, COUNT).
*   **Default Action:** If no rule matches the request, the default action of the Web ACL applies.

### 6. What is COUNT mode and when do you use it?
**Answer:**
In COUNT mode, the rule evaluates traffic and logs matches but does not block the request.
**Used for:**
*   Testing new rules before enforcement.
*   Monitoring for false positives.
*   Gradual rollout of security policies.

### 7. How does AWS WAF protect against OWASP Top 10?
**Answer:**
AWS provides Managed Rule Groups that specifically target OWASP Top 10 vulnerabilities, including:
*   SQL Injection (SQLi)
*   Cross-Site Scripting (XSS)
*   Command Injection
*   Local/Remote File Inclusion (LFI/RFI)

It uses pattern matching, regular expressions (regex), and anomaly scoring to detect these attacks.

### 8. Explain rate-based rules with a real example.
**Answer:**
Rate-based rules track the number of requests from a single IP address over a 5-minute sliding window.
**Example:**
*   **Rule:** Block IP if it sends > 2000 requests in 5 minutes.
**Useful for:**
*   DDoS mitigation.
*   Preventing brute-force attacks.
*   Stopping API abuse.

### 9. How do you enable and analyze AWS WAF logs?
**Answer:**
1.  **Enable WAF logging** in the Web ACL configuration.
2.  **Send logs to:**
    *   Amazon Kinesis Data Firehose
    *   Amazon S3
    *   Amazon OpenSearch Service
3.  **Analyze using:**
    *   Amazon Athena (SQL queries on S3 logs)
    *   Amazon CloudWatch metrics and dashboards.

### 10. What metrics are available in CloudWatch for AWS WAF?
**Answer:**
*   `AllowedRequests`
*   `BlockedRequests`
*   `CountedRequests`
*   `RateBasedRuleTriggered`
*   `RuleMatches`

### 11. How would you debug a legitimate request being blocked?
**Answer:**
1.  Check WAF logs.
2.  Identify the rule ID causing the block.
3.  Switch the rule to **COUNT** mode.
4.  Add an exception or scope-down statement.
5.  Test and re-enable **BLOCK** mode.

### 12. Difference between AWS WAF and AWS Shield?
**Answer:**

| Feature | AWS WAF | AWS Shield |
| :--- | :--- | :--- |
| **Layer** | Layer 7 (Application) | Layer 3/4 (Network/Transport) |
| **Protection** | Web attacks (SQLi, XSS) | DDoS attacks |
| **Custom rules** | Yes | No |

### 13. How does AWS WAF integrate with Shield Advanced?
**Answer:**
*   **Automatic mitigation:** Shield can automatically create WAF rules.
*   **Cost protection:** Shield Advanced covers WAF costs for DDoS mitigation.
*   **DDoS Response Team (DRT):** Access to experts who can write custom WAF rules during an attack.
*   **Enhanced visibility:** Real-time metrics and reports.

### 14. How do you block traffic from specific countries?
**Answer:**
*   Use a **GeoMatch statement** in your rule.
*   Combine with an allowlist for trusted regions if necessary.

### 15. Can AWS WAF replace a traditional firewall?
**Answer:**
**No.** AWS WAF operates only at Layer 7 (Application Layer).
It must be combined with:
*   **Security Groups** (Instance level firewall)
*   **NACLs** (Subnet level firewall)
*   **AWS Network Firewall** (VPC level inspection)

### 16. How is AWS WAF priced?
**Answer:**
Pricing is based on:
1.  **Web ACL:** Monthly fee per Web ACL.
2.  **Rule count:** Monthly fee per rule.
3.  **Request volume:** Per million requests processed.
4.  **Managed rule group fees:** Additional charges for premium managed rules.

### 17. How do you optimize AWS WAF cost?
**Answer:**
*   Minimize rule count.
*   Use managed rules.
*   Use scope-down statements.
*   Use **COUNT** mode before **BLOCK** mode to avoid unnecessary blocking.

### 18. How would you protect a login API from brute-force attacks?
**Answer:**
1.  **Rate-based rule:**
    *   Threshold: 100 requests / 5 minutes per IP.
    *   Scope-down: URI path `/login`, HTTP method `POST`.
2.  **Add Bot Control Managed Rules.**
3.  **Enable CAPTCHA** after threshold breach.
4.  **Monitor CloudWatch metrics.**

### 19. Your app receives abnormal traffic from one country. How do you mitigate?
**Answer:**
1.  Analyze WAF logs to confirm attack pattern.
2.  Create a **GeoMatch rule**.
3.  Combine with:
    *   Allow list for trusted IPs.
    *   Rate-based rule.
4.  Apply rule to CloudFront WAF.
5.  Monitor impact.

### 20. How do you stop bots scraping your website?
**Answer:**
1.  Enable **AWS Bot Control Managed Rule Group**.
2.  Add:
    *   Challenge
    *   CAPTCHA
3.  Inspect `User-Agent` headers.
4.  Rate-limit suspicious patterns.
5.  Use labels for behavior tracking.
