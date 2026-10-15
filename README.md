# SIEM Detection Lab (Wazuh + Sysmon)

A home SOC lab where I simulate attacks on a Windows VM and detect them with custom Wazuh rules mapped to MITRE ATT&CK.

> Based on the open-source [siem-detection-lab](https://github.com/REPLACE-WITH-ORIGINAL-REPO-URL) (MIT). I built and ran the lab myself; results, screenshots and write-ups below are from my own environment.

## Architecture

```
Windows 10 VM (VirtualBox)            Laptop host (Docker)
 Sysmon + Security logs  ──────────►  Wazuh Manager  ──► custom rules ──► alerts.json
 Wazuh agent (1514/tcp)
```

- **Victim:** Windows 10 VM, Sysmon (SwiftOnSecurity config), Windows audit policies
- **SIEM:** Wazuh 4.9.0 manager in Docker (manager only, no dashboard, to fit an 8 GB laptop); alerts read from `alerts.json`
- **Attacks:** manual one-liners and Atomic Red Team, run only inside the VM

## Detections I tested

| Detection | ATT&CK | Rule ID | Result |
|---|---|---|---|
| PowerShell encoded command | T1059.001 | 100101 | [ ] fired / [ ] not yet |
| New local admin account | T1136.001 | 100108 | [ ] fired / [ ] not yet |
| Scheduled task created | T1053.005 | 100109 | [ ] fired / [ ] not yet |

_Tick only what you actually ran. Add more rows as you test them._

## How I ran it

1. Started the Wazuh manager: `docker compose up -d wazuh.manager`
2. Loaded `wazuh/local_rules.xml` into the manager and restarted it
3. Installed Sysmon, enabled audit policies, enrolled the Wazuh agent on the VM
4. Ran an attack in the VM, then checked the alert:
   ```bash
   docker exec single-node-wazuh.manager-1 grep -F '"id":"100101"' /var/ossec/logs/alerts/alerts.json
   ```

## Example: PowerShell encoded command (T1059.001)

- **Simulated:** `powershell -enc VwByAGkAdABlAC0ASABvAHMAdAAgAGgAaQA=`
- **Telemetry:** Sysmon Event ID 1 (process creation) with the encoded command line
- **Alert:** rule 100101, level 12
- **Screenshot:** `screenshots/t1059-alert.png`
- **Triage notes:** [what the parent process was, what the decoded command did, whether it is a false-positive risk]

## Problems I hit and fixed

- [Write your own here, e.g. VM on NAT couldn't reach the manager; firewall rule for ports 1514/1515 was missing; used `10.0.2.2` as the manager address]

## Repo layout

```
wazuh/local_rules.xml   custom Wazuh rules
rules/                  Sigma versions of the detections
docs/                   setup and simulation notes
screenshots/            my alert screenshots
LICENSE                 MIT (original copyright kept)
```

## Skills

SIEM (Wazuh), Sysmon, Windows event logs, MITRE ATT&CK, detection rules (Sigma / Wazuh), attack simulation, Docker, VirtualBox, alert triage
