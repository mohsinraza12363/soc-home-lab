# SOC Home Lab: Threat Detection & Automated Active Response

An enterprise-grade, multi-node SOC Home Lab designed for security telemetry monitoring, threat detection, log analysis, and automated incident containment using **Wazuh SIEM**, **Linux**, and **Windows** environments.

---

## 🛠️ Lab Architecture & Network Overview

```text
               +----------------------------------+
               |    Kali Linux (Attacker Host)    |
               |         IP: 192.168.0.113        |
               +-----------------+----------------+
                                 |
                      [ SSH Brute-Force Attack ]
                                 |
                                 v
               +-----------------+----------------+
               |    Ubuntu 24.04 Target Agent     |
               |         IP: 192.168.0.111        |
               +-----------------+----------------+
                                 |
                    [ Agent Telemetry / Syslog ]
                                 |
                                 v
               +-----------------+----------------+
               |          Wazuh Manager           |
               |      Correlation & Rule Engine   |
               +-----------------+----------------+
                                 |
                   [ Active Response Triggered ]
                                 |
                                 v
               +-----------------+----------------+
               |  firewall-drop (IP Containment)  |
               +----------------------------------+
