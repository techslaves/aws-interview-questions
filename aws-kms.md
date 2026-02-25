# AWS KMS Interview Questions and Answers

This document contains common interview questions related to AWS Key Management Service (KMS), encryption, and key security.

### 1. What is AWS KMS?
**Answer:**
**AWS KMS (Key Management Service)** is a managed service by Amazon Web Services that allows you to create, manage, and control cryptographic keys used to encrypt data across AWS services.

**It helps with:**
*   Data encryption.
*   Key lifecycle management.
*   Access control.
*   Compliance requirements.

### 2. What are KMS keys?
**Answer:**
KMS keys are logical representations of cryptographic keys used for encryption and decryption operations.

**There are two main types:**

**a) AWS Managed Keys**
*   Created and managed automatically by AWS.
*   Used by AWS services like S3, EBS, etc.
*   **Limitation:** No manual control over the key policy.

**b) Customer Managed Keys (CMK)**
*   Created, managed, and controlled by the customer.
*   **Full control over:** Key policy, Rotation, Deletion, and Permissions.

### 3. What is the difference between AWS Managed Keys and Customer Managed Keys?
**Answer:**

| Feature | AWS Managed Key | Customer Managed Key |
| :--- | :--- | :--- |
| **Creation** | AWS | Customer |
| **Cost** | Free | Charged monthly |
| **Key Policy Control** | Limited | Full control |
| **Rotation Control** | Automatic (every 3 years) | Configurable (every 1 year) |
| **Deletion** | Not allowed | Allowed |

### 4. What is Envelope Encryption?
**Answer:**
Envelope encryption is a method where:
1.  A **Data Key** encrypts the actual data.
2.  The **Data Key** itself is encrypted using a root **KMS Key**.
3.  The encrypted data key is stored along with the encrypted data.

**Benefits:**
*   **Performance:** Symmetric data keys are faster for large data.
*   **Security:** The master key never leaves KMS.
*   **Cost:** Reduces KMS API calls (you only call KMS to decrypt the data key, not the whole dataset).

### 5. What is a Data Key in KMS?
**Answer:**
A data key is generated using the `GenerateDataKey` API call.
**It returns two things:**
1.  **Plaintext data key:** Used to encrypt data in memory (should be deleted immediately after use).
2.  **Encrypted data key:** Stored safely alongside the encrypted data.

### 6. What is Key Rotation?
**Answer:**
Key rotation is the process of replacing old cryptographic material with new material while keeping the same Key ID.
*   **Support:** Supported for symmetric customer-managed keys.
*   **Frequency:** Can be enabled to rotate automatically every **year**.
*   **Benefit:** Helps meet compliance requirements and limits the amount of data encrypted with a single key version.

### 7. What are the types of KMS keys?
**Answer:**
1.  **Symmetric Keys:**
    *   Most commonly used.
    *   Used for encryption/decryption.
    *   Single key for both operations.
2.  **Asymmetric Keys:**
    *   Public and private key pair.
    *   Used for **Digital signing** and **Verification** (or encryption outside AWS).

### 8. What is the difference between Symmetric and Asymmetric Keys?
**Answer:**

| Feature | Symmetric | Asymmetric |
| :--- | :--- | :--- |
| **Structure** | Single key | Public/private pair |
| **Speed** | Faster | Slower |
| **Use Case** | Bulk encryption | Signing, verification, external exchange |

### 9. What is a Key Policy?
**Answer:**
A **Key Policy** is a JSON document attached to a KMS key that defines:
*   Who can use the key.
*   What actions are allowed (e.g., `kms:Encrypt`, `kms:Decrypt`).
*   Which AWS principals have access.

**Note:** It is **mandatory** for every KMS key. Without a key policy, no one (not even the root user) can access the key.

### 10. What is the difference between IAM Policy and Key Policy?
**Answer:**

| Feature | IAM Policy | Key Policy |
| :--- | :--- | :--- |
| **Attached to** | Users/Roles | KMS Key |
| **Scope** | Grants permissions broadly | Controls key-specific access |
| **Requirement** | Not sufficient alone for KMS | **Required** for KMS access |

**Rule:** For KMS access, the **Key Policy** must allow access, OR the Key Policy must allow the account to use IAM policies to grant access.

### 11. What are Grants in AWS KMS?
**Answer:**
A **Grant** allows temporary, delegated permission to use a KMS key.
*   **Use Case:** When AWS services (like EC2 or ASG) need short-term access to encrypt/decrypt volumes on your behalf without modifying the Key Policy directly.

### 12. Can KMS keys be shared across accounts?
**Answer:**
**Yes.**
To share a key:
1.  Modify the **Key Policy** in the source account to add the external AWS Account ID.
2.  Add an **IAM Policy** in the external account to allow the user/role to use the key.

### 13. What is the difference between Disable and Schedule Deletion of a KMS key?
**Answer:**

| Feature | Disable Key | Schedule Deletion |
| :--- | :--- | :--- |
| **State** | Temporarily unusable | Permanently deleted (after wait period) |
| **Recovery** | Can re-enable instantly | Cannot recover after deletion |
| **Effect** | Immediate | Waiting period (7–30 days) |

### 14. What is a Customer Master Key (CMK)?
**Answer:**
**CMK** was the old term for Customer Managed Keys. AWS now simply refers to them as **KMS keys**.

### 15. How does AWS KMS integrate with other AWS services?
**Answer:**
KMS integrates seamlessly to provide encryption at rest for:
*   **Amazon S3** (SSE-KMS).
*   **Amazon EBS** (Volume encryption).
*   **Amazon RDS** (Database encryption).
*   **AWS Lambda** (Environment variable encryption).
*   **Amazon Redshift**.

### 16. What is BYOK (Bring Your Own Key)?
**Answer:**
BYOK allows you to import your own cryptographic key material into KMS instead of having AWS generate it.
*   **Benefit:** Greater control over key origin and durability.
*   **Risk:** If the imported key material expires or is deleted, the key becomes unusable and data is lost.

### 17. What is a Multi-Region Key in KMS?
**Answer:**
A Multi-Region key is a KMS key that has the **same Key ID** across multiple AWS regions.
*   **Use Case:** Disaster Recovery (DR) and Active-Active architectures.
*   **Benefit:** You can encrypt data in one region and decrypt it in another without re-encrypting.

### 18. What is the pricing model for AWS KMS?
**Answer:**
*   **Storage:** Monthly fee for each Customer Managed Key (approx. $1/month).
*   **Usage:** Charged per API request (e.g., per 10,000 requests).
*   **Free:** AWS Managed Keys are free to store.

### 19. What happens if a KMS key is deleted?
**Answer:**
Any data encrypted with that key becomes **permanently inaccessible**. There is no way to recover the data once the key is gone.
*   **Best Practice:** Always verify usage before scheduling deletion.

### 20. How do you secure KMS keys?
**Answer:**
*   **Enable Key Rotation:** Automatically rotate backing keys.
*   **Least Privilege:** Restrict Key Policies to specific users/roles.
*   **Monitor:** Use **CloudTrail** to audit key usage (`kms:Decrypt` events).
*   **Conditions:** Use `kms:ViaService` to restrict key usage to specific AWS services (e.g., only S3 can use this key).
*   **MFA:** Require MFA for deletion operations.
