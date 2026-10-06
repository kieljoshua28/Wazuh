# Incident Report: SSH Brute-Force Attack Detection

## 1. Executive Summary

On October 6, 2026, a simulated SSH brute-force attack was launched against a monitored Linux server (Wazuh SIEM host) as part of a controlled home lab exercise. The attack targeted the `root` account using automated password-guessing via Hydra and the rockyou.txt wordlist. The Wazuh SIEM successfully detected and logged the activity in real time, generating over 3,000 security alerts within minutes. This report documents the attack simulation, detection process, evidence collected, and recommended remediation steps.

| Field | Detail |
|---|---|
| **Report Date** | October 6, 2026 |
| **Analyst** | Kiel Joshua P. Lozada |
| **Environment** | Isolated home lab (VirtualBox/VMware, NAT network) |
| **Severity** | Medium (simulated; no real compromise) |
| **Status** | Detected, Investigated, Closed |

---

## 2. Scope and Environment

This was a controlled, self-contained lab exercise. No production systems, third-party networks, or unauthorized targets were involved.

| Role | System | IP Address | Purpose |
|---|---|---|---|
| Attacker | Kali Linux | 192.168.0.187 | Simulated threat actor |
| Target / Monitored Host | Wazuh Server (Ubuntu-based) | 192.168.0.132 | SIEM + attack target |
| Monitoring Tool | Wazuh 4.14.8 | 192.168.0.132 | Log collection, detection, alerting |

**Tools used:**
- **Hydra v9.6** — automated login brute-forcing
- **rockyou.txt** — ~14 million-entry password wordlist
- **Wazuh** — SIEM for log analysis and alert generation

---

## 3. Detection and Evidence

### 3.1 Alert Volume (Before vs. After)

**Before attack (baseline, last 24 hours):**
- Critical: 0
- High: 0
- Medium: 2
- Low: 11

**After attack:**
- Critical: 0
- High: 0
- Medium: **419**
- Low: **2,624**

This represents a clear, measurable spike directly correlated with the attack window, confirming the SIEM's detection capability for brute-force authentication activity.

### 3.2 Sample Alert — Full Detail

A representative alert was selected from the Wazuh "Threat Hunting" module and examined in detail:
<img width="1907" height="868" alt="Screenshot 2026-10-06 203614" src="https://github.com/user-attachments/assets/8daa6c96-4629-4ee0-8277-fbd5a45c3470" />


Two things stand out here, this is a **PAM-level escalation alert** distinct from the individual "Failed password" events in section 3.2: it fires specifically once a source has missed the password multiple times in a session, which is a stronger brute-force indicator than a single failed login. Second, Wazuh's default ruleset **automatically maps this activity to MITRE ATT&CK (T1110 – Brute Force, Credential Access)** and to multiple compliance frameworks (NIST 800-53, GDPR, HIPAA), with no custom rule authoring required. This confirms the SIEM's out-of-the-box detection logic is both accurate and audit-ready.

---

## 4. Analysis

### 4.1 Attack Technique

The activity observed is consistent with a **brute-force credential attack** against SSH, specifically a dictionary-based attack using a large, publicly known password wordlist (rockyou.txt). The attacker targeted a high-value default account (`root`) rather than attempting account enumeration first, a common but noisy technique.

### 4.2 MITRE ATT&CK Mapping

Wazuh's ruleset auto-tagged the activity as **T1110 (Brute Force) / Credential Access** (see section 4.4). This aligns with the expected classification for this attack pattern:

| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Brute Force: Password Guessing | T1110.001 (sub-technique of T1110) |
| Initial Access | Valid Accounts (attempted) | T1078 |

### 4.3 Indicators of Compromise (IOCs)

- **Source IP:** 192.168.0.187
- **Target account:** root
- **Target service:** SSH (port 22)
- **Pattern:** High-frequency repeated authentication failures from a single source IP within a short time window (237+ attempts/min sustained)

### 4.4 Outcome

The attack did **not** result in a successful authentication (no valid credentials were found in the wordlist within the test window). The exercise was stopped deliberately once sufficient detection evidence was gathered, rather than run to completion, since a full dictionary run against ~14 million entries was estimated at 1,000+ hours and success was not the goal of this exercise.

---

## 5. Recommendations

Based on this simulated attack, the following hardening measures are recommended for any internet-facing or production SSH service:

1. **Disable root SSH login** (`PermitRootLogin no` in `sshd_config`) — forces attackers to also guess a valid username, significantly increasing attack difficulty.
2. **Enforce key-based authentication** instead of passwords, eliminating brute-force risk entirely.
3. **Implement account lockout / rate limiting** (e.g., `fail2ban` or `sshguard`) to automatically block IPs after repeated failed attempts.
4. **Change the default SSH port** as a minor deterrent against automated scanning (defense-in-depth, not a primary control).
5. **Enable MFA** for SSH access where supported.
6. **Configure active alerting** (email/Slack webhook) in Wazuh for authentication failure thresholds, rather than relying on manual dashboard review.
7. **Tune alert correlation rules** to automatically escalate severity when failed-login volume from a single IP exceeds a defined threshold in a short time window (brute-force-specific detection rule).

---

## 6. Lessons Learned

- Confirmed that Wazuh's default ruleset detects SSH authentication failures out of the box, including automatic MITRE ATT&CK and compliance-framework mapping (NIST 800-53, GDPR, HIPAA), with no custom rule configuration required.
- Observed the direct relationship between raw log volume and alert volume, reinforcing the importance of log source coverage in SIEM deployments.
- Learned to move from an unfiltered alert flood to a scoped, targeted query (log source + severity band + keyword) to isolate relevant activity, a core part of alert triage.
- Practiced the full analyst workflow: hypothesis (simulate known attack) → detection (SIEM alerting) → triage (drill into alert fields, filter the noise) → documentation (this report).

---


