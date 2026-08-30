# SSH Brute-Force Detection & Automated Response

## Objective

Detect repeated SSH authentication failures from the same source IP and automatically block the detected attacker IP using Wazuh.

## Lab Workflow

The activity was performed in an isolated security lab environment using Wazuh, Linux systems, and a Kali Linux testing system.

```text
SSH Authentication Attempts
            ↓
      Authentication Logs
            ↓
       Wazuh Manager
            ↓
 Custom Correlation Rule 127000
            ↓
     Level 12 Alert
            ↓
    Wazuh Active Response
            ↓
       firewall-drop
            ↓
       IP Containment
