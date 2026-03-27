# 🛡️ Wazuh SIEM Lab — Stack Installation, Configuration & Agent Deployment
(Latest wazuh_branch="4.14.4")

This project demonstrates the **deployment of a Wazuh SIEM (Security Information and Event Management) system** in a lab environment.

The lab simulates a real-world security monitoring setup where:

- Logs are generated on an endpoint
- Collected by an agent
- Sent to a SIEM
- Analyzed and visualized in a dashboard

---

#  Project Overview

Wazuh is an open-source security platform that provides:

- Log Analysis
- Host-based Intrusion Detection System (HIDS)
- File Integrity Monitoring (FIM)
- Malware Detection
- Compliance Monitoring

---

#  Lab Environment

| Component | Role |
|----------|------|
| Kali Purple | Wazuh SIEM Server |
| Kali Linux | Endpoint (Victim System) |
| Wazuh Manager | Log analysis |
| Wazuh Agent | Log collection |
| Wazuh Indexer | Data storage |
| Wazuh Dashboard | Visualization |

---

#  System Requirements

| Resource | Minimum | Recommended |
|---------|--------|-------------|
| Disk | 50 GB | 100+ GB |
| RAM | 8 GB | 16 GB |
| CPU | 2 vCPU | 4 vCPU |

**This lab using 16GB and 4 processors for kali purple & 8GB and 2 processors for kali linux**

---

# 🟢 Phase 1 — Initial Setup

### Snapshot Setup

Take VM snapshots before starting to allow rollback.

---

### Update Systems

```bash
sudo apt update && sudo apt upgrade -y
```

---

### Set Hostnames

On Kali Purple:

```bash
sudo hostnamectl set-hostname wazuh-siem

# also change old host name in editor: sudo nano /etc/hosts -> then change old host name to new.
```

On Kali Linux:

```bash
sudo hostnamectl set-hostname victim-endpoint

# also change old host name in editor: sudo nano /etc/hosts -> then change old host name to new.
```

Verify:

```bash
hostname
```



### Verify Connectivity

```bash
ping <target_ip>
```

Both systems must communicate successfully.

---

# 🟢 Phase 2 — Prepare Wazuh Server

### Install Required Packages

```bash
sudo apt install -y curl apt-transport-https lsb-release gnupg2
```


<img width="661" height="501" alt="Install Required Base Packages_01" src="https://github.com/user-attachments/assets/0206665a-1a7f-4ec9-a7e4-a88c93b3efad" />



### Disable Sleep Mode (SIEM servers should never sleep)

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```


<img width="715" height="122" alt="Disable Automatic Sleep-Suspend_02" src="https://github.com/user-attachments/assets/61e11f7d-17b5-477d-a0ba-c549bb030a0e" />



### Configure Time Synchronization (Logs without correct time is useless)

```bash
sudo apt install -y chrony
sudo systemctl enable chrony
sudo systemctl start chrony

##Verify-
timedatectl
```


<img width="849" height="745" alt="Set Proper Time Synchronization_03" src="https://github.com/user-attachments/assets/c5404877-53dd-4939-a9c3-e91830ee1b45" />



### Open Required Firewall Ports

```bash

#-	Install UFW if not present (ufw is a simple firewall manager)
sudo apt install -y ufw

sudo ufw allow 1514/tcp
sudo ufw allow 1515/tcp
sudo ufw allow 443/tcp
sudo ufw allow 55000/tcp
sudo ufw enable
```


<img width="356" height="250" alt="Open Required Firewall Ports_04" src="https://github.com/user-attachments/assets/58fd8799-9636-48fd-ab89-2750c80dcbdc" />


---

# 🟢 Phase 3 — Install Wazuh Stack (On Kali Purple)

### Download Installer (check for latest branch [https://documentation.wazuh.com/current/quickstart.html])

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
chmod +x wazuh-install.sh
```

---

### Run Installation (single-node) 

```bash
sudo ./wazuh-install.sh -a
# Installation time: ~10–20 minutes, depending on system specs.
```


<img width="909" height="691" alt="Run single-node installation_05" src="https://github.com/user-attachments/assets/4d36a473-2e9b-4208-aef6-b7511d25d613" />


**-*-*At the end of the successful installation, the terminal will give a dashboard URL, including username and password to access the Wazuh dashboard. Save the credentials in a text file (or) in a nano text file.-*-***

**- nano wazuh-creds.txt**



### Verify Services

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All services should be **active and running**.


<img width="766" height="56" alt="Verify services_001" src="https://github.com/user-attachments/assets/29398722-b678-4ab1-be28-9fb8828cfb46" />

<img width="763" height="51" alt="Verify services_002" src="https://github.com/user-attachments/assets/28e79935-7ab5-4ab3-bfdd-8941296f9516" />

<img width="746" height="51" alt="Verify services_003" src="https://github.com/user-attachments/assets/1d47ddf8-5e37-47dd-a23a-650eefea92f5" />


### Access Dashboard

```
https://<YOUR_IP>
```

Login using generated credentials.


<img width="1701" height="819" alt="Wazuh Dashboard_06" src="https://github.com/user-attachments/assets/030ce2a7-63e6-48b7-92af-31c967d2ee9f" />


---

# 🟢 Phase 4 — Deploy Wazuh Agent

### Generate Agent Command

Navigate in dashboard:

```
1. Agents Management → Deploy New Agent

2.	Select Operating System and pakage
    To check the system specs of linux based system:
   -	Check Distribution pakage: cat /etc/os-release
   -	Check Architecture: uname -m

3.	Enter Agent name (Example: victim-endpoint)

4.	Copy the Generated Command

5.  Paste and Run the Command on Kali linux (Victim system) to install the agent.

```

*Agent Deployment Page*

<img width="1300" height="916" alt="P4_Open Agent Deployment Page _07" src="https://github.com/user-attachments/assets/74a99d76-ad2f-4f3d-a84d-ff1f61d3a158" />


*Running generated command on kali linux*

<img width="1692" height="333" alt="P4_Copy the Generated Command_08" src="https://github.com/user-attachments/assets/b98788f1-6a8c-49f3-91fd-de0dc19a48f7" />


---


### Start Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```


### Verify Agent

```bash
sudo systemctl status wazuh-agent
```

<img width="746" height="112" alt="image" src="https://github.com/user-attachments/assets/efe68c64-c7b3-46d5-a64f-63556dc8e397" />


Confirm agent appears in dashboard.


<img width="1615" height="589" alt="image" src="https://github.com/user-attachments/assets/327df335-0289-4ce5-9951-f92da442d4b4" />


---

# Follow the link to see detection results and troubleshooting:

```bash
#(Troubleshooting notes)
https://github.com/siddhesh-001/Wazuh-SIEM-Lab/blob/main/Project%20Notes%20(troubleshooting%20steps)
```

# Wazuh Alert Levels

Wazuh rates every alert from Level 0 to Level 15

Open the dashboard → go to Security Events Click on any alert to expand it

| Level Range | Severity | Example Event |
|-------------|----------|---------------|
| 0–3 | Informational | System startup, successful login |
| 4–7 | Low | Single failed login attempt |
| 8–11 | Medium | Multiple failed login attempts |
| 12–14 | High | Brute-force attack detected |
| 15 | Critical | Rootkit or system takeover |


---

#  Key Learning Outcomes

This project demonstrates:

- SIEM deployment and configuration
- Log collection and analysis pipeline
- Endpoint monitoring using agents
- Security event detection workflow
- SOC-style investigation process

