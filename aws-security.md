# AWS Security Interview Questions and Answers

This document contains common interview questions related to AWS Security, focusing on CORS, Brute Force protection, and infrastructure hardening.

### 1. What is CORS?
**Answer:**
**CORS (Cross-Origin Resource Sharing)** controls how browsers allow requests between different domains. It is a browser security feature that restricts cross-origin HTTP requests that are initiated from scripts running in the browser.

**Risks of Misconfiguration:**
*   Any domain can access your API.
*   Sensitive data can be exposed.
*   Tokens can be abused.

**Example Risky Configuration:**
```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```
This combination allows credential sharing across all origins, which is a major security vulnerability.

### 2. How to prevent/configure CORS in AWS?
**Answer:**
The strategy depends on which service is acting as your "Origin."

**1. Amazon S3 (Static Content)**
If your frontend is trying to fetch a file or upload directly to an S3 bucket from a different domain, you must add a CORS configuration to the bucket.
*   **Steps:** Go to S3 Console > Permissions > Cross-origin resource sharing (CORS) > Edit.
*   **Policy Example:**
    ```json
    [
      {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
        "AllowedOrigins": ["https://your-app-domain.com"],
        "ExposeHeaders": []
      }
    ]
    ```
*   **Security Tip:** Avoid using `"*"` in `AllowedOrigins` for production. Specify your exact frontend URL.

**2. API Gateway (REST or HTTP APIs)**
API Gateway is the most common place where CORS errors occur, especially with Lambda integrations.
*   **For REST APIs:**
    1.  Select Resource > Actions > **Enable CORS**.
    2.  Specify allowed origins and methods. This creates an `OPTIONS` method for the "preflight" request.
    3.  **Crucial Step:** If using **Lambda Proxy Integration**, your Lambda function code must also return the headers:
        ```javascript
        exports.handler = async (event) => {
            return {
                statusCode: 200,
                headers: {
                    "Access-Control-Allow-Origin": "https://your-app-domain.com",
                    "Access-Control-Allow-Methods": "OPTIONS,POST,GET"
                },
                body: JSON.stringify({ message: "Hello from Lambda!" }),
            };
        };
        ```
*   **For HTTP APIs:**
    *   Go to Develop > CORS > Configure. Add domains to the "Allow origins" list. HTTP APIs handle preflight logic automatically.

**3. CloudFront (CDN)**
If CloudFront sits in front of S3 or an API, it might strip CORS headers or cache the wrong ones.
*   **Response Headers Policy:** Attach a "CORS Response Header Policy" to your CloudFront Behavior. This ensures CloudFront adds required headers even if the origin doesn't.
*   **Forward Headers:** Ensure the `Origin` header is included in the **Cache Key Settings**. If CloudFront doesn't "see" the origin header, it won't pass it to the backend, and CORS will fail.

### 3. What is a Brute Force Attack and how do you prevent it in AWS?
**Answer:**
A Brute Force Attack is a trial-and-error method used to guess login credentials, encryption keys, or hidden URLs. Attackers use automated software to "force" their way into a system by submitting thousands of combinations.

**Prevention Strategies in AWS:**

**1. Edge Protection (The Front Line)**
The goal is to stop the attack at the "edge" before it ever reaches your application servers or database.
*   **AWS WAF (Web Application Firewall):**
    *   **Rate-Based Rules:** Automatically block IP addresses that exceed a specific request threshold (e.g., >100 requests to `/login` in 5 minutes).
    *   **ATP (Account Takeover Prevention):** Detects "credential stuffing" by checking if login attempts match leaked credentials from other data breaches.
*   **AWS Shield:** Provides protection against volumetric DDoS attacks that often accompany large-scale brute force attempts.

**2. Infrastructure Hardening**
Protect management ports (SSH for Linux, RDP for Windows).
*   **AWS Systems Manager (SSM) Session Manager:** The "gold standard" for security. Allows managing instances without opening Port 22 or 3389. If the port is closed, it cannot be brute-forced.
*   **Security Groups:** Use "Least Privilege." Never open SSH (22) to `0.0.0.0/0`. Only allow specific, known IP addresses (like your office VPN).
*   **EC2 Instance Connect:** Provides temporary SSH keys, reducing the window of opportunity for an attacker.

**3. Identity & Access Management (IAM)**
*   **Multi-Factor Authentication (MFA):** The single most effective defense. Even if an attacker guesses a password, they cannot log in without the second factor.
*   **Amazon Cognito Threat Protection:** Enable "Advanced Security" to detect unusual sign-in attempts and automatically prompt for MFA or block the user.
*   **Account Lockout Policies:** Ensure your application locks an account after a certain number of failed attempts (e.g., 5 failures = 15-minute lockout).

**4. Detection & Automated Response**
*   **Amazon GuardDuty:** Uses ML to monitor VPC Flow Logs and CloudTrail. Triggers findings like `UnauthorizedAccess:EC2/SSHBruteForce`.
*   **Auto-Remediation:**
    *   GuardDuty detects attack → EventBridge triggers Lambda → Lambda adds attacker's IP to a WAF blocklist or Network ACL.

### 4. What is Cross-Site Scripting (XSS)?
**Answer:**
Cross-Site Scripting (XSS) is a high-severity security vulnerability where an attacker "injects" malicious scripts (usually JavaScript) into a trusted website. The browser, thinking the script came from a legitimate source, executes it, allowing the attacker to steal data or hijack user sessions.

**Types:**
*   **Stored XSS:** Malicious script is permanently stored on the target server (e.g., in a database).
*   **Reflected XSS:** Malicious script is reflected off the web server (e.g., in an error message or search result).
*   **DOM-based XSS:** Vulnerability exists in client-side code rather than server-side code.

**Impact:**
*   Session hijacking.
*   Token theft.
*   Credential compromise.

**How to Prevent XSS in AWS & DevSecOps:**
*   **AWS WAF:** Enable the **Core Rule Set (CRS)** managed rule group. It includes specific patterns to detect and block common XSS injection attempts in headers, query strings, and body parameters.

### 5. What is SSRF (Server-Side Request Forgery) and how to prevent it in AWS?
**Answer:**
SSRF is a vulnerability where an attacker forces a server-side application to make HTTP requests to an arbitrary domain of the attacker's choosing. In AWS, attackers often target the **Instance Metadata Service (IMDS)** to steal IAM credentials.

**How SSRF Works in AWS:**
1.  **Vulnerability:** App fetches data from a user-supplied URL.
2.  **Attack:** Attacker inputs `http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name`.
3.  **Theft:** Server fetches the URL and returns the IAM credentials (AccessKeyId, SecretAccessKey, Token).
4.  **Result:** Attacker gains the permissions of the EC2 instance.

**Prevention Strategies:**

**1. Enable IMDSv2 (Critical):**
*   **Why:** IMDSv2 requires a session token (PUT request) before accessing metadata (GET request). Most SSRF attacks can only perform simple GET requests.
*   **Config:** Set "Metadata response hop limit" to 1 to prevent token forwarding.

**2. AWS WAF:**
*   **Managed Rules:** Enable `AnonymousIPList` and `CoreRuleSet`.
*   **Custom Rules:** Block any string containing `169.254.169.254`.

**3. Network-Level Defenses (VPC):**
*   **Egress Filtering:** Use NACLs or Security Groups to restrict outbound traffic.
*   **Web Proxy:** Force outbound traffic through a proxy with an allow-list of approved domains.

**4. Application-Layer Validation:**
*   **Allow-listing:** Only allow specific domains (e.g., `*.trusted-bucket.s3.amazonaws.com`).
*   **Block IPs:** Deny input that is a raw IP address; resolve DNS and verify it's not a private range.
