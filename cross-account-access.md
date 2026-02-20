# AWS Cross-Account Access Interview Questions

This document contains common interview questions related to AWS Cross-Account Access, IAM Roles, and Security.

## 🔹 Basic Level Questions

### 1. What is AWS Cross-Account Access?
**Answer:**
Cross-account access allows resources in one AWS account to securely access resources in another AWS account using IAM roles, policies, and trust relationships, without sharing long-term credentials (like access keys).

### 2. How is cross-account access typically achieved in AWS?
**Answer:**
It is achieved using:
*   **IAM Roles:** An identity with specific permissions.
*   **Trust Policies:** Defines who can assume the role.
*   **Resource-based Policies:** Attached directly to resources (e.g., S3 Bucket Policy).
*   **AWS STS (Security Token Service):** Generates temporary security credentials.

**Main components:**
1.  Trust relationship (Who can assume).
2.  AssumeRole permission (Permission to switch).
3.  Temporary credentials via STS.

### 3. What is an IAM Role in cross-account access?
**Answer:**
An IAM Role is an AWS identity with permission policies that determine what actions are allowed. Unlike a user, it does not have long-term credentials (password or access keys). One account creates a role and allows another account to assume it.

### 4. What is a Trust Policy?
**Answer:**
A trust policy is a JSON document attached to an IAM role that defines **who** (the principal) is allowed to assume the role.

**Example:**
```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::123456789012:root"
  },
  "Action": "sts:AssumeRole"
}
```

### 5. What is AWS STS?
**Answer:**
**AWS Security Token Service (STS)** is a web service that enables you to request temporary, limited-privilege credentials for AWS IAM users or for users that you authenticate (federated users). It is the core mechanism behind `AssumeRole`.

### 6. Explain the steps for setting up cross-account access.
**Answer:**
1.  **Account A (Target):** Create an IAM Role.
2.  **Account A:** Add a **Trust Policy** allowing Account B to assume this role.
3.  **Account A:** Attach a **Permission Policy** to the role (defining what the role can do).
4.  **Account B (Source):** Grant an IAM User/Role permission to call `sts:AssumeRole` on the role ARN in Account A.
5.  **Action:** User in Account B assumes the role and receives temporary credentials to access Account A.

### 7. Difference between Resource-based policy and Role-based access?
**Answer:**

| Feature | Role-Based Access | Resource-Based Policy |
| :--- | :--- | :--- |
| **Mechanism** | User assumes an IAM Role. | Policy attached directly to the resource. |
| **Credentials** | Requires switching identity (AssumeRole). | No identity switch required; uses original credentials. |
| **Common Use Cases** | Cross-account EC2, Lambda, Admin access. | S3 Buckets, SQS Queues, Lambda, KMS Keys. |
| **Complexity** | Higher (requires STS). | Lower (direct access). |

### 8. Can you access S3 in another account without assuming a role?
**Answer:**
**Yes.** You can use an **S3 Bucket Policy** (Resource-based policy) in the target account to explicitly grant permissions to the IAM user ARN of the source account. The source user stays in their own account but can read/write to the target bucket.

### 9. What is External ID and why is it used?
**Answer:**
An **External ID** is an optional string that you can use in an IAM role trust policy to designate who can assume the role.
*   **Purpose:** It prevents the **Confused Deputy Problem**.
*   **Usage:** It ensures that a third-party service (like a SaaS monitoring tool) can only assume your role when acting on *your* behalf, not on behalf of another customer.

### 10. What is the confused deputy problem?
**Answer:**
A security issue where an entity (e.g., a third-party service) that doesn't have permission to perform an action can be coerced by a malicious party to misuse its privileges to perform the action on the malicious party's behalf.

### 11. How do you restrict cross-account access to specific users only?
**Answer:**
You can restrict access using the Trust Policy conditions:
*   **Principal:** Specify the specific User ARN instead of the root account.
*   **Condition Keys:**
    *   `aws:PrincipalArn`: Restrict to specific ARNs.
    *   `aws:MultiFactorAuthPresent`: Require MFA.
    *   `sts:ExternalId`: Enforce External ID.

### 12. How do you enable cross-account access between EC2 instances?
**Answer:**
1.  Create an IAM Role in the **Target Account** with the necessary permissions and a trust policy allowing the Source Account.
2.  Create an IAM Role for the **EC2 Instance** in the **Source Account**.
3.  Attach a policy to the EC2 role allowing `sts:AssumeRole` on the Target Role ARN.
4.  The application on EC2 calls STS to assume the target role and uses the returned credentials.

### 13. What is the difference between `AssumeRole` and `AssumeRoleWithSAML`?
**Answer:**
*   **`AssumeRole`:** Used by IAM users, roles, or AWS services to assume a role.
*   **`AssumeRoleWithSAML`:** Used for **Federated Identity**. It allows users authenticated via a SAML 2.0 compliant Identity Provider (IdP) like Active Directory or Okta to assume an IAM role without needing IAM user credentials.

### 14. How does cross-account access work with AWS Organizations?
**Answer:**
AWS Organizations simplifies cross-account access by allowing you to:
*   Use **Service Control Policies (SCPs)** to set permission boundaries across accounts.
*   Easily create roles (like `OrganizationAccountAccessRole`) in member accounts during creation.
*   Use `PrincipalOrgID` in resource policies to allow access to the entire organization easily.

### 15. How do you audit cross-account activity?
**Answer:**
*   **AWS CloudTrail:** Logs all API calls, including `AssumeRole` events. You can look for events where `userIdentity.accountId` differs from the resource's account ID.
*   **IAM Access Analyzer:** Analyzes resource-based policies to identify resources shared with external principals.

### 16. What is IAM Access Analyzer?
**Answer:**
A service that mathematically analyzes resource-based policies (S3 buckets, KMS keys, etc.) to identify and alert you about resources that are accessible from outside your AWS account or organization.

### 17. How do temporary credentials work in cross-account access?
**Answer:**
1.  User calls `sts:AssumeRole`.
2.  STS validates the request.
3.  STS returns a set of temporary credentials:
    *   **Access Key ID**
    *   **Secret Access Key**
    *   **Session Token**
4.  These credentials expire after the defined **Session Duration** (15 minutes to 12 hours).

### 18. How do you secure cross-account access?
**Answer:**
**Best Practices:**
*   **Least Privilege:** Grant only necessary permissions.
*   **External ID:** Use for third-party access.
*   **MFA:** Require MFA in the trust policy (`"Bool": {"aws:MultiFactorAuthPresent": "true"}`).
*   **Short Session Duration:** Limit the time credentials are valid.
*   **Monitoring:** Use CloudTrail and Access Analyzer.
*   **SCPs:** Use Service Control Policies to restrict maximum permissions.

### 19. Company A wants to allow Company B to upload files to a specific S3 bucket. How would you design it?
**Answer:**
**Option 1 (Role-Based - Preferred for programmatic access):**
1.  Create an IAM Role in Company A with `s3:PutObject` permission on the bucket.
2.  Trust Company B's account ID in the role's trust policy.
3.  Company B assumes the role to upload files.

**Option 2 (Resource-Based - Simpler for S3):**
1.  Add a **Bucket Policy** to the S3 bucket in Company A.
2.  Grant `s3:PutObject` permission to `arn:aws:iam::CompanyB-Account-ID:root` (or specific user).

### 20. A Lambda function in Account A needs to access DynamoDB in Account B. How would you configure it?
**Answer:**
1.  **Account B:** Create an IAM Role with permissions to access the DynamoDB table.
2.  **Account B:** Configure the Trust Policy to allow the **Lambda Execution Role ARN** from Account A to assume it.
3.  **Account A:** Attach a policy to the Lambda Execution Role allowing `sts:AssumeRole` on the role in Account B.
4.  **Code:** In the Lambda function code, call STS to assume the role, get temporary credentials, and instantiate the DynamoDB client with those credentials.

### 21. Can a role assume another role?
**Answer:**
**Yes**, this is called **Role Chaining**.
*   **Limitation:** When you chain roles (Role A assumes Role B), the maximum session duration for Role B is limited to **1 hour**, regardless of the role's configured maximum duration.

### 22. What happens if both IAM policy and SCP allow access?
**Answer:**
**Access is Allowed.**
*   SCPs (Service Control Policies) act as a **filter** or boundary. They do not grant permissions themselves; they only allow or deny what the IAM policy grants.
*   For access to be granted, **both** the Identity-based policy (IAM) AND the SCP (if present) must allow the action.

### 23. Can you deny access even if trust policy allows it?
**Answer:**
**Yes.**
*   An **Explicit Deny** in any relevant policy (Identity-based policy, SCP, or Resource-based policy) always overrides an Allow.
*   Even if the Trust Policy allows the assumption, if the user's IAM policy has a `Deny` on `sts:AssumeRole`, the request will fail.

### 24. When would you use a Resource-Based Policy instead of an IAM Role for cross-account access?
**Answer:**
Resource-based policies (like S3 Bucket Policies or KMS Key Policies) allow a principal to access a resource without switching roles.

*   **Use Resource-Based Policies when:** You want the user to stay in their own account while accessing the resource (e.g., uploading a file to a central logging bucket). This avoids the overhead of `AssumeRole` and allows the user to retain their original permissions.
*   **Use IAM Roles when:** The task requires multiple actions across different services in the target account, or when the service doesn't support resource-based policies (like EC2 or IAM).

### 25. A Lambda function in Account A needs to read from a DynamoDB table in Account B. Walk me through the setup.
**Answer:**
1.  **In Account B (Destination):** Create an IAM Role.
    *   **Trust Policy:** Allow Account A's root (or better, the specific Lambda execution role ARN) to `sts:AssumeRole`.
    *   **Permissions Policy:** Grant `dynamodb:GetItem` or `dynamodb:Query` on the specific table.
2.  **In Account A (Source):** Update the Lambda's execution role.
    *   **Policy:** Add `sts:AssumeRole` permission pointing to the Role ARN in Account B.
3.  **In the Code:** Use the AWS SDK to call `sts.assume_role()`, then use those returned temporary credentials to initialize the DynamoDB client.

### 26. How does policy evaluation change when cross-account access is involved?
**Answer:**
For cross-account access, AWS follows a **double-check logic**:
1.  **Account A (Owner)** must explicitly grant permission (via Role Trust Policy or Resource Policy).
2.  **Account B (User)** must explicitly grant permission (via Identity Policy).

**Result:** Access is only granted if **both** accounts allow it. An explicit Deny in either account will always override any Allow.
