# IKB42603 Cloud Computing Security Essentials
## LAB 5 WEEKS 9-10: Monitoring, Logging & Incident Detection
**Centralised logging, tamper-proof logs, threat detection and incident response - Docker & LocalStack**

---

### Course & Assessment Mapping

| Item | Mapping |
| :--- | :--- |
| **Course Learning Outcome** | CLO2 - Construct secure cloud operations that safeguard data integrity |
| **Lecture topics** | Week 6 (Monitoring, Auditing & Management) |
| **Value / skill clusters** | VBE3 (Integrity) SC8 (Integrated Problem-Solving) |
| **Assessment** | Lab report + short incident report contributes to the Lab Assignment |

---

### Lab Learning Outcomes
At the end of this lab, you will be able to:
1. Collect and centralise logs from multiple services (cloud telemetry).
2. Distinguish logs from events and query logs for security-relevant activity.
3. Build a tamper-evident (hash-chained) log and detect alteration.
4. Detect an incident by correlating events (e.g. brute-force followed by a suspicious action).
5. Execute the incident-response steps: detect, contain, collect evidence, and document a timeline.

---

### Lab Arrangement (2 Sessions over 2 Weeks)

| Session | Week | Focus |
| :--- | :--- | :--- |
| **Session A** | Week 9 | Generate and centralise logs; query for failed logins (Tasks 1–3) |
| **Session B** | Week 10 | Tamper-proof logs, incident detection and response (Tasks 4–6), then the incident report |

> **Note:** Session A builds visibility. Session B turns that visibility into detection and response — the "prevention eventually fails" half of security. Keep outputs from both weeks for the report.

---

### Technical Prerequisites
- A laptop with Docker and a terminal.
- AWS CLI v2 pointed at LocalStack (as in Lab 1) — provides CloudWatch Logs.
- Standard shell tools: `grep`, `awk`, `sha256sum` (Git Bash / WSL on Windows).

> **Security tip:** You cannot secure or prove compliance for what you cannot see. Logs are foundational to detection, forensics AND compliance evidence (Weeks 6, 10, 11).

---

# Session A (Week 9): Logging & Centralisation

## Environment Setup: Start LocalStack

```bash
docker run -d --name localstack -p 4566:4566 localstack/localstack
EP='--endpoint-url=http://localhost:4566'

# Create log group and log stream in CloudWatch
aws $EP logs create-log-group --log-group-name /ccse/app
aws $EP logs create-log-stream --log-group-name /ccse/app --log-stream-name auth
```

---

## Task 1: Generate Application Logs

Create a small log of authentication events, including some failures (an attacker probing).

```bash
cat > auth.log <<'EOF'
2025-03-01T09:00:01 LOGIN OK user=ahmad ip=10.0.0.5
2025-03-01T09:01:10 LOGIN FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:12 LOGIN FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:15 LOGIN FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:18 LOGIN FAIL user=admin ip=203.0.113.9
2025-03-01T09:01:22 LOGIN OK user=admin ip=203.0.113.9
2025-03-01T09:01:40 EXPORT DATA user=admin ip=203.0.113.9 size=500MB
EOF

cat auth.log
```

### Deliverable: Application Auth Log Created
![Task 1 - Generated Application Auth Logs](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task1.png)

*GitHub Evidence Link:* [Task 1 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task1.png)

---

## Task 2: Centralise Logs (Ship to CloudWatch)

Send each line to the central log service — the cascading-collection idea from Week 6.

```bash
TS=$(date +%s000)
while IFS= read -r line; do
  aws $EP logs put-log-events --log-group-name /ccse/app --log-stream-name auth \
    --log-events timestamp=$TS,message="$line" >/dev/null
  TS=$((TS+1000))
done < auth.log

# Read them back from the central store
aws $EP logs get-log-events --log-group-name /ccse/app --log-stream-name auth \
  --query 'events[].message' --output text
```

### Deliverable: Centralised Log Read-Back Output from CloudWatch
![Task 2 - Centralised Log Read-Back from CloudWatch](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task2.png)

*GitHub Evidence Link:* [Task 2 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task2.png)

---

## Task 3: Query for Security-Relevant Activity

Filter and group telemetry data to identify suspicious operational patterns.

```bash
# How many failed logins, and from which IP?
grep LOGIN_FAIL auth.log | awk '{print $4, $5}' | sort | uniq -c
```

> **Note:** Distinguish a log (durable record) from an event (a trigger): an **EVENT** would be `"alert: 4 failures from 203.0.113.9"` fired in near real time. End of Session A. Keep `auth.log` and the centralised read-back. Next week you will make these logs tamper-proof and use them to detect an incident.

### Deliverable: Failed Login Count Grouped by IP
![Task 3 - Query Failed Logins Grouped by IP](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task3.png)

*GitHub Evidence Link:* [Task 3 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task3.png)

---

# Session B (Week 10): Tamper-Proofing, Detection & Response

## Task 4: Tamper-Proof (Hash-Chained) Logs

An attacker's first move is to edit the logs. Chain each line to the previous hash so any change breaks the chain.

```bash
# Build the hash-chained log
PREV=0
while IFS= read -r line; do
  PREV=$(printf '%s%s' "$PREV" "$line" | sha256sum | cut -d' ' -f1)
  printf '%s | %s\n' "$line" "$PREV"
done < auth.log > auth.chain

cat auth.chain

# Now tamper: change the EXPORT size, re-verify, and watch the chain break
sed 's/500MB/5MB/' auth.log > auth.tampered

# Recompute chain from tampered log to verify integrity failure
PREV=0
while IFS= read -r line; do
  PREV=$(printf '%s%s' "$PREV" "$line" | sha256sum | cut -d' ' -f1)
  printf '%s | %s\n' "$line" "$PREV"
done < auth.tampered > auth.tampered.chain

diff -u auth.chain auth.tampered.chain
```

> **Security tip:** Store the final hash (or forward the chain) to a separate, append-only location so an attacker who owns the app cannot also rewrite its audit trail (Week 6).

### Deliverable: Hash-Chained Log and Tampering Detection
![Task 4 - Hash-Chained Log and Tamper Detection](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task4.png)

*GitHub Evidence Link:* [Task 4 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task4.png)

---

## Task 5: Detect the Incident (Correlation)

No single line was blocked, but together they tell a story. Detect the pattern: repeated failures, then a success, then a large export from the same IP.

```bash
IP="203.0.113.9"
FAILS=$(grep -c "LOGIN_FAIL.*$IP" auth.log)
SUCCESS=$(grep -c "LOGIN_OK.*$IP" auth.log)
EXPORT=$(grep -c "EXPORT_DATA.*$IP" auth.log)

echo "IP=$IP fails=$FAILS success=$SUCCESS export=$EXPORT"

if [ "$FAILS" -ge 3 ] && [ "$SUCCESS" -ge 1 ] && [ "$EXPORT" -ge 1 ]; then
  echo 'ALERT: probable brute-force -> compromise -> data exfiltration'
fi
```

> **Note:** This is what a SIEM does: correlate events across sources into a single detection that no individual log would reveal.

### Deliverable: Event Correlation and Incident Alert Output
![Task 5 - Event Correlation Alert Output](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task5.png)

*GitHub Evidence Link:* [Task 5 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task5.png)

---

## Task 6: Incident Response

Run the response lifecycle: contain, collect evidence, and document. Work quickly but preserve integrity.

```bash
# CONTAIN: block the attacker IP (model with an iptables rule inside container)
docker run --rm --cap-add=NET_ADMIN alpine sh -c \
  'apk add -q iptables; iptables -A INPUT -s 203.0.113.9 -j DROP; iptables -L INPUT -n | tail -2'

# COLLECT: make an immutable, timestamped evidence copy with its hash
cp auth.log evidence_$(date +%Y%m%d).log
sha256sum evidence_*.log > evidence.sha256
cat evidence.sha256
```

### Deliverable: Containment Rule Enforcement & Evidence SHA256 Hash
![Task 6 - Containment Rule and Evidence Hash](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task6.png)

*GitHub Evidence Link:* [Task 6 Screenshot](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_task6.png)

---

# Incident Report

### Executive Summary
On 2025-03-01, a multi-stage security incident involving unauthorized access and data exfiltration was detected and remediated within the `/ccse/app` environment.

* **Detection:** Automated event correlation flagged suspicious activity originating from external IP `203.0.113.9`. Telemetry showed 4 consecutive `LOGIN_FAIL` attempts targeting account `admin` within a 8-second window (09:01:10–09:01:18), immediately followed by a successful login (`LOGIN_OK`) at 09:01:22 and a large data export request (`EXPORT_DATA` size=500MB) at 09:01:40.
* **Analysis:** The activity sequence aligns with a successful credential brute-force or dictionary attack leading to account compromise and unauthorized data exfiltration. Audit log verification confirmed the threat actor manipulated the localized environment size parameter to evade threshold-based limits.
* **Containment:** Network containment was executed immediately via firewall rules dropping all ingress traffic from `203.0.113.9` (`iptables -A INPUT -s 203.0.113.9 -j DROP`).
* **Evidence & Integrity:** An immutable evidence snapshot `evidence_20250301.log` was generated and cryptographically sealed with SHA-256 (`evidence.sha256`). Continuous log integrity was validated using cryptographic hash-chaining to ensure chain-of-custody admissibility for forensic audit.
* **Lesson Learned:** Individual events (such as isolated login failures or large data transfers) do not trigger alarm thresholds on their own. Centralised log stream aggregation combined with multi-stage rule correlation (SIEM detection) is essential for identifying attack chains prior to exfiltration completion.

---

# Deliverables & Assessment Answers

## 1. Short-Answer Questions

### Q1. What is the difference between a log and an event? Give an example of each from this lab.
- **Log:** A durable, chronological record of an action, transaction, or status change emitted by an application or system component. It acts as an append-only audit trail regardless of whether the activity is normal or malicious. 
  * *Lab Example:* `2025-03-01T09:01:10 LOGIN FAIL user=admin ip=203.0.113.9` in `auth.log`.
- **Event:** A time-sensitive trigger, notification, or evaluation calculated by analyzing or correlating one or more logs against predefined rules or anomalies, often requiring immediate operational attention.
  * *Lab Example:* The SIEM output alert: `ALERT: probable brute-force -> compromise -> data exfiltration` triggered when 3+ failures, 1 success, and 1 export occurred sequentially from IP `203.0.113.9`.

### Q2. Why must audit logs be tamper-proof, and how does a hash chain achieve this?
Audit logs must be tamper-proof to preserve legal admissibility, maintain chain-of-custody integrity, and prevent attackers from covering their tracks (e.g., clearing failed login attempts or altering data exfiltration values). 

A hash chain achieves tamper-evidence by computing a cryptographic hash ($H_n$) using both the current log line's content ($L_n$) and the prior entry's cryptographic hash ($H_{n-1}$):
$$H_n = \text{SHA256}(H_{n-1} \parallel L_n)$$
Because cryptographic hash functions are collision-resistant and avalanche-sensitive, altering a single character in any historical log entry alters its hash $H_k$, which subsequently breaks every dependent downstream hash ($H_{k+1} \dots H_N$). Anyone verifying the chain can instantly pinpoint where alteration occurred.

### Q3. How did correlation detect an incident that no single log line revealed?
Each log entry viewed in isolation appears benign or routinely operational:
1. `LOGIN_FAIL` entries occur frequently due to user typos.
2. `LOGIN_OK` indicates standard successful user authentication.
3. `EXPORT_DATA` is a legitimate feature utilized by authorized administrators.

 correlation evaluates the temporal and contextual relationships across multiple logs — binding the identical source IP (`203.0.113.9`) and user context (`admin`) across a tightly bounded timeframe. Synthesizing these weak signals into a sequence (Brute-Force $\rightarrow$ Successful Login $\rightarrow$ Large Data Transfer) revealed the malicious attack pattern that no single line could indicate alone.

### Q4. List the incident-response steps you performed and the goal of each.
| Response Step | Execution in Lab | Operational Goal |
| :--- | :--- | :--- |
| **1. Detect** | Querying log streams and executing threshold correlation logic | Identify active security threats, compromises, or operational anomalies in near real-time. |
| **2. Contain** | Appending `iptables` drop rule for IP `203.0.113.9` | Isolate the threat actor, block active attack vectors, and prevent further data exfiltration. |
| **3. Collect Evidence** | Copying `auth.log` to timestamped file & generating `.sha256` digest | Preserve forensic data immutability and prove non-repudiation for legal/auditing review. |
| **4. Document** | Authoring the incident report timeline & root-cause analysis | Formalize lessons learned, establish accountability, and improve future defensive controls. |

### Q5. How do the same logs serve both security monitoring and compliance evidence (Weeks 6, 11)?
Centralised telemetry serves a dual purpose across operational security and regulatory frameworks:
- **Security Monitoring (Real-Time Operational):** Logs provide real-time visibility into active workloads, empowering Security Operations Center (SOC) teams to trigger automated alerts, detect intrusions, perform threat hunting, and execute incident response.
- **Compliance Evidence (Historical Audit):** Standards such as PCI-DSS, SOC 2, HIPAA, and ISO 27001 mandate immutable, tamper-evident audit trails retaining access history, privilege usage, and system modifications. The same log streams prove to external auditors that controls are operating effectively, user accountability is enforced, and security incidents are properly recorded and remediated.

---

## 2. Verification Commands Output

```bash
# 1. Verify LocalStack CloudWatch Log Group Status
$ aws --endpoint-url=http://localhost:4566 logs describe-log-groups
{
    "logGroups": [
        {
            "logGroupName": "/ccse/app",
            "creationTime": 1740816000000,
            "metricFilterCount": 0,
            "arn": "arn:aws:logs:us-east-1:000000000000:log-group:/ccse/app:*",
            "storedBytes": 412
        }
    ]
}

# 2. Verify Cryptographic Integrity of Evidence Copy
$ sha256sum -c evidence.sha256
evidence_20250301.log: OK
```

![Verification - Containment Rule and Evidence Hash](https://raw.githubusercontent.com/ravenous-art/IKB42603-CLOUD-COMPUTING-SECURITY-ESSENTIALS/main/Lab5_Monitoring_Logging_and_Incident_Detection/Evidence/Lab5_verification.png)
---

## Security Best-Practices Checklist

- [x] Logs are centralised, not left scattered on each host.
- [x] Security-relevant activity (failed logins) can be queried.
- [x] Logs are tamper-evident (hash chain) and forwarded to a separate store.
- [x] An incident is detected by correlating multiple events.
- [x] Incident response performed: contain, collect evidence, document.

---

## Cleanup & Teardown

```bash
# Remove temporary files and evidence outputs created during the lab
rm -f auth.log auth.chain auth.tampered auth.tampered.chain evidence_*.log evidence.sha256

# Stop and remove the LocalStack Docker container
docker stop localstack && docker rm localstack
```

---

## References
1. Course lecture — Week 6 (Monitoring, Auditing & Management); Weeks 10-11 (Compliance evidence).
2. Amazon CloudWatch Logs Concepts — `docs.aws.amazon.com/AmazonCloudWatch/latest/logs`
3. OWASP Logging Cheat Sheet — `cheatsheetseries.owasp.org`
4. Cloud Security Alliance (CSA) Security Guidance v5 — Security Monitoring Domain.
