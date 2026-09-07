# IKB42603 Cloud Computing Security Essentials
## Lab 6 Report: Object Storage Security & the Data Security Lifecycle[cite: 1]

**Student Name:** SYAHMI IKBAL  
**Course Code:** IKB42603[cite: 1]  
**Repository Path:** `Lab6_Object_Storage_and_Data_Lifecycle/`  
**Evidence Directory:** [GitHub Evidence Folder](https://github.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/tree/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence)

---

## 1. Screenshots & Task Evidences[cite: 1]

### Task 1: Data Classification Before Storage[cite: 1]
- **Description:** Created the patient records bucket, uploaded three objects of varying sensitivity (`public/notice.txt`, `internal/roster.txt`, `confidential/record.txt`), and assigned key-value tags (`classification=public`, `classification=internal`, `classification=confidential`)[cite: 1].
- **Evidences:**
  - Formatted object listing and confidential tag verification:  
    ![Task 1 Object Listing](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task1.png)

---

### Task 2: Reproducing the Archetypal Breach[cite: 1]
- **Description:** Misconfigured the bucket policy by setting `"Principal": "*"` to grant unrestricted global access, then retrieved the unauthenticated confidential record using an anonymous `curl` request[cite: 1].
- **Evidences:**
  - Anonymous `curl` returning `HTTP 200` with unauthenticated patient diagnosis:  
    ![Task 2 Public Leak](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task2.png)

---

### Task 3: Remediating with Block Public Access & Least Privilege[cite: 1]
- **Description:** Deleted the offending public policy, applied the account-level Block Public Access guardrail across all four flags, verified configuration enforcement, and applied a least-privilege policy scoped strictly to the `internal/*` prefix[cite: 1].
- **Evidences:**
  - Block Public Access configuration verification output (`True, True, True, True`):  
    ![Task 3 Guardrails](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task3.png)
  - Least-privilege resource policy applied:  
    ![Task 3 Least Privilege](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task3.1.png)

---

### Task 4: Identity Policy vs. Resource Policy[cite: 1]
- **Description:** Created the `DataAnalyst` IAM user with full S3 read permissions (`S3ReadAll`)[cite: 1]. Applied a bucket resource policy explicitly denying access to `confidential/*` while allowing access to `internal/*` to test evaluation rules[cite: 1].
- **Evidences:**
  - IAM user creation and credential assignment:  
    ![Task 4 IAM User Setup](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task4.png)
  - Resource policy definition with explicit `Deny`:  
    ![Task 4 Policy Document](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task4.1.png)
  - Execution test output under `analyst` profile (`internal: ALLOWED`):  
    ![Task 4 Execution Test](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task4.2.png)

---

### Task 5: Default Encryption at Rest (SSE-KMS)[cite: 1]
- **Description:** Generated a Customer Managed KMS Key (`$KEY_ID`) and configured bucket-wide default server-side encryption (`aws:kms`) with `BucketKeyEnabled=true`[cite: 1]. Uploaded an object without encryption parameters to confirm automatic key application[cite: 1].
- **Evidences:**
  - KMS key creation and default encryption configuration:  
    ![Task 5 KMS Setup](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task5.png)
  - `head-object` confirmation displaying `aws:kms`, KMS Key ARN, and `BucketKeyEnabled: True`:  
    ![Task 5 Head Object](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task5.1.png)

---

### Task 6: Delegated Access and Condition-Key Mechanics[cite: 1]
- **Description:** Issued a time-bounded presigned URL (60 seconds) for temporary access[cite: 1]. Created and evaluated the `aws:SecureTransport` TLS condition policy to demonstrate environment-dependent policy evaluation mechanics[cite: 1].
- **Evidences:**
  - Presigned URL generation and expiration testing:  
    ![Task 6 Presigned URL](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task6.png)
  - Transport condition policy formulation:  
    ![Task 6 Condition Policy](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task6.1.png)
  - Object listing response under condition policy:  
    ![Task 6 Execution Test](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task6.2.png)
  - Policy state verification (`aws:SecureTransport`):  
    ![Task 6 Policy Output](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task6.3.png)

---

### Task 7: Versioning, Delete Markers & Data Remanence[cite: 1]
- **Description:** Enabled object versioning, uploaded object revisions, and executed a standard `delete-object` command to write a Delete Marker[cite: 1]. Retrieved the unredacted historical revision using `--version-id null` to demonstrate object-level data remanence[cite: 1].
- **Evidences:**
  - Version history listing and Delete Marker creation:  
    ![Task 7 Versioning Table](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task7.png)
  - Data remanence recovery output (`recovered.txt` containing original diagnosis):  
    ![Task 7 Data Remanence Recovery](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task7.1.png)

---

### Task 8: Lifecycle, Retention & Cryptographic Erasure[cite: 1]
- **Description:** Configured automated lifecycle rules for object expiration and incomplete multipart upload cleanup[cite: 1]. Executed cryptographic erasure by disabling and scheduling deletion of the underlying Customer Managed KMS Key[cite: 1].
- **Evidences:**
  - Lifecycle rules configuration and KMS key state set to `PendingDeletion`:  
    ![Task 8 Lifecycle & Cryptographic Erasure](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_task8.png)

---

### Task Cleanup Verification
- **Description:** Permanently purged all historical object versions and delete markers, deleted the bucket, and removed temporary IAM accounts and LocalStack containers[cite: 1].
- **Evidences:**
  - Object versions purge output:  
    ![Cleanup Versions Purge](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_cleanup.png)
  - Delete markers purge and final container environment teardown:  
    ![Cleanup Final Teardown](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_cleanup2.png)

---

## 2. Data Classification Table[cite: 1]

| Classification | Who may read it | Impact if leaked | Control Applied |
| :--- | :--- | :--- | :--- |
| **public**[cite: 1] | Unauthenticated Public[cite: 1] | None / Negligible | Unrestricted access / Key prefixing (`public/`)[cite: 1] |
| **internal**[cite: 1] | Authenticated Employees / Account Users[cite: 1] | Low to Moderate (Operational exposure) | Least-privilege bucket policy (`internal/*`)[cite: 1] |
| **confidential**[cite: 1] | Authorized Personnel Only[cite: 1] | High (Privacy breach, regulatory non-compliance)[cite: 1] | Explicit Deny policies, SSE-KMS encryption, lifecycle retention[cite: 1] |

---

## 3. Short-Answer Questions[cite: 1]

### Question 1: Exposure Primary Cause & Risk Comparison[cite: 1]
- **Primary Cause:** The inclusion of `"Principal": "*"` within the bucket resource policy allowed anonymous, unauthenticated HTTP access to `s3:GetObject`[cite: 1].
- **Risk Comparison:** An over-broad IAM policy attached to a single user only expands permissions for callers authenticated with those specific AWS credentials[cite: 1]. A bucket policy containing `"Principal": "*"` exposes the resource globally across the public internet to anyone who discovers the URL endpoint, requiring no credentials or identity verification whatsoever[cite: 1].

---

### Question 2: Policy Types & Authorization Evaluation Rules[cite: 1]
- **Policy Definitions:**
  - *Identity-Based Policies:* Attached to IAM identities (users, groups, roles) defining what actions they can perform across resources[cite: 1].
  - *Resource-Based Policies:* Attached directly to resources (S3 buckets) defining which principals can perform actions on that specific resource[cite: 1].
- **Evaluation Logic Pipeline:** Evaluation follows the explicit precedence: **Default Deny → Any Explicit Deny → Any Explicit Allow**[cite: 1].
  - `internal/roster.txt`: Evaluates to **ALLOW** because both the user's IAM policy (`S3ReadAll`) and the bucket policy statement `AllowAnalystInternal` grant explicit `Allow` permissions[cite: 1].
  - `confidential/record.txt`: Evaluates to **DENY** because statement `DenyAnalystConfidential` in the bucket policy enforces an explicit `Deny` (`"Action": "s3:*"` on `"Resource": ".../confidential/*"`)[cite: 1]. An explicit `Deny` overrides any explicit `Allow` granted by identity policies[cite: 1].

---

### Question 3: Security Guardrails vs. Operational Controls[cite: 1]
- **Core Distinction:** A standard security control (such as a bucket policy) operates at the resource level and can be altered, misconfigured, or removed by individual developers[cite: 1]. A administrative guardrail (like Block Public Access) operates at the account level to override and prevent misconfigurations across all managed resources regardless of individual policies[cite: 1].
- **Organizational Impact:** Centralized guardrails enforce non-negotiable security baselines automatically across large engineering teams, mitigating human error and preventing accidental exposures at scale[cite: 1].

---

### Question 4: SSE-KMS Protection Boundaries[cite: 1]
- **Protection Scope:** Default SSE-KMS provides encryption at rest, protecting physical storage media from theft, underlying hypervisor exposure, or direct disk reads[cite: 1].
- **Analyst Threat Model:** SSE-KMS does **not** protect against the analyst in Task 4[cite: 1]. Because the analyst interacts via authorized S3 API requests, S3 automatically handles decryption using the associated KMS key on behalf of authenticated callers possessing `kms:Decrypt` access[cite: 1]. Storage-level encryption does not replace API-level authorization controls[cite: 1].

---

### Question 5: Privacy Erasure Compliance & Provable Deletion[cite: 1]
- **`delete-object` Non-Compliance:** Executing a standard `delete-object` command on a versioned bucket merely writes a 0-byte Delete Marker over the top as the latest revision[cite: 1]. The original unredacted historical versions remain intact on physical storage and remain accessible via explicit `--version-id` calls, failing privacy erasure requirements under PDPA/GDPR[cite: 1].
- **Two Provable Deletion Mechanisms:**
  1. *Per-Version Purging:* Issuing explicit `delete-object` commands targeting every individual `VersionId` to remove all underlying data blocks[cite: 1].
  2. *Cryptographic Erasure:* Revoking or permanently destroying the Customer Managed KMS Key used to encrypt the dataset[cite: 1]. Without the decryption key, all stored ciphertext across primary storage, backups, and replicas becomes unrecoverable noise[cite: 1].

---

### Question 6: Compliance Evidence Collection[cite: 1]
1. `aws s3api get-public-access-block`: Evidences account-level preventative guardrails blocking public bucket exposures[cite: 1].
2. `aws s3api get-bucket-encryption`: Evidences mandatory default server-side encryption at rest (SSE-KMS) across all uploaded objects[cite: 1].
3. `aws s3api get-bucket-lifecycle-configuration`: Evidences automated, policy-driven data retention and expiration rule enforcement[cite: 1].

---

## 4. Verification Command Output[cite: 1]

Executed script output verifying final security posture[cite: 1]:

```text
=== IKB42603 Lab 6 verification: miit-patient-records-29203 ===
True    True    True    True
Enabled
aws:kms f2c54e7b-076c-4ca7-bed5-95133868576f
RetireConfidentialRecords       Enabled
AbortIncompleteUploads  Enabled
PendingDeletion
```
![Verification](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab6_Object_Storage_and_Data_Lifecycle/Evidence/Lab6_verification.png)

