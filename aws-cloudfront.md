# AWS CloudFront Interview Questions and Answers

This document contains common interview questions related to AWS CloudFront and its security, performance, and architecture.

### 1. What is CloudFront and why use it?
**Answer:**
CloudFront is a global Content Delivery Network (CDN) that:
*   Caches content at edge locations.
*   Reduces latency.
*   Offloads origin traffic.
*   Improves availability.
*   Adds edge-layer security.

**Supports:**
*   Static content (S3).
*   Dynamic APIs.
*   Streaming.
*   Custom origins (ALB, EC2).

**Architect Insight:** CDN is not just for speed — it is also a security boundary.

### 2. How does CloudFront reduce latency?
**Answer:**
*   Uses global edge locations.
*   Routes via AWS backbone network.
*   TCP optimization.
*   Caching.
*   **Origin Shield** (centralized cache layer).
*   **Bonus:** Supports HTTP/2 and HTTP/3 for performance improvement.

### 3. What metrics do you monitor for CloudFront?
**Answer:**
**Availability:**
*   4xx error rate.
*   5xx error rate.
*   Origin connection errors.

**Performance:**
*   Total requests.
*   Cache hit ratio.
*   Origin latency.

**Security:**
*   Blocked requests (via WAF).
*   Geo restriction triggers.

**Architect Insight:** Low cache hit ratio increases cost and origin pressure.

### 4. How do you design a highly available CDN architecture?
**Answer:**
*   Multiple origins (failover origin groups).
*   Multi-AZ backend.
*   Health checks.
*   TTL tuning.
*   Origin Shield enabled.
*   WAF at edge.

### 5. How do you secure CloudFront from DDoS attacks?
**Answer:**
**Multi-layered defense:**
1.  **Edge protection:**
    *   AWS Shield Standard (automatic).
    *   Shield Advanced (for L7 attacks).
2.  **Application protection:**
    *   AWS WAF rules (rate limiting, bot control).
3.  **Network hardening:**
    *   Restrict origin access (OAC / OAI).
    *   Allow only CloudFront IPs to backend.

**Architect Insight:** Security should block traffic at the edge before it hits the origin.

### 6. What is Origin Access Control (OAC) and why use it?
**Answer:**
OAC allows CloudFront to securely access S3 origins without exposing them publicly.
*   **Mechanism:** CloudFront signs requests, and the S3 bucket policy restricts direct access.
*   **Prevents:** Bypass attacks and direct S3 URL abuse.

### 7. How do you prevent users from bypassing CloudFront and hitting the origin directly?
**Answer:**
*   **For S3:** Use OAC (Origin Access Control).
*   **For ALB/EC2:** Restrict security groups to CloudFront IP ranges.
*   **Validation:** Use custom headers validation.
*   **Network:** Use private subnets.

**Architect Insight:** Origin should never be publicly accessible.

### 8. How do you secure APIs behind CloudFront?
**Answer:**
*   TLS enforcement (ACM certificates).
*   WAF rate limiting.
*   Geo-blocking.
*   Signed URLs or signed cookies.
*   JWT validation at edge (Lambda@Edge).
*   API throttling.

### 9. What are Signed URLs and Signed Cookies?
**Answer:**
Used for restricting access to premium content or temporary access control.
*   **Signed URLs:** Single file access, expiration-based.
*   **Signed Cookies:** Access to multiple files.
*   **Common in:** Video streaming, SaaS dashboards.

### 10. How do you implement geo-restrictions?
**Answer:**
*   **CloudFront Geographic Restrictions:** Whitelist/Blacklist countries.
*   **WAF:** Country-based rules.
*   **Use Cases:** Licensing restrictions, fraud prevention.

### 11. A customer reports a sudden spike in 5xx errors. What do you check?
**Answer:**
1.  **Identify Source:** Check if 5xx is from CloudFront or Origin.
2.  **Check Origin:** Origin health, ALB target health, Backend CPU/memory.
3.  **Check WAF:** Is it blocking legitimate traffic?
4.  **Check Cache:** Low hit ratio increases origin load.

**Architect Thinking:** Many 5xx errors are backend overload, not CDN issues.

### 12. How do you protect against bot scraping?
**Answer:**
*   AWS WAF Bot Control.
*   Rate limiting.
*   CAPTCHA.
*   User-agent filtering.
*   Behavioral analysis.
*   Signed URLs.

### 13. How do you encrypt traffic end-to-end?
**Answer:**
*   **Viewer → CloudFront:** HTTPS.
*   **CloudFront → Origin:** HTTPS.
*   **Certificates:** ACM certificates.
*   **Policy:** TLS 1.2+ enforcement, HSTS headers.
*   **Bonus:** Perfect forward secrecy.

### 14. What security headers would you add at the CDN layer?
**Answer:**
Using Lambda@Edge or CloudFront functions:
*   `Strict-Transport-Security`
*   `Content-Security-Policy`
*   `X-Frame-Options`
*   `X-Content-Type-Options`
*   `Referrer-Policy`

### 15. How do you prevent cache poisoning?
**Answer:**
*   Proper cache key configuration.
*   Limit forwarded headers.
*   Validate query strings.
*   Sanitize inputs.
*   Use WAF rules.

**Architect Insight:** Forwarding too many headers increases the attack surface.

### 16. Design a secure, global SaaS platform using CloudFront.
**Answer:**
*   **Edge:** CloudFront with WAF + Shield.
*   **Storage:** S3 with OAC (Origin Access Control).
*   **Compute:** ALB in private subnet, Auto-scaling backend.
*   **Database:** RDS Multi-AZ.
*   **Access:** Signed URLs for tenant access.
*   **Compliance:** Geo-based restrictions if needed.
*   **Observability:** Centralized logging and SIEM integration.

---

## 17. What Metrics Matter Most for CDN Security?

| Category | Key Metrics |
| :--- | :--- |
| **DDoS** | Request spikes, rate limit triggers |
| **Application** | 4xx/5xx error rate |
| **Abuse** | Geo anomalies |
| **Performance** | Cache hit ratio |
| **Origin Health** | Origin latency |

**❓ Is CloudFront enough for DDoS protection?**
No. It helps, but layered protection with WAF and Shield Advanced is required.

**❓ Should S3 buckets behind CloudFront be public?**
*   **Correct answer:** No. Use OAC to prevent direct access.

### 18. Sudden Spike in 5xx Errors
**Scenario:** Traffic is normal, but users report random failures. CloudFront shows increasing 5xx errors.
*   **Impact:** Intermittent downtime, Revenue loss, SLA breach.
*   **What to Check:** CloudFront 5xx error rate, Origin latency, ALB target health, Backend CPU/memory, Cache hit ratio.
*   **Common Root Causes:** Backend auto-scaling lag, DB connection pool exhaustion, Deployment introduced memory leak, Low cache TTL → excessive origin hits.
*   **Architect Fix:** Increase TTL for cacheable content, Enable Origin Shield, Pre-scale during known traffic peaks, SLO-based alerting on P95 latency.

### 19. Cache Hit Ratio Suddenly Drops
**Scenario:** Origin load spikes even though traffic is stable.
*   **Symptoms:** Cache hit ratio drops from 90% → 40%, Origin CPU spikes, Latency increases.
*   **Root Causes:** Misconfigured cache key (forwarding all headers), Query string forwarding enabled incorrectly, Cookies added unintentionally, New release changed response headers.
*   **Architect Fix:** Restrict forwarded headers, Normalize query strings, Separate static vs dynamic behaviors, Review cache policy carefully.
*   **Insight:** Cache key misconfiguration is one of the most common CDN production issues.

### 20. Origin Bypass Attack
**Scenario:** Attackers discover the direct ALB DNS and bypass CloudFront protections.
*   **Impact:** WAF bypassed, Backend overwhelmed, Security exposure.
*   **Root Cause:** Origin publicly accessible, No IP restriction, No Origin Access Control (for S3).
*   **Fix:** Restrict ALB security group to CloudFront IP ranges, Use custom header validation, For S3 → use OAC, Private subnets.
*   **Principle:** Origin must never be internet-exposed.

### 21. Regional Edge Location Outage
**Scenario:** Users in one geography experience high latency or errors.
*   **Symptoms:** Errors only in specific region, Latency spike from one country.
*   **Root Cause:** Edge location degraded, DNS routing anomaly, ISP routing issue.
*   **Fix:** Monitor geo-distributed metrics, Synthetic monitoring from multiple regions, Enable origin failover, Multi-region origin architecture.

### 22. Deployment Breaks Cached Content
**Scenario:** New deployment changes API response structure but CDN still serves cached old format.
*   **Impact:** Frontend crashes, Inconsistent responses.
*   **Root Cause:** Cache invalidation not triggered, TTL too high.
*   **Fix:** Versioned URLs (best practice), Automated invalidation in CI/CD, Blue/green deployment.
*   **Principle:** Never rely on manual invalidation in production.

### 23. Massive Traffic Spike (Flash Sale / Viral Event)
**Scenario:** Traffic increases 10x within minutes.
*   **Symptoms:** Origin overloaded, 503 errors, Increased latency.
*   **Root Cause:** Dynamic endpoints not cacheable, No autoscaling warm-up, No rate limiting.
*   **Fix:** Cache as much as possible, Use WAF rate limiting, Pre-scale backend, Queue-based buffering.
*   **Mindset:** CDN reduces load only if content is cacheable.

### 24. Bot Scraping Attack
**Scenario:** Traffic doubles but conversions drop.
*   **Symptoms:** High request rate, Low cache hit ratio, Same IP ranges.
*   **Root Cause:** Bot scraping content, No rate limiting, No bot protection.
*   **Fix:** Enable WAF Bot Control, CAPTCHA for suspicious patterns, Signed URLs, IP reputation filtering.

### 25. Cache Poisoning Attack
**Scenario:** Users receive malicious or incorrect cached content.
*   **Root Cause:** Improper cache key config, Forwarding unvalidated headers, Query string injection.
*   **Fix:** Whitelist headers, Validate query params, Strict cache policies, Security testing.

### 26. SSL/TLS Certificate Expiry
**Scenario:** Sudden HTTPS errors across regions.
*   **Symptoms:** TLS handshake failures, Browser security warnings.
*   **Root Cause:** Certificate expired, Incorrect ACM region usage.
*   **Fix:** Auto-renew via ACM, Expiry alerts, Certificate monitoring dashboards.
*   **Insight:** Always automate certificate lifecycle.

### 27. Origin Database Bottleneck
**Scenario:** CloudFront healthy, but latency increasing gradually.
*   **Symptoms:** Origin latency high, DB CPU 90%, Query queue buildup.
*   **Root Cause:** Slow queries, Missing indexes, Increased traffic not matched by DB scaling.
*   **Fix:** Read replicas, Caching layer (Redis), Query optimization, Connection pooling.
*   **View:** CDN cannot fix database bottlenecks.

---

### 28. What is the most common production failure with CDN?
**Answer:**
Misconfigured cache behavior and improper origin protection are more common than CDN outages themselves. Most failures are configuration and architecture issues, not CloudFront service failures.

### 29. Metrics to Detect CDN Failures Early
| Category | Metric |
| :--- | :--- |
| **Availability** | 5xx error rate |
| **Security** | WAF blocked requests |
| **Performance** | Origin latency |
| **Cost Risk** | Cache hit ratio |
| **Abuse** | Request spikes by country |
| **TLS** | Certificate expiration days |
