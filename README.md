# 🛡️ ML-Intrusion-Defense (Cowrie ML Firewall)

> **Honeypot-based Intrusion Detection & Automated Firewall Response System for MikroTik Networks**

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Machine Learning](https://img.shields.io/badge/Machine%20Learning-FF6F00?style=for-the-badge&logo=tensorflow&logoColor=white)
![MikroTik](https://img.shields.io/badge/MikroTik-293239?style=for-the-badge&logo=mikrotik&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

---

## 📌 What This Project Does

This system automatically **detects, blocks, and alerts** on malicious network activity in real time — without any manual intervention.

It works by:
1. Redirecting suspicious traffic into a **Cowrie SSH/Telnet honeypot**
2. Using a **Random Forest ML model + keyword rules** to classify attacker behaviour from honeypot logs
3. Automatically **pushing firewall drop rules** to a MikroTik router via Netmiko
4. Sending **email alerts** to the admin and displaying live results on a **Streamlit dashboard**

---

## 🧩 System Architecture

```
MikroTik Router
    │
    │  Detects port scans / brute-force attempts
    │  Redirects suspicious traffic (port 22 → honeypot)
    ▼
Cowrie Honeypot (Linux VM)
    │
    │  Captures attacker sessions
    │  Saves logs → cowrie.json
    ▼
parse_and_detect.py
    │
    │  Parses logs
    │  Hybrid detection: Random Forest + keyword rules
    ▼
    ├──► update_mikrotik.py   →  Adds IP drop rules (1-day block)
    ├──► send_alert_email.py  →  Notifies admin via email
    └──► dashboard.py         →  Live Streamlit monitoring dashboard
```

---

## 📁 Project Structure

```
cowrie-ml-firewall/
│
├── src/
│   ├── parse_and_detect.py      # ML-based log analysis & threat detection
│   ├── send_alert_email.py      # Email alert system for admin notifications
│   └── update_mikrotik.py       # Automated MikroTik firewall rule updater
│
├── dashboard/
│   └── dashboard.py             # Streamlit live monitoring dashboard
│
├── data/                        # Honeypot log samples & training data
├── requirements.txt             # Python dependencies
└── README.md
```

---

## ⚙️ How It Works

### 1. MikroTik — Traffic Redirection & Detection

The router detects suspicious behaviour (port scans, brute-force) and silently redirects it to the honeypot:

```bash
# Redirect port 22 traffic to Cowrie honeypot VM
/ip firewall nat add chain=dstnat protocol=tcp dst-port=22 \
  action=dst-nat to-addresses=<HONEYPOT_IP> to-ports=2222

# Detect and log port scanners
/ip firewall filter add chain=input protocol=tcp psd=21,3s,3,1 \
  action=add-src-to-address-list address-list=malicious \
  address-list-timeout=1d comment="Port scan detection"

# Drop all malicious IPs
/ip firewall filter add chain=input \
  src-address-list=malicious action=drop comment="Block malicious IPs"
```

### 2. Cowrie Honeypot — Attacker Capture

Cowrie emulates an SSH/Telnet server, recording every command an attacker runs.

```bash
# Install and run Cowrie
sudo adduser --disabled-password cowrie
sudo su - cowrie
git clone https://github.com/cowrie/cowrie.git && cd cowrie
python3 -m venv cowrie-env && source cowrie-env/bin/activate
pip install -r requirements.txt
cp cowrie.cfg.dist cowrie.cfg
bin/cowrie start
```

### 3. ML Detection — `parse_and_detect.py`

Reads `cowrie.json` logs and classifies each session using:
- **Random Forest classifier** trained on attacker behaviour patterns
- **Keyword-based rules** for known malicious commands

### 4. Automated Firewall Block — `update_mikrotik.py`

Once an attacker IP is flagged, it's automatically added to the MikroTik block list via Netmiko — no manual action needed. Block duration: **1 day**.

### 5. Alerts & Dashboard

- **`send_alert_email.py`** — sends an admin email with attacker IP and session details
- **`dashboard.py`** — Streamlit dashboard showing live detections, blocked IPs, and attack trends

---

## 🚀 Getting Started

### Prerequisites

- Linux VM (for Cowrie honeypot)
- MikroTik router with API/SSH access
- Python 3.8+

### Installation

```bash
# Clone the repo
git clone https://github.com/Harikanth2307/cowrie-ml-firewall.git
cd cowrie-ml-firewall

# Install Python dependencies
pip install -r requirements.txt
```

### Configuration

1. Update MikroTik IP, username, and password in `update_mikrotik.py`
2. Configure email credentials in `send_alert_email.py`
3. Point `parse_and_detect.py` to your Cowrie log path (`cowrie.json`)

### Run

```bash
# Run detection + auto-block
python src/parse_and_detect.py

# Launch dashboard
streamlit run dashboard/dashboard.py
```

---

## 🔧 Tech Stack

| Component | Technology |
|---|---|
| Honeypot | Cowrie (SSH/Telnet) |
| ML Detection | Python, Scikit-learn (Random Forest) |
| Firewall Automation | Netmiko, MikroTik RouterOS |
| Alerting | Python smtplib |
| Dashboard | Streamlit |
| Log Format | JSON (cowrie.json) |

---

## 👤 Author

**Hari Kanth Karne** — Cloud Engineer | Cybersecurity | AI | ML

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/hari-kanth-karne-741083368)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Harikanth2307)
