# SSH Brute-Force Detection, Investigation, and Automated Response

## Objective
Simulate an automated brute-force attack against an SSH service running on a Linux endpoint, detect the attack using a custom Wazuh correlation rule, analyze the event telemetry, and automatically contain the threat using Wazuh Active Response (`firewall-drop`).

## Lab Architecture & Environment
- **Attacker System:** Kali Linux (`192.168.0.113`)
- **Target Endpoint:** Ubuntu-24.04 (`192.168.0.111`, Agent ID `002`)
- **SIEM / Manager:** Wazuh Manager (`DESKTOP-BAHFM2F`)
- **Tools Used:** Hydra, Wazuh SIEM, Journald, iptables

---

## Investigation Workflow

### 1. Attacker Setup & Interface Verification
The attacker machine verifies its IP address and network configuration prior to launching the security simulation.
| Attacker Host Details | Target Endpoint Details |
| :---: | :---: |
| ![Attacker Details](../screenshots/1%29%20attacker-details.png) | ![Endpoint Details](../screenshots/2%29%20endpoint-details.png) |

### 2. Attack Simulation (Hydra Execution)
Using Hydra from the Kali Linux system, a dictionary-based brute-force attack is executed against the target SSH service (`192.168.0.111`).
![Attack Initialization](../screenshots/3%29%20attack-initialization.png)

### 3. Endpoint Log Collection (`journald` / Auth Logs)
The target Ubuntu endpoint records repeated authentication failures (`pam_unix(sshd:auth): authentication failure`) as the attack progresses.
![Endpoint Auth Logs](../screenshots/4%29%20endpoint-auth_logs.png)

### 4. Wazuh SIEM Detection & Rule Correlation
Wazuh ingests the authentication logs from the endpoint agent. Individual failed login attempts generate low-severity alerts, while custom correlation rule `127000` groups these events to trigger a high-severity incident.
- **Rule ID:** `127000`
- **Alert Level:** 12 (Critical)
- **Threshold:** 3 matching events within 120 seconds from the same source IP (`same_source_ip`)

![Wazuh Dashboard Alerts](../screenshots/5%29%20wazuh-dashboard-alerts.png)

#### Detailed Event Inspection (JSON)
| Single Event View (Auth Failure) | Correlated Alert View (Rule 127000) |
| :---: | :---: |
| ![Attack Details](../screenshots/6%29%20attack-details.png) | ![Rule Triggered](../screenshots/7%29%20rule-triggered.jpg) |

### 5. Automated Containment (Wazuh Active Response)
Upon triggering Rule `127000`, Wazuh Active Response invokes the `firewall-drop` command to automatically block the attacker's IP (`192.168.0.113`). Verification from the Kali machine shows 100% packet loss during ping tests, followed by the automatic unblock event after the timeout window.
| Host Blocked Proof (Ping Failure) | Host Unblocked Event (Post-Timeout) |
| :---: | :---: |
| ![Host Blocked Proof](../screenshots/8%29%20host-blocked-proof.png) | ![Host Unblocked](../screenshots/9%29%20host_Unblocked.png) |

---

## Detection Rule Configuration (`local_rules.xml`)
```xml
<group name="Brute-Force-Correlation,">

  <rule id="127000" level="12" frequency="3" timeframe="120" ignore="300">
    <if_matched_sid>5763</if_matched_sid>
    <if_matched_sid>5710</if_matched_sid>
    <if_matched_sid>5760</if_matched_sid>
    <if_matched_sid>5716</if_matched_sid>
    <if_matched_sid>5503</if_matched_sid>
    <same_source_ip />
    <description>Correlation Brute Force SSH Attack Detected</description>
    <group>authentication_failed,brute_force,</group>
  </rule>

</group>
```
---

## MITRE ATT&CK Mapping

This simulated attack and detection lifecycle maps to the following enterprise tactics and techniques:

- **Tactic:** Credential Access ([TA0006](https://attack.mitre.org/tactics/TA0006/))
- **Technique:** Brute Force ([T1110](https://attack.mitre.org/techniques/T1110/))
- **Sub-Technique:** Password Guessing ([T1110.001](https://attack.mitre.org/techniques/T1110/001/))
- **Defense Evasion / Mitigation:** Network Content Filtering / Host Firewall Blocking (Active Response)

---

## Lessons Learned & SOC Analyst Takeaways

1. **Alert Fatigue Reduction via Correlation:** 
   Single authentication failure logs (`Rule 5760`) are extremely noisy in production environments. Relying on default single-event rules can flood a Tier 1 queue with false positives. Implementing custom correlation rule `127000` (requiring multiple failures from the *same source IP* within a tight timeframe) ensures analysts only investigate high-confidence threats.
   
2. **Speed of Containment:** 
   Manual triage introduces dwell time. Integrating Wazuh Active Response (`firewall-drop`) reduced containment time to milliseconds, automatically neutralizing the threat before manual intervention was possible.
   
3. **Threshold Tuning Considerations:** 
   Setting the frequency too low risks blocking legitimate users who mistype their password multiple times. Tuning the threshold ($\ge 3$ attempts in $120$ seconds) balances strict security enforcement with user accessibility.
