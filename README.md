# 🛡️ Wazuh SIEM Lab — Installation, Configuration & Agent Deployment

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



### Disable Sleep Mode (SIEM servers should never sleep)

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```



### Configure Time Synchronization (Logs without correct time is useless)

```bash
sudo apt install -y chrony
sudo systemctl enable chrony
sudo systemctl start chrony

##Verify-
timedatectl
```



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

---

# 🟢 Phase 3 — Install Wazuh Stack (On Kali Purple)

### Download Installer

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
chmod +x wazuh-install.sh
```

---

### Run Installation (single-node) 

```bash
sudo ./wazuh-install.sh -a
```

Installation time: ~10–20 minutes, depending on system specs.

**-*-*At the end of the successful installation, the terminal will give a dashboard URL, including username and password to access the Wazuh dashboard. Save the credentials in a text file (or) in a nano text file.-*-***

**- nano wazuh-creds.txt**

---

### Verify Services

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All services should be **active and running**.

---

### Access Dashboard

```
https://<YOUR_IP>
```

Login using generated credentials.

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

---

### Install Agent on Endpoint

Run generated command on Kali Linux.

---

### Start Agent

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

### Verify Agent

```bash
sudo systemctl status wazuh-agent
```

Confirm agent appears in dashboard.

---

# 🟢 Detection Workflow

This lab simulates real SOC monitoring:

1. Logs generated on endpoint
2. Wazuh agent collects logs
3. Logs sent to manager
4. Rules applied
5. Alerts generated
6. Dashboard displays alerts
7. Analyst investigates

---

#  Key Learning Outcomes

This project demonstrates:

- SIEM deployment and configuration
- Log collection and analysis pipeline
- Endpoint monitoring using agents
- Security event detection workflow
- SOC-style investigation process

---

#  Skills Demonstrated

- SIEM (Wazuh)
- Log Analysis
- Threat Detection
- Linux System Administration
- Network Monitoring
- Security Operations (SOC)
