
# 🛡️ Wazuh Lab 4 — Active Response (Auto-Blocking SSH Brute Force)

<div align="center">

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-00a7ff?style=for-the-badge&logo=wazuh&logoColor=white)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-Victim-557C94?style=for-the-badge&logo=kalilinux&logoColor=white)
![Kali Purple](https://img.shields.io/badge/Kali_Purple-Attacker_+_SIEM-6C3483?style=for-the-badge&logo=kalilinux&logoColor=white)
![Tool](https://img.shields.io/badge/Tool-Hydra-red?style=for-the-badge)
![Active Response](https://img.shields.io/badge/Feature-Active_Response-orange?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

**Wazuh detects an SSH brute force attack and automatically blocks the attacker's IP — no human needed.**

</div>

---

##  Table of Contents

- [Overview](#-overview)
- [Lab Architecture](#-lab-architecture)
- [How Active Response Works](#-how-active-response-works)
- [Prerequisites](#-prerequisites)
- [Configuration](#-configuration)
- [Attack Simulation](#-attack-simulation)
- [Active Response in Action](#-active-response-in-action)
- [Verification](#-verification)
- [Wazuh Rules & Response Triggered](#-wazuh-rules--response-triggered)
- [Screenshots](#-screenshots)
- [Key Takeaways](#-key-takeaways)
- [References](#-references)

---

##  Overview

This lab builds on Lab 1 by enabling **Wazuh Active Response** — an automated defense mechanism that reacts to threats without manual intervention. When Hydra (running on Kali Purple) launches an SSH brute force attack against Kali Linux, Wazuh detects it, fires **Rule 5763**, and immediately instructs the Wazuh Agent to **drop all packets from the attacker's IP** using `iptables` / `firewall-cmd`. Hydra gets blocked mid-attack.

| Field | Details |
|---|---|
| **Lab Type** | Defensive Automation (Purple Team) |
| **Attack Tool** | Hydra (from Kali Purple) |
| **Detection Rule** | Rule 5763 — SSH brute force detected |
| **Auto-Defense** | Wazuh Active Response → `firewall-drop` script |
| **Block Method** | `iptables` / `firewall-cmd` on Kali Linux (victim) |
| **Attacker Machine** | Kali Purple (also hosts Wazuh Stack) |
| **Victim Machine** | Kali Linux (Wazuh Agent installed) |

---

##  Lab Architecture

```
  KALI PURPLE (Attacker + Wazuh Manager)
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  │  [1] Hydra launches SSH brute force             │
  │       hydra -l root -P wordlist.txt             │
  │       ssh://<KaliLinux-IP>                      │
  │                          │                      │
  │                          │ SSH login attempts   │
  │                          ▼                      │
  │                   ┌─────────────────────────────────────────────┐
  │                   │          KALI LINUX (Victim)                │
  │                   │                                             │
  │                   │  [2] Failed logins → written to auth.log   │
  │                   │                                             │
  │                   │  [3] Wazuh Agent reads auth.log            │
  │                   │       → forwards events to Manager          │
  │                   └──────────────────┬──────────────────────────┘
  │                                      │ Events forwarded
  │  [4] Wazuh Manager receives events   │
  │       → Rule 5763 fires              ◄┘
  │       (brute force threshold hit)    │
  │                                      │
  │  [5] Manager sends Active Response   │
  │       command to Agent:              │
  │       "Block <KaliPurple-IP>"        │
  │                          │           │
  │                          ▼           │
  │                   ┌──────────────────▼──────────────────────────┐
  │                   │          KALI LINUX (Victim)                │
  │                   │                                             │
  │                   │  [6] Agent runs firewall-drop script        │
  │                   │  iptables -I INPUT -s <KaliPurple-IP>       │
  │                   │           -j DROP                           │
  │                   │                                             │
  │                   │  [7] All packets from Kali Purple DROPPED   │
  │                   └─────────────────────────────────────────────┘
  │
  │  [8] Hydra gets blocked mid-attack 
  │       "Connection refused / timed out"
  │
  └─────────────────────────────────────────────────┘
```

---

##  How Active Response Works

```
Hydra Attack
    │
    ▼
auth.log on victim
    │  (Wazuh Agent tails this file)
    ▼
Wazuh Manager
    │  Rule 5763 threshold hit
    ▼
Active Response triggered
    │  Manager pushes command to Agent
    ▼
firewall-drop script runs on victim
    │  iptables DROP rule inserted
    ▼
Attacker IP blocked 
```

Wazuh's **`firewall-drop`** is a built-in Active Response script located at:
```
/var/ossec/active-response/bin/firewall-drop
```
It accepts the attacker's IP as an argument and adds a DROP rule to `iptables` (Linux) or `firewall-cmd` (systems using firewalld).

---

##  Prerequisites

- [ ] Lab 1 completed (Wazuh stack on Kali Purple, Agent on Kali Linux)
- [ ] Both VMs on the same network and able to ping each other
- [ ] Hydra installed on Kali Purple
- [ ] SSH running on Kali Linux (victim)
- [ ] Root / sudo access on both machines

---

##  Configuration

### 1. Enable Active Response on Wazuh Manager (Kali Purple)

Edit the Wazuh Manager config:
```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following block inside `<ossec_config>`:
```xml
<active-response>
  <command>firewall-drop</command>
  <location>local</location>
  <rules_id>5763</rules_id>
  <timeout>180</timeout>
</active-response>
```

| Parameter | Meaning |
|---|---|
| `<command>` | Built-in script to run: `firewall-drop` |
| `<location>` | `local` = run on the agent where the event was detected |
| `<rules_id>` | Trigger only when Rule 5763 fires (SSH brute force) |
| `<timeout>` | Auto-unblock after 180 seconds (3 minutes) |

### 2. Restart Wazuh Manager to Apply Config
```bash
sudo systemctl restart wazuh-manager
```

### 3. Verify Active Response is Enabled on Agent (Kali Linux)
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Ensure this is **not** present (it would disable AR):
```xml
<!-- <active-response>
  <disabled>yes</disabled>
</active-response> -->
```
The default is enabled — no changes needed unless it was explicitly disabled.

---

##  Attack Simulation

> ⚠️ **Disclaimer:** Performed in a **controlled, isolated lab environment** only.

### From Kali Purple — Launch Hydra SSH Brute Force:
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<KALI_LINUX_IP> -t 4 -V
```

| Flag | Description |
|---|---|
| `-l root` | Target username |
| `-P rockyou.txt` | Password wordlist |
| `ssh://` | Protocol + target IP |
| `-t 4` | 4 parallel threads |
| `-V` | Verbose — shows each attempt |

### Expected Hydra Output (before block):
```
[DATA] attacking ssh://<KALI_LINUX_IP>:22/
[ATTEMPT] target <KALI_LINUX_IP> - login "root" - pass "123456" - 1 of N
[ATTEMPT] target <KALI_LINUX_IP> - login "root" - pass "password" - 2 of N
[ATTEMPT] target <KALI_LINUX_IP> - login "root" - pass "qwerty" - 3 of N
...
```

---

##  Active Response in Action

### Once Rule 5763 fires, Hydra gets cut off:
```
[ATTEMPT] target <KALI_LINUX_IP> - login "root" - pass "letmein" - 15 of N
[ERROR] could not connect to ssh://<KALI_LINUX_IP>:22 - Connection refused
[ERROR] could not connect to ssh://<KALI_LINUX_IP>:22 - Connection refused
[ERROR] could not connect to ssh://<KALI_LINUX_IP>:22 - Connection refused
```
> Hydra keeps trying but every packet from Kali Purple is now **silently dropped** by iptables on the victim.

### On Kali Linux — verify the block rule was inserted:
```bash
sudo iptables -L INPUT -n --line-numbers
```
```
Chain INPUT (policy ACCEPT)
num  target   prot opt source               destination
1    DROP     all  --  <KALI_PURPLE_IP>     0.0.0.0/0
```
> Wazuh's `firewall-drop` script added this rule automatically the moment Active Response triggered.

### Active Response log on Kali Linux (victim):
```bash
sudo tail -f /var/ossec/logs/active-responses.log
```
```
Active response initiated: firewall-drop add - <KALI_PURPLE_IP>
Active response completed: firewall-drop add - <KALI_PURPLE_IP>
```

---

##  Verification

### 1. Confirm block from Kali Purple side:
```bash
# Try to ping the victim — should fail (packets dropped)
ping <KALI_LINUX_IP>
# Expected: Request timeout / no response
```

### 2. After timeout (180s), confirm auto-unblock:
```bash
# Wazuh removes the iptables rule after the configured timeout
sudo iptables -L INPUT -n
# The DROP rule for KaliPurple-IP should be gone
```

### 3. Check Active Response alert in Wazuh Dashboard:
- Go to **Security Events** on the dashboard
- Look for event description: `Host Blocked by firewall-drop Active Response`
- Rule ID: **601** (Active Response alert)

---

## 📜 Wazuh Rules & Response Triggered

| Rule ID | Description | Level | Action |
|---|---|---|---|
| **5710** | sshd: Attempt to login using a non-existent user | 5 | Log only |
| **5712** | sshd: SSHD brute force trying to get access | 10 | Log only |
| **5763** | sshd: brute force attack (more aggressive threshold) | **10** |  **Triggers Active Response** |
| **601** | Host blocked by Active Response | 3 | Logged on dashboard |

---

##  Key Takeaways

1. **Wazuh Active Response automates defense** — no SOC analyst needed to manually block an attacker.
2. **Rule 5763** is the brute force threshold trigger specifically wired to fire Active Response.
3. **`firewall-drop`** is a built-in Wazuh script — it uses `iptables`/`firewall-cmd` to insert a DROP rule at the OS level on the victim machine.
4. **Hydra gets blocked mid-attack** — you can visually see it go from attempts → connection refused in real time.
5. **The block is temporary** — the `<timeout>` value auto-removes the rule, preventing permanent lockout from legitimate users.
6. This lab demonstrates the full **detect → respond → recover** loop that mirrors real-world SOC automation (SOAR-lite).

---

## ⚠️ Troubleshooting Tips

| Issue | Fix |
|---|---|
| Active Response not triggering | Confirm `<rules_id>5763</rules_id>` is correct and manager was restarted |
| iptables rule not appearing | Check `/var/ossec/logs/active-responses.log` for errors |
| Hydra not blocked fast enough | Lower the brute force threshold in Wazuh rules or increase Hydra threads |
| Block rule persists after timeout | Manually flush: `sudo iptables -F INPUT` |
| Agent not receiving AR command | Verify agent is **Active** in dashboard and AR is not disabled in agent config |

---

## 📚 References

- [Wazuh Active Response Docs](https://documentation.wazuh.com/current/user-manual/capabilities/active-response/index.html)
- [Wazuh PoC — Brute Force Detection](https://documentation.wazuh.com/current/proof-of-concept-guide/detect-brute-force-attack.html)
- [MITRE ATT&CK T1110.001 — Password Guessing](https://attack.mitre.org/techniques/T1110/001/)
- [Hydra — THC Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- [iptables Man Page](https://linux.die.net/man/8/iptables)

---

<div align="center">

Made with 🔒 for learning purposes &nbsp;|&nbsp; **Lab by:** Your Name  
⭐ Star this repo if you found it helpful!

</div>
