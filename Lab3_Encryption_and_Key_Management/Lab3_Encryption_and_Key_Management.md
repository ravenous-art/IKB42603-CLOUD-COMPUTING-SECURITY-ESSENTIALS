# IKB42603 Cloud Computing Security Essentials Lab Manual[cite: 10]
**UniKL MIIT · Prof. Dr. Shahrulniza Musa**[cite: 10]

---

## LAB 3 · WEEKS 5–6
### Data Protection: Encryption & Key Management[cite: 10]
*At-rest & in-transit encryption, envelope encryption and cryptographic erasure — OpenSSL & LocalStack KMS*[cite: 10]

---

### Lab Learning Outcomes[cite: 10]
At the end of this lab, you will be able to:
1. Encrypt and decrypt data with symmetric (AES) and asymmetric (RSA) cryptography.[cite: 10]
2. Protect data in transit with TLS and observe the difference between plaintext and encrypted traffic.[cite: 10]
3. Use a Key Management Service (KMS) and implement envelope encryption.[cite: 10]
4. Apply per-tenant keys and perform cryptographic erasure to make data provably unrecoverable.[cite: 10]
5. Verify data integrity with hashing and build a tamper-evident (hash-chained) record.[cite: 10]

---

### Course & Assessment Mapping[cite: 10]

| Item | Mapping |
| :--- | :--- |
| **Course Learning Outcome** | **CLO2** — Construct secure cloud operations that safeguard data integrity (VBE3)[cite: 10] |
| **Lecture topics** | Week 4 (Data Protection) · reinforced by Week 9 (Key Management patterns)[cite: 10] |
| **Value / skill clusters** | VBE3 (Integrity) · SC8 (Integrated Problem-Solving)[cite: 10] |
| **Assessment** | Lab report (outputs + short answers) — contributes to the Lab Assignment[cite: 10] |

---

### Lab Arrangement (2 Sessions over 2 Weeks)[cite: 10]

| Session | Week | Focus |
| :--- | :--- | :--- |
| **Session A** | Week 5 | Symmetric/asymmetric encryption; data at rest and in transit (Tasks 1–3)[cite: 10] |
| **Session B** | Week 6 | KMS, envelope encryption, per-tenant keys, cryptographic erasure, integrity (Tasks 4–7), then the report[cite: 10] |

> **Note:** Session A builds the cryptographic fundamentals by hand.[cite: 10] Session B shows how a cloud KMS manages keys at scale and enables provable deletion.[cite: 10]

---

### Technical Prerequisites[cite: 10]
* A laptop with Docker and a terminal.[cite: 10]
* OpenSSL — pre-installed on macOS/Linux; on Windows use Git Bash or WSL.[cite: 10]
* AWS CLI v2 pointed at LocalStack (as in Lab 1) for the KMS tasks.[cite: 10]
* Basic comfort with the command line.[cite: 10]

> 💡 **Security tip:** Encryption is only as strong as its key management.[cite: 10] Watch carefully in Session B where the keys live — that is the real security control, not the algorithm.[cite: 10]

---

## Session A (Week 5) — Encryption Fundamentals[cite: 10]

### Task 1 — Symmetric Encryption (Data at Rest)[cite: 10]
Create a sensitive file and encrypt it with AES-256.[cite: 10] Then decrypt it.[cite: 10] One shared key does both — fast, but the key must be protected.[cite: 10]

```bash
# Create a sample sensitive record
echo 'Patient: Ahmad, Diagnosis: confidential' > record.txt

# Encrypt with AES-256 (you will be prompted for a passphrase = the key)
openssl enc -aes-256-cbc -pbkdf2 -salt -in record.txt -out record.enc

# Prove it is unreadable
cat record.enc

# Decrypt back
openssl enc -d -aes-256-cbc -pbkdf2 -in record.enc -out record.dec.txt
diff record.txt record.dec.txt && echo 'MATCH: decryption successful'
```

*In your report: what is the key-distribution problem with symmetric encryption, and why does it matter for the cloud?*[cite: 10]

#### Task 1 Evidence
![Task 1 - Symmetric Encryption](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step1.png)

*(Relative Path: `Evidence/task1.png`)*

---

### Task 2 — Asymmetric Encryption & Digital Signatures[cite: 10]
Generate an RSA key pair.[cite: 10] Anyone can encrypt with the public key; only the private key decrypts.[cite: 10] Signatures prove origin and integrity.[cite: 10]

```bash
# Generate a 2048-bit key pair
openssl genrsa -out private.pem 2048
openssl rsa -in private.pem -pubout -out public.pem

# Encrypt with the PUBLIC key, decrypt with the PRIVATE key
openssl pkeyutl -encrypt -pubin -inkey public.pem -in record.txt -out record.rsa
openssl pkeyutl -decrypt -inkey private.pem -in record.rsa -out record.rsa.txt

# Sign with the PRIVATE key; verify with the PUBLIC key
openssl dgst -sha256 -sign private.pem -out record.sig record.txt
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```

> **Note:** Note how the roles reverse: encryption uses the public key, signing uses the private key.[cite: 10] This is the basis of PKI and TLS.[cite: 10]

#### Task 2 Evidence
![Task 2 - Asymmetric Encryption & Digital Signatures](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step2.png)

*(Relative Path: `Evidence/task2.png`)*

---

### Task 3 — Encryption in Transit (TLS)[cite: 10]
Serve a file over HTTPS with a self-signed certificate and confirm the channel is encrypted.[cite: 10]

```bash
# Generate a self-signed certificate
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem \
 -days 7 -nodes -subj '/CN=localhost'

# Serve HTTPS on port 8443 using a small container
docker run --rm -d --name tls -p 8443:443 \
 -v $(pwd)/cert.pem:/etc/nginx/cert.pem -v $(pwd)/key.pem:/etc/nginx/key.pem \
 -v $(pwd)/record.txt:/usr/share/nginx/html/record.txt nginx

# Connect over TLS (-k accepts the self-signed cert)
curl -k https://localhost:8443/record.txt
```

> 💡 **Security tip:** Compare mentally with plain HTTP: over HTTP the record would travel in clear text and any on-path attacker could read it (eavesdropping, Week 3).[cite: 10] TLS makes intercepted traffic unreadable.[cite: 10]

> **Note:** End of Session A.[cite: 10] Stop the TLS container (`docker stop tls`).[cite: 10] Keep `record.enc`, the RSA keys, and all outputs for the report.[cite: 10]

#### Task 3 Evidence
![Task 3 - Encryption in Transit (TLS)](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step3.1.png)

![Task 3 - Encryption in Transit (TLS) 2](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step3.2.png)

*(Relative Path: `Evidence/task3.png`)*

---

## Session B (Week 6) — Key Management, Envelope Encryption & Erasure[cite: 10]

Start LocalStack if it is not running (see Lab 1).[cite: 10] Set `EP='--endpoint-url=http://localhost:4566'`.[cite: 10]

### Task 4 — Create and Use a KMS Master Key[cite: 10]

```bash
EP='--endpoint-url=http://localhost:4566'

# Create a customer master key (CMK) and capture its KeyId
aws $EP kms create-key --description 'CCSE tenant-A master key'

# Copy the KeyId from the output into KEY_A below
KEY_A=<PASTE_KEYID>

# Encrypt a small secret directly with KMS
aws $EP kms encrypt --key-id $KEY_A --plaintext "$(echo -n 'hello' | base64)" \
 --query CiphertextBlob --output text
```

#### Task 4 Evidence
![Task 4 - Create and Use KMS Master Key](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/Lab3_step4.png)

*(Relative Path: `Evidence/task4.png`)*

---

### Task 5 — Envelope Encryption[cite: 10]
For large data you do not encrypt with the master key directly.[cite: 10] You generate a data key, encrypt the data with it locally, and store the data key wrapped by the master key.[cite: 10] This is envelope encryption.[cite: 10]

```bash
# 5.1 Ask KMS for a data key (returns plaintext + encrypted versions)
aws $EP kms generate-data-key --key-id $KEY_A --key-spec AES_256 \
 --query '[Plaintext,CiphertextBlob]' --output text

# Save column 1 as datakey.b64 (plaintext) and column 2 as datakey.enc (wrapped)

# 5.2 Encrypt the big file locally with the PLAINTEXT data key
base64 -d datakey.b64 > datakey.bin
openssl enc -aes-256-cbc -pbkdf2 -in record.txt -out record.env.enc \
 -pass file:./datakey.bin

# 5.3 Destroy the plaintext data key from disk — keep only the wrapped copy
rm datakey.bin datakey.b64
echo 'Only the KMS-wrapped data key (datakey.enc) remains.'
```

> **Note:** To read the data later you send `datakey.enc` back to KMS (`kms decrypt`) to unwrap it, use it, then discard it.[cite: 10] Only the small master key ever needs hardware-grade protection.[cite: 10]

#### Task 5 Evidence
![Task 5 - Envelope Encryption](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step5.png)

*(Relative Path: `Evidence/task5.png`)*

---

### Task 6 — Per-Tenant Keys & Cryptographic Erasure[cite: 10]
Create a second tenant key and show that one tenant's key cannot read another's data.[cite: 10] Then delete a key to make its data permanently unrecoverable — cryptographic erasure.[cite: 10]

```bash
# A separate key for tenant B
aws $EP kms create-key --description 'CCSE tenant-B master key'
KEY_B=<PASTE_KEYID>

# Schedule deletion of tenant A's key (min window)
aws $EP kms schedule-key-deletion --key-id $KEY_A --pending-window-in-days 7

# Disable it immediately to simulate erasure
aws $EP kms disable-key --key-id $KEY_A

# Attempt to unwrap tenant A's data key now — it should FAIL
aws $EP kms decrypt --ciphertext-blob fileb://datakey.enc 2>&1 | head -3
```

> ⚠️ **Caution:** Once the key that wrapped the data key is gone, `record.env.enc` is just noise — no one, not even the provider, can decrypt it.[cite: 10] This is why per-object/per-tenant keys make deletion provable (Week 4).[cite: 10]

#### Task 6 Evidence
![Task 6 - Per-Tenant Keys & Cryptographic Erasure](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step6.png)

*(Relative Path: `Evidence/task6.png`)*

---

### Task 7 — Integrity & Tamper-Evidence[cite: 10]
Encryption protects confidentiality; hashing protects integrity.[cite: 10] Detect tampering and build a simple hash chain.[cite: 10]

```bash
# Fingerprint the file
sha256sum record.txt

# Tamper with a copy and show the hash changes
cp record.txt tampered.txt; echo 'x' >> tampered.txt
sha256sum record.txt tampered.txt

# Hash chain: each entry includes the previous hash (tamper-evident log)
PREV=0
for line in 'login ok' 'file read' 'export data'; do \
 PREV=$(echo -n "$PREV$line" | sha256sum | cut -d' ' -f1); \
 echo "$line | $PREV"; done
```

#### Task 7 Evidence
![Task 7 - Integrity & Tamper-Evidence](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_step7.png)
*(Relative Path: `Evidence/task7.png`)*

---

## Deliverables & Assessment[cite: 10]

### 1. Evidence (label each clearly)[cite: 10]
* AES encrypt/decrypt with the `MATCH` confirmation (Task 1).[cite: 10]
* RSA signature verify output showing ‘Verified OK’ (Task 2).[cite: 10]
* The `curl -k https://…` output over TLS (Task 3).[cite: 10]
* The KMS KeyId(s) and the envelope-encryption steps (Tasks 4–5).[cite: 10]
* The failed `kms decrypt` after key erasure (Task 6).[cite: 10]
* The two differing SHA-256 hashes and the hash chain (Task 7).[cite: 10]

### 2. Short-Answer Questions Solutions[cite: 10]

#### Q1. Compare symmetric and asymmetric encryption: speed, key distribution, and typical use.[cite: 10]
* **Speed:** Symmetric encryption (e.g., AES-256) is computationally fast and lightweight, making it ideal for processing large files or continuous data streams. Asymmetric encryption (e.g., RSA-2048) relies on complex modular arithmetic, making it orders of magnitude slower.
* **Key Distribution:** Symmetric encryption uses a single shared key for both encryption and decryption, creating a major distribution challenge—the key must be securely transmitted beforehand over an out-of-band channel. Asymmetric encryption uses a public/private key pair; the public key can be shared openly across untrusted networks while the private key remains secret.
* **Typical Use:** Symmetric encryption is primarily used for bulk data protection at rest (e.g., encrypting disk volumes, database columns, S3 objects). Asymmetric encryption is used for identity verification, digital signatures, key exchange protocols, and establishing initial secure channels (e.g., TLS/HTTPS handshakes).

#### Q2. Why is key management described as the weakest link, not the algorithm?[cite: 10]
Modern cryptographic algorithms like AES-256 or RSA-2048 are mathematically resilient against brute-force attacks given current computational capabilities. However, security collapses if operational key management fails. Vulnerabilities such as hardcoding keys into source code, storing keys in plaintext on disk, retaining overly permissive IAM roles, failing to rotate compromised keys, or leaking private keys during transit expose decrypted data without an attacker ever needing to break the cipher itself. Therefore, the key lifecycle (generation, storage, access control, and revocation) represents the actual attack surface.

#### Q3. Explain envelope encryption and why only the master key needs hardware-grade protection.[cite: 10]
Envelope encryption is a hierarchical key management practice where data is encrypted locally using a unique, fast symmetric Data Encryption Key (DEK). The DEK itself is then encrypted (wrapped) using a Customer Master Key (CMK) stored inside a Key Management Service (KMS).

Only the CMK requires hardware-grade protection (such as FIPS 140-2/3 validated Hardware Security Modules) because:
1. **Performance & Scalability:** Sending large payloads over a network to an HSM for direct encryption introduces latency and network bottlenecks. Local DEK encryption keeps data processing fast.
2. **Blast Radius Control:** The CMK never leaves the HSM boundary. KMS only performs lightweight operations—wrapping and unwrapping the tiny 256-bit DEKs. By protecting the single master key in hardware, all underlying DEKs wrapped by it inherit the same security standard.

#### Q4. How does cryptographic erasure achieve provable deletion where overwriting cannot (in the cloud)?[cite: 10]
In multi-tenant cloud environments, data is distributed, virtualized, replicated across multiple availability zones, and automatically backed up to snapshot storage. Physically overwriting every physical sector or zeroing out storage drives (magnetic shredding) is technically impossible and unverifiable for cloud tenants.

Cryptographic erasure guarantees provable deletion by destroying or permanently revoking the specific Master Key or per-tenant DEK used to encrypt the dataset. Without the decryption key, the ciphertext stored across all physical drives, snapshots, and backups instantly reverts to un-decryptable high-entropy mathematical noise. Once the key is provably deleted from the KMS, the data becomes unrecoverable by anyone, including the cloud service provider.

#### Q5. How does a hash chain make a log tamper-evident (link to tamper-proof logs, Week 6)?[cite: 10]
A hash chain operates by linking sequential log entries cryptographically, where each entry's SHA-256 hash calculation incorporates the hash value of the immediately preceding entry ($H_n = \text{Hash}(\text{Entry}_n + H_{n-1})$). 

If an attacker modifies, inserts, or deletes a historical log entry at index $k$, the resulting hash $H_k$ will change. Because every subsequent hash ($H_{k+1}, H_{k+2}, \dots$) depends on $H_k$, the entire downstream chain breaks. During automated verification, comparing recalculated hashes against the stored chain instantly highlights any point of tampering, rendering the log store tamper-evident.

### 3. Verification Command[cite: 10]
```bash
aws --endpoint-url=http://localhost:4566 kms list-keys
openssl dgst -sha256 -verify public.pem -signature record.sig record.txt
```
![Task cleanup - cleanup](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_cleanup.png)

```

```

## Security Best-Practices Checklist[cite: 10]
- [x] Data encrypted at rest (AES) and decryption verified.[cite: 10]
- [x] Asymmetric keys used correctly (encrypt with public, sign with private).[cite: 10]
- [x] Data protected in transit with TLS.[cite: 10]
- [x] Envelope encryption used; plaintext data key not left on disk.[cite: 10]
- [x] Per-tenant keys used; cryptographic erasure demonstrated.[cite: 10]
- [x] Integrity verified with hashing / hash chain.[cite: 10]

---

## Cleanup & Teardown[cite: 10]
```bash
docker stop tls 2>/dev/null
rm -f record.* private.pem public.pem key.pem cert.pem datakey.* tampered.txt
docker stop localstack && docker rm localstack
```
![Task verification - verify](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab3_Encryption_and_Key_Management/Evidence/lab3_verification.png)

---

## Expansion Ideas (Advanced Students)[cite: 10]
* Store the master key in a software HSM (SoftHSM) and use PKCS#11 to sign — model hardware key protection.[cite: 10]
* Stand up HashiCorp Vault in a container and use its transit engine for envelope encryption.[cite: 10]
* Configure mutual TLS (mTLS) between two containers so both sides authenticate.[cite: 10]
* Automate key rotation and re-wrap existing data keys under a new master key.[cite: 10]

---

## References[cite: 10]
* Course lecture — Week 4 (Data Protection); Week 9 (Key Management patterns).[cite: 10]
* OpenSSL documentation — www.openssl.org/docs[cite: 10]
* AWS KMS concepts (envelope encryption) — docs.aws.amazon.com/kms[cite: 10]
* CSA Security Guidance v5 — Data Security & Encryption.[cite: 10]
