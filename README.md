# SOC Home Lab

Hands-on SOC home lab for security monitoring, threat detection, log analysis, and automated incident response using Wazuh across Windows and Linux environments.

## SSH Brute-Force Detection

This lab uses a custom Wazuh correlation rule to detect repeated SSH authentication failures originating from the same source IP.

### Detection Logic

- **Rule ID:** 127000
- **Alert Level:** 12
- **Frequency:** 3 matching authentication events
- **Timeframe:** 120 seconds
- **Source Correlation:** Same source IP
- **Event Category:** Authentication failure / brute-force

The rule correlates multiple authentication-related Wazuh events within a defined time window and generates a high-severity alert when repeated failures are detected from the same source IP.

The complete rule is available in [`detection-rules/ssh-brute-force.xml`](detection-rules/ssh-brute-force.xml).

## Automated Response

The lab uses Wazuh Active Response to automatically block the source IP after the SSH brute-force correlation rule is triggered.

### Response Workflow

1. SSH authentication failures are generated during the controlled lab simulation.
2. Wazuh correlates the authentication events using custom rule **127000**.
3. The correlation rule generates a **Level 12** brute-force alert.
4. Wazuh Active Response executes the `firewall-drop` response.
5. The detected source IP is blocked automatically.

This demonstrates an end-to-end SOC workflow from **detection and correlation to automated containment**.
