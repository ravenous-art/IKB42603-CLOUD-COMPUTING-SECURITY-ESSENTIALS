cat <<'EOF' > IKB42603_Lab5_Monitoring_Logging_and_Incident_Detection.md
# IKB42603 Cloud Computing Security Essentials
## LAB 5 WEEKS 9-10: Monitoring, Logging & Incident Detection
**Centralised logging, tamper-proof logs, threat detection and incident response — Docker & LocalStack**

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

> **Note:** Session A builds visibility. Session B turns that visibility into detection and response — the 'prevention eventually fails' half of security. Keep outputs from both weeks for the report.

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
aws $EP logs create-log-group --log-group-name /ccse/app
aws $EP logs create-log-stream --log-group-name /ccse/app --log-stream-name auth
