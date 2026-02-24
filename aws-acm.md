# AWS ACM Interview Questions and Answers

This document contains common interview questions related to AWS Certificate Manager (ACM) and SSL/TLS security.

### 1. What is AWS ACM and why do we use it?
**Answer:**
**AWS Certificate Manager (ACM)** is a managed service that lets you:
*   Provision SSL/TLS certificates.
*   Automatically renew certificates.
*   Deploy certificates to AWS services.

**Why use it?**
We use ACM to enable **HTTPS** (secure communication) for services like:
*   Amazon CloudFront.
*   Elastic Load Balancing (ALB/NLB).
*   Amazon API Gateway.

Instead of manually buying, uploading, and renewing certificates, ACM automates the entire lifecycle.

### 2. What is the difference between HTTP and HTTPS?
**Answer:**
*   **HTTP:** Data is sent in **plain text**. It can be easily intercepted by attackers.
*   **HTTPS:** Data is **encrypted** using SSL/TLS certificates. It protects sensitive information like passwords, tokens, and payment details.

### 3. What are the types of certificates in ACM?
**Answer:**
There are two main types:

1.  **Public Certificates:**
    *   Used for public internet-facing websites.
    *   **Free** in ACM.
    *   Automatically renewed.
    *   Valid only for AWS-integrated services (ALB, CloudFront, etc.).

2.  **Private Certificates:**
    *   Used inside internal networks (private VPCs).
    *   Issued via **AWS Private CA**.
    *   Useful for secure microservices communication.
    *   **Paid** service.

### 4. How does ACM validate domain ownership?
**Answer:**
ACM supports two validation methods:

1.  **DNS Validation (Recommended):**
    *   You add a specific CNAME record to your DNS configuration.
    *   ACM checks the record to verify ownership.
    *   **Benefit:** Supports **automatic renewal** without manual intervention.

2.  **Email Validation:**
    *   ACM sends a verification email to the domain administrator.
    *   Requires manual approval.
    *   **Drawback:** Harder to automate; renewal requires manual action if the record isn't preserved.

### 5. Why must CloudFront certificates be in `us-east-1`?
**Answer:**
CloudFront is a **global service**. Even if your infrastructure (S3, EC2) is in another region (e.g., `ap-south-1`), CloudFront requires the ACM certificate to be created in the **N. Virginia (us-east-1)** region.
*   If you create it in another region, CloudFront will not detect it.
*   *Note:* For ALBs, the certificate must be in the same region as the load balancer.

### 6. Does ACM automatically renew certificates?
**Answer:**
**Yes**, but only if:
1.  The certificate is actively used by an AWS service (e.g., attached to an ALB).
2.  The DNS validation record is still present in your DNS provider.

ACM starts the renewal process **60 days** before expiration. This prevents outages caused by expired certificates.

### 7. What happens if a certificate expires?
**Answer:**
*   **Impact:** HTTPS connections fail immediately. Browsers show security warnings ("Your connection is not private"). Users cannot access the site.
*   **Prevention:** Enable CloudWatch alerts for certificate expiration (e.g., alert if `DaysToExpiry < 30`) and use DNS validation for auto-renewal.

### 8. Can ACM certificates be exported?
**Answer:**
*   **Public ACM Certificates:** ❌ **No.** You cannot export the private key. They can only be used with supported AWS services (ALB, CloudFront, API Gateway).
*   **Private ACM Certificates:** ✅ **Yes.** If created with the export option via AWS Private CA, you can export them for use on EC2 instances or on-premise servers.

### 9. How do you secure communication between microservices?
**Answer:**
To ensure encryption inside the VPC (End-to-End Encryption):
*   Use **Private ACM Certificates**.
*   Enable **TLS** between services.
*   Use a **Service Mesh** (like Istio or AWS App Mesh) to enforce **mTLS** (Mutual TLS).
*   Use internal Load Balancers with HTTPS listeners.

### 10. How does ACM work with a Load Balancer?
**Answer:**
**SSL Termination (Offloading):**
1.  Request a certificate in ACM.
2.  Attach the certificate to the **HTTPS listener (Port 443)** of the ALB/NLB.
3.  The Load Balancer handles the TLS handshake and decryption.
4.  Traffic is forwarded to the backend targets (EC2/Pods) usually over HTTP (or HTTPS if end-to-end encryption is required).
*   **Benefit:** Reduces CPU load on application servers.

### 11. What is a wildcard certificate?
**Answer:**
A wildcard certificate secures a domain and multiple subdomains.
*   **Example:** `*.example.com`
*   **Covers:** `api.example.com`, `app.example.com`, `login.example.com`.
*   **Does NOT Cover:** Nested subdomains like `test.api.example.com` (requires `*.api.example.com`).

### 12. What security best practices should be followed with ACM?
**Answer:**
*   Use **DNS Validation** for auto-renewal.
*   Monitor expiration dates via **CloudWatch**.
*   Enforce **TLS 1.2** or higher on Load Balancers/CloudFront.
*   Enable **HSTS** (HTTP Strict Transport Security) headers.
*   Use strong cipher policies.
*   Rotate imported certificates regularly.

### 13. Scenario: Users suddenly see "SSL certificate invalid" error. What do you check?
**Answer:**
**Troubleshooting Steps:**
1.  **Check Expiration:** Is the certificate expired?
2.  **Check Region:** Is the certificate in `us-east-1` (if using CloudFront)?
3.  **Check Association:** Is the certificate actually attached to the Load Balancer listener?
4.  **Check Domain:** Does the domain name match the certificate's CN (Common Name) or SANs (Subject Alternative Names)?
5.  **Check DNS:** Is the DNS pointing to the correct resource?

### 14. Is ACM free?
**Answer:**
*   **Public Certificates:** **Free** when used with AWS services (ALB, CloudFront, etc.).
*   **Private CA:** **Paid** service (monthly fee + per-certificate fee).

### 15. Can I use an ACM certificate on an EC2 instance directly?
**Answer:**
**No**, not for public certificates.
*   Public ACM certificates cannot be exported, so you cannot install them on an Nginx/Apache server running on EC2.
*   **Workaround:** Use an **ALB** (Application Load Balancer) in front of the EC2 instance to handle the certificate, or use Let's Encrypt (Certbot) for certificates directly on EC2.
