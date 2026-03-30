
# 🛡️ Wazuh Lab — SSH Brute Force Attack Detection

<div align="center">

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-00a7ff?style=for-the-badge&logo=wazuh&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)
![Tool](https://img.shields.io/badge/Tool-Hydra-red?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

**A hands-on cybersecurity lab demonstrating SSH brute force attack detection using Wazuh SIEM.**

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Lab Architecture](#-lab-architecture)
- [Prerequisites](#-prerequisites)
- [Environment Setup](#-environment-setup)
- [Attack Simulation](#-attack-simulation)
- [Detection & Alerts](#-detection--alerts)
- [Wazuh Rules Triggered](#-wazuh-rules-triggered)
- [Screenshots](#-screenshots)
- [Key Takeaways](#-key-takeaways)
- [References](#-references)

---

## 🔍 Overview

This lab simulates a **SSH brute force attack** on a target Linux machine and demonstrates how **Wazuh SIEM** detects, logs, and alerts on such activity in real time.

| Field | Details |
|---|---|
| **Lab Type** | Offensive + Defensive (Purple Team) |
| **Attack Technique** | SSH Brute Force (MITRE ATT&CK: T1110.001) |
| **Detection Tool** | Wazuh SIEM |
| **Attacker Machine** | <!-- e.g., Kali Linux 2024.x --> |
| **Target Machine** | <!-- e.g., Ubuntu 22.04 --> |
| **Network** | <!-- e.g., Host-Only / NAT / VirtualBox Internal --> |

---

## 🏗️ Lab Architecture

```
┌─────────────────────┐          SSH Brute Force          ┌─────────────────────┐
│                     │  ─────────────────────────────►   │                     │
│   Attacker Machine  │                                    │   Target Machine    │
│   (Kali Linux)      │  ◄─────────────────────────────   │   (Ubuntu / CentOS) │
│   IP: x.x.x.x       │        SSH Responses               │   IP: x.x.x.x       │
└─────────────────────┘                                    └──────────┬──────────┘
                                                                      │
                                                                      │ Log forwarding
                                                                      │ (Wazuh Agent)
                                                                      ▼
                                                           ┌─────────────────────┐
                                                           │                     │
                                                           │   Wazuh Manager     │
                                                           │   (SIEM / Dashboard)│
                                                           │   IP: x.x.x.x       │
                                                           └─────────────────────┘
```

---

## ✅ Prerequisites

- [ ] VirtualBox / VMware installed
- [ ] Wazuh Manager + Dashboard deployed (OVA or Docker)
- [ ] Wazuh Agent installed on target machine
- [ ] Kali Linux (attacker VM)
- [ ] Target Linux machine (SSH service running)
- [ ] Hydra / Medusa installed on attacker
- [ ] Basic familiarity with Linux CLI

---

## ⚙️ Environment Setup

### 1. Wazuh Manager
```bash
# Verify Wazuh manager is running
sudo systemctl status wazuh-manager

# Check agent connection
sudo /var/ossec/bin/agent_control -l
```

### 2. Target Machine — Install & Register Wazuh Agent
```bash
# Install Wazuh agent (Ubuntu/Debian)
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt-get update && sudo apt-get install wazuh-agent

# Configure manager IP
sudo sed -i 's/MANAGER_IP/<YOUR_WAZUH_MANAGER_IP>/' /var/ossec/etc/ossec.conf

# Start agent
sudo systemctl enable wazuh-agent && sudo systemctl start wazuh-agent
```

### 3. Verify SSH is Running on Target
```bash
sudo systemctl status ssh
# If not running:
sudo systemctl enable ssh && sudo systemctl start ssh
```

---

## ⚔️ Attack Simulation

> ⚠️ **Disclaimer:** This attack was performed in an **isolated lab environment**. Never perform brute force attacks on systems you do not own or have explicit permission to test.

### 1. Create a Wordlist (or use existing)
```bash
# Use RockYou or generate a small custom list
echo -e "password\n123456\nadmin\nroot\npassword123\nletmein" > wordlist.txt
```

### 2. Run SSH Brute Force with Hydra
```bash
hydra -l <TARGET_USERNAME> -P wordlist.txt ssh://<TARGET_IP> -t 4 -V
```

| Flag | Description |
|---|---|
| `-l` | Target username |
| `-P` | Password wordlist |
| `ssh://` | Protocol and target IP |
| `-t 4` | Number of parallel threads |
| `-V` | Verbose output |

### 3. Expected Hydra Output
```
[DATA] attacking ssh://<TARGET_IP>:22/
[ATTEMPT] target <TARGET_IP> - login "root" - pass "password" - 1 of 6
[ATTEMPT] target <TARGET_IP> - login "root" - pass "123456" - 2 of 6
...
[22][ssh] host: <TARGET_IP>   login: <USER>   password: <PASS>  ← if successful
```

---

## 🔔 Detection & Alerts

### Wazuh Dashboard — Alerts Generated

After running the attack, navigate to **Wazuh Dashboard → Security Events** to observe:

- Multiple **authentication failure** alerts from `/var/log/auth.log`
- Active Response triggers (if configured)
- Spike in events on the **Security Events** timeline

### Auth Log on Target (for reference)
```bash
sudo tail -f /var/log/auth.log
```
```
sshd[XXXX]: Failed password for root from <ATTACKER_IP> port XXXXX ssh2
sshd[XXXX]: Failed password for root from <ATTACKER_IP> port XXXXX ssh2
sshd[XXXX]: Failed password for root from <ATTACKER_IP> port XXXXX ssh2
...
sshd[XXXX]: message repeated N times
```

---

## 📜 Wazuh Rules Triggered

| Rule ID | Description | Level |
|---|---|---|
| **5710** | sshd: Attempt to login using a non-existent user | 5 |
| **5711** | sshd: Attempt to login using a denied user | 5 |
| **5712** | sshd: SSHD brute force trying to get access to the system | **10** |
| **5760** | sshd: insecure connection attempt | 6 |

> Rule `5712` is the key brute force detection rule — it fires after multiple failed login attempts within a short window.

---

## 📸 Screenshots

> Replace the placeholders below with your actual screenshots.

### Wazuh Dashboard — Alert Overview
![Wazuh Alerts Overview](./screenshots/wazuh-alerts-overview.png)

### Security Events Timeline
![Security Events Timeline](./screenshots/security-events-timeline.png)

### Brute Force Rule Triggered (Rule 5712)
![Rule 5712 Triggered](./screenshots/rule-5712-triggered.png)

### Hydra Attack Output
![Hydra Output](./screenshots/hydra-output.png)

---

## 💡 Key Takeaways

1. **Wazuh detects SSH brute force in real time** using built-in rules that analyze `/var/log/auth.log`.
2. **Rule 5712** triggers after a configurable threshold of failed logins from the same source IP.
3. **Active Response** can be configured to automatically block the attacker's IP using `firewall-drop`.
4. **Log centralization** via Wazuh agents gives SOC teams a single pane of glass for threat detection.
5. **Mitigations** include: disabling password auth (use SSH keys), fail2ban, port knocking, and MFA.

---

## 🔐 Recommended Mitigations

```bash
# 1. Disable password authentication in SSH
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo systemctl restart ssh

# 2. Allow only specific users
echo "AllowUsers <your_user>" | sudo tee -a /etc/ssh/sshd_config

# 3. Change default SSH port (optional, security through obscurity)
sudo sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config
```

---

## 📚 References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [MITRE ATT&CK T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)
- [Hydra – THC Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- [Wazuh Active Response](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html)

---

<div align="center">

Made with 🔒 for learning purposes | **Lab by:** Your Name  
⭐ Star this repo if you found it helpful!

</div>
