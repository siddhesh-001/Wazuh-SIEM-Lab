# 🛡️ Wazuh Lab 1 — SSH Brute Force Attack Detection

<div align="center">

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-00a7ff?style=for-the-badge&logo=wazuh&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-Victim-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Kali Purple](https://img.shields.io/badge/Kali_Purple-Attacker_+_SIEM-6C3483?style=for-the-badge&logo=kalilinux&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

**A hands-on cybersecurity lab demonstrating SSH authentication failure detection using Wazuh SIEM.**

> ⚠️ **Disclaimer:** This was performed in a **controlled, isolated lab environment**. Never perform attacks on systems you don't own or have explicit written permission to test.

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

In this lab, **Kali Purple** acts as both the **attacker** and the **Wazuh SIEM host**, while a **Kali Linux** machine serves as the **victim endpoint** with a Wazuh agent installed. The attack involves attempting SSH login with a non-existent user and wrong passwords — all captured and alerted on by Wazuh in real time.

| Field | Details |
|---|---|
| **Lab Type** | Offensive + Defensive (Purple Team) |
| **Attack Technique** | SSH Failed Authentication — Invalid User (MITRE ATT&CK: T1110.001) |
| **Detection Tool** | Wazuh SIEM (hosted on Kali Purple) |
| **Attacker Machine** | Kali Purple (also hosts Wazuh Stack) |
| **Victim Machine** | Kali Linux (Wazuh Agent installed) |
| **Attack Method** | Manual SSH login attempts with fake user + wrong passwords |
| **Network** | Local network / Host-Only |

---

## 🏗️ Lab Architecture

```
┌──────────────────────────────────┐
│         Kali Purple              │
│  ┌────────────────────────────┐  │
│  │  Wazuh Stack               │  │         SSH Attempts
│  │  (Manager + Dashboard)     │  │   ssh fakeuser@<KaliLinux-IP>
│  └────────────────────────────┘  │ ─────────────────────────────►  ┌──────────────────────┐
│                                  │   Wrong passwords entered       │     Kali Linux       │
│  [ Attacker Role ]               │ ◄─────────────────────────────  │  (Victim Endpoint)   │
│  ssh fakeuser@<KaliLinux-IP>     │   Auth Failed responses         │  Wazuh Agent running │
└──────────────────────────────────┘                                 └───────────┬──────────┘
              ▲                                                                  │
              │           Auth failure logs forwarded via Wazuh Agent            │
              └──────────────────────────────────────────────────────────────────┘
                                Alerts visible on Wazuh Dashboard
```

---

## ✅ Prerequisites

- [ ] Kali Purple VM with Wazuh Stack (Manager + Dashboard) running
- [ ] Kali Linux VM with Wazuh Agent installed and enrolled to Kali Purple
- [ ] Both VMs on the same network (can ping each other)
- [ ] SSH service running on Kali Linux (victim)

---

## ⚙️ Environment Setup

### 1. Verify Wazuh Stack is Running (Kali Purple)
```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard
sudo systemctl status wazuh-indexer
```

### 2. Verify Wazuh Agent is Running & Connected (Kali Linux — Victim)
```bash
sudo systemctl status wazuh-agent
```
> On Kali Purple dashboard, go to **Agents** tab and confirm the Kali Linux agent shows as **Active**.

### 3. Verify SSH is Running on Victim (Kali Linux)
```bash
sudo systemctl status ssh
# If not running:
sudo systemctl enable ssh && sudo systemctl start ssh
```

### 4. Get the Victim's IP Address (Kali Linux)
```bash
ip a
# Note down the IP — e.g., 192.168.x.x
```

---

## ⚔️ Attack Simulation

> ⚠️ **Disclaimer:** This was performed in a **controlled, isolated lab environment**. Never perform attacks on systems you don't own or have explicit written permission to test.

### From Kali Purple (Attacker), attempt SSH with a fake user:

```bash
ssh fakeuser@<KALI_LINUX_IP>
```

When prompted for a password, **enter wrong passwords** multiple times:

```
fakeuser@<KALI_LINUX_IP>'s password: wrongpassword1   ← Press Enter
Permission denied, please try again.
fakeuser@<KALI_LINUX_IP>'s password: wrongpassword2   ← Press Enter
Permission denied, please try again.
fakeuser@<KALI_LINUX_IP>'s password: wrongpassword3   ← Press Enter
Permission denied (publickey,password).
```

> SSH disconnects after 3 failed attempts by default. Repeat the command a few times to generate more alerts.

### What's happening on the Victim (Kali Linux):
```bash
# Auth failures are logged here in real time:
sudo tail -f /var/log/auth.log
```
```
sshd[XXXX]: Invalid user fakeuser from <KALI_PURPLE_IP> port XXXXX
sshd[XXXX]: Failed password for invalid user fakeuser from <KALI_PURPLE_IP> port XXXXX ssh2
sshd[XXXX]: Connection closed by invalid user fakeuser <KALI_PURPLE_IP> port XXXXX [preauth]
```
The **Wazuh Agent** on Kali Linux reads these logs in real time and forwards them to the **Wazuh Manager on Kali Purple**.

---

## 🔔 Detection & Alerts

### Viewing Alerts on Wazuh Dashboard (Kali Purple)

1. Open browser → `https://<KALI_PURPLE_IP>` (Wazuh Dashboard)
2. Navigate to **Security Events** (or **Threat Hunting**)
3. Filter by the **Kali Linux agent**
4. Observe alerts generated for the SSH failures

### Alerts You Should See:

| Alert Description | Trigger |
|---|---|
| `sshd: Invalid user or authentication failure` | Logging in with `fakeuser` (non-existent user) |
| `sshd: authentication failed` | Wrong passwords entered |
| `sshd: Attempt to login using a non-existent user` | Rule 5710 fires |

---

## 📜 Wazuh Rules Triggered

| Rule ID | Description | Level |
|---|---|---|
| **5710** | sshd: Attempt to login using a non-existent user | 5 |
| **5711** | sshd: Attempt to login using a denied user | 5 |
| **5712** | sshd: SSHD brute force trying to get access to the system | **10** |

> **Rule 5710** fires immediately for `fakeuser` since the user doesn't exist on the system.  
> **Rule 5712** fires after repeated failures from the same IP within a short timeframe.

---

## 📸 Screenshots

> Add your screenshots inside a `/screenshots` folder in this repo and update paths below.

### 1. Wazuh Dashboard — Alerts Overview
![Wazuh Alerts Overview](./screenshots/wazuh-alerts-overview.png)

### 2. Auth Failure Alert Detail (Rule 5710 / 5712)
![Auth Failure Alert](./screenshots/auth-failure-alert.png)

### 3. SSH Terminal Output from Kali Purple (Attacker Side)
![SSH Failed Attempt](./screenshots/ssh-failed-attempt.png)

### 4. auth.log on Kali Linux (Victim Side)
![Auth Log](./screenshots/auth-log-victim.png)

---

## 💡 Key Takeaways

1. **Wazuh detects invalid SSH users instantly** — even a single attempt with a non-existent username triggers Rule 5710.
2. **Repeated failures escalate** to Rule 5712 (brute force detection), which has a higher severity level (10).
3. **Kali Purple is a powerful purple team platform** — it can run both offensive tools and a full SIEM stack simultaneously, making it ideal for solo labs.
4. **Log forwarding via Wazuh Agent is real-time** — alerts appear on the dashboard within seconds of the attack.
5. This lab demonstrates the core SOC workflow: **Attack → Log Generation → Agent Forwarding → SIEM Alerting**.

---

## 🔐 Recommended Mitigations

```bash
# 1. Disable SSH password authentication — use key-based auth only
sudo nano /etc/ssh/sshd_config
# Set: PasswordAuthentication no

# 2. Restrict SSH to specific users only
echo "AllowUsers your_actual_user" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart ssh

# 3. Install fail2ban to auto-ban IPs after N failed attempts
sudo apt install fail2ban -y
sudo systemctl enable fail2ban --now
```

---

## 📚 References

- [Wazuh Documentation](https://documentation.wazuh.com/)
- [Kali Purple Overview](https://www.kali.org/blog/kali-linux-2023-1-release/#kali-purple)
- [MITRE ATT&CK T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [Wazuh Brute Force Detection PoC Guide](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-brute-force-attack.html)

---

<div align="center">

|&nbsp; **Lab by:** Siddhesh Chute  
⭐ Star this repo if you found it helpful!

</div>
