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
| ![Attacker Details](screenshots/1)%20attacker-details.png | ![Endpoint Details](screenshots/2)%20endpoint-details.png |

### 2. Attack Simulation (Hydra Execution)
Using Hydra from the Kali Linux system, a dictionary-based brute-force attack is executed against the target SSH service (`192.168.0.111`).
![Attack Initialization](screenshots/3)%20attack-initialization.png)

### 3. Endpoint Log Collection (`journald` / Auth Logs)
The target Ubuntu endpoint records repeated authentication failures (`pam_unix(sshd:auth): authentication failure`) as the attack progresses.
![Endpoint Auth Logs](screenshots/4)%20endpoint-auth_logs.png

### 4. Wazuh SIEM Detection & Rule Correlation
Wazuh ingests the authentication logs from the endpoint agent. Individual failed login attempts generate low-severity alerts, while custom correlation rule `127000` groups these events to trigger a high-severity incident.
- **Rule ID:** `127000`
- **Alert Level:** 12 (Critical)
- **Threshold:** 3 matching events within 120 seconds from the same source IP (`same_source_ip`)

![Wazuh Dashboard Alerts](screenshots/5)%20wazuh-dashboard-alerts.png

#### Detailed Event Inspection (JSON)
| Single Event View (Auth Failure) | Correlated Alert View (Rule 127000) |
| :---: | :---: |
| ![Attack Details](screenshots/6)%20attack-details.png | ![Rule Triggered](screenshots/7)%20rule-triggered.jpg |

### 5. Automated Containment (Wazuh Active Response)
Upon triggering Rule `127000`, Wazuh Active Response invokes the `firewall-drop` command to automatically block the attacker's IP (`192.168.0.113`). Verification from the Kali machine shows 100% packet loss during ping tests, followed by the automatic unblock event after the timeout window.
| Host Blocked Proof (Ping Failure) | Host Unblocked Event (Post-Timeout) |
| :---: | :---: |
| ![Host Blocked Proof](screenshots/8)%20host-blocked-proof.png | ![Host Unblocked](screenshots/9)%20host_Unblocked.png |

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
