# 🛡️Wi-Fi Security Audit 
Using Python, Wireshark, Kismet & Aircrack-ng on Kali Linux

## 🔍 Objective:
This project aims to perform a **Wi-Fi security audit** to identify vulnerabilities in wireless networks using **Kali Linux tools**, **Python**, and GUI-based tools like **Wireshark** and **Kismet**.
To evaluate the security posture of wireless networks by conducting a comprehensive Wi-Fi security audit using open-source tools such as Wireshark, Kismet, and Aircrack-ng.

The goal is to identify vulnerabilities such as weak encryption protocols, exposed SSIDs, rogue access points, and unauthorized devices; simulate potential attack scenarios; and provide actionable recommendations to strengthen wireless network defenses while ensuring ethical and legal compliance throughout the assessment.

---

## 📁 Table of Contents
1. [Overview](#overview)
2. [Tools and Lab Environment](#toolsandlabenvironment)
3. [Prerequisites](#prerequisites)
4. [Tools Used](#tools-used)
5. [Setup Instructions](#setup-instructions)
6. [Audit Workflow](#audit-workflow)
7. [Python Scripts](#python-scripts)
8. [Wireshark Use Case](#wireshark-use-case)
9. [Kismet Use Case](#kismet-use-case)
10. [Aircrack-ng Use Case](#aircrack-ng-use-case)
11. [Security Recommendations](#security-recommendations)
12. [Output](output)
13. [Scan-Result Summary](#scan-resultsummary)
14. [Summary](#summary)
15. [Challenges & Resolutions](#challenges&resolutions)
16. [Key Takeaways](#keytakeaways)
17. [Future-Enhancements](#future-enhancements)
18. [References](#references)
19. [Acknowledgement](#acknowledgement)
20. [Project-Snapshot](#project-snapshot)
21. [Directory Structure](#directory-structure)
22. [Authors](#authors)

---

## 🔎 Overview

The audit focuses on:
- Scanning and detecting all nearby networks
- Capturing handshake packets
- Analyzing encryption levels and vulnerabilities
- Using Python for automation
- Identifying weak or open Wi-Fi access points

---

# 🧰 Tools & Lab Environment

| Component            | Specification                                |
| -------------------- | -------------------------------------------- |
| **Operating System** | Kali Linux 2024.1                            |
| **Tools Utilized**   | Nmap (CLI), Zenmap (GUI)                     |
| **Virtualization**   | VirtualBox (Host-Only Networking)            |
| **Target Machine**   | Metasploitable2 (Deliberately Vulnerable VM) |

---

## ⚙️ Prerequisites

- Kali Linux (latest version)
- Wi-Fi adapter with monitor mode support
- Python 3.x
- sudo/root access
- Installed tools: aircrack-ng, kismet, wireshark

---

## 🛠️ Tools Used

| Tool           | Description                                  |
|----------------|----------------------------------------------|
| Aircrack-ng    | Command-line suite for Wi-Fi penetration     |
| Kismet         | GUI-based passive wireless detection         |
| Wireshark      | GUI tool to analyze packets                  |
| Pyshark/Scapy  | Python packet processing libraries           |
| ifconfig/iwconfig | Network configuration commands            |

---

## 🖥️ Setup Instructions

```bash
sudo apt update
sudo apt install aircrack-ng kismet wireshark python3-pip
pip3 install pyshark scapy
```

Enable monitor mode:
```bash
sudo airmon-ng check kill
sudo airmon-ng start wlan0
```

---

## 🔧 Audit Workflow

### Step 1: Scan for APs using airodump-ng
```bash
sudo airodump-ng wlan0mon
```

### Step 2: Target one AP and capture handshake
```bash
sudo airodump-ng --bssid <BSSID> -c <CH> -w capture wlan0mon
```

### Step 3: Optional Deauth to force handshake
```bash
sudo aireplay-ng --deauth 10 -a <BSSID> wlan0mon
```

### Step 4: Analyze with Wireshark
- Open `capture.cap` in Wireshark
- Use filters: `eapol` to view handshakes

### Step 5: Crack with Aircrack-ng
```bash
aircrack-ng capture-01.cap -w /path/to/wordlist.txt
```

---

## 🐍 Python Scripts

### Extract Wi-Fi Info from pcap
```python
import pyshark
cap = pyshark.FileCapture('capture-01.cap', display_filter='wlan.fc.type_subtype == 0x08')

for pkt in cap:
    print(f"SSID: {pkt.wlan.ssid} | Signal: {pkt.radiotap.dbm_antsignal}dBm")
```

---

## 🧪 Wireshark Use Case

1. Open `capture.cap`
2. Use filters like:
   - `wlan.ssid`
   - `eapol` for handshakes
   - `http` for unsecured traffic
3. Analyze frame details to find anomalies or exposed data

---

## 📡 Kismet Use Case

1. Launch Kismet: `sudo kismet`
2. Monitor real-time device connections
3. Export logs for analysis (`.pcap` format)
4. Identify unknown or suspicious MACs

---

## 🔓 Aircrack-ng Use Case

1. Capture and crack handshake:
```bash
aircrack-ng capture-01.cap -w rockyou.txt
```
2. If success, shows decrypted key
3. Test vulnerability of weak PSKs

---

## 🛡️ Security Recommendations

- Use **WPA3** or **WPA2-AES**
- Avoid open networks or WEP
- Disable **WPS** feature
- Change default router credentials
- Use long, complex passphrases

---

## 📄 Output

```
SSID: Public_Wifi | Signal: -70dBm
SSID: HomeNetwork | Signal: -45dBm

Captured Handshake Packets: 4
Encryption Detected: WPA2-PSK
Cracked Key: P@ssw0rd123
```

---
# 📊Scan Result Summary
During the audit, the following Wi-Fi networks were detected using airodump-ng, kismet, and verified with Wireshark:

SSID	BSSID	Channel	Signal Strength	Encryption	Handshake Captured
Public_WiFi	AA:BB:CC:DD:EE:FF	6	-72 dBm	Open	N/A
HomeNetwork	11:22:33:44:55:66	11	-43 dBm	WPA2-PSK	✔️ Yes
CafeSecure	77:88:99:AA:BB:CC	1	-60 dBm	WPA3	❌ No
OfficeGuest	DD:EE:FF:00:11:22	3	-55 dBm	WPA2-PSK	✔️ Yes

Packet capture size: 3.2 MB

Tool used for capture: airodump-ng, verified with Wireshark

Timeframe: ~20 minutes of scan and collection

---

# 📝 Summary
Multiple Wi-Fi access points were scanned and identified.

Open networks (Public_WiFi) were detected and marked as insecure.

Successful WPA2 handshakes were captured from two secured networks.

WPA3 access point handshake was not captured due to advanced protection.

Packet analysis using Python and Wireshark confirmed packet integrity and handshake frames.

---

# 🚧Challenges and Resolutions
Challenge	Resolution
Interface not entering monitor mode	Used airmon-ng check kill and restarted networking services
No handshake capture despite traffic	Performed deauthentication attack to trigger client reconnection
Pyshark failing to parse large pcap	Split .cap file into smaller chunks or used filter during loading
WPA3 networks unable to crack or capture handshakes	Acknowledged limitation; WPA3 uses forward secrecy and SAE authentication
GUI crashes when running Kismet on low RAM	Ran in terminal mode and limited scan range

---

# 🌟Key Takeaways
* Open networks are still commonly deployed and pose high risk.

* WPA2 networks can still be vulnerable to dictionary attacks if weak passwords are used.

* Kismet provides rich contextual data, including device types and manufacturers.

* Wireshark is invaluable for forensic packet analysis and validating captured frames.

* Automation using Python helps in filtering and processing large pcap datasets.

---

# 🚀 Future Enhancements
* Integrate machine learning to auto-classify traffic as normal or suspicious.

* Build a real-time dashboard using Flask or Streamlit to visualize scan results.

* Include alerts for rogue APs using MAC comparison with a known safe list.

* Extend toolset with bettercap or nmap for full network topology mapping.

* Add support for email notifications when weak/open networks are found.

---

# 📚 References

- https://www.kali.org/tools/
- https://www.aircrack-ng.org/
- https://www.kismetwireless.net/
- https://www.wireshark.org/
- https://github.com/KimiNewt/pyshark
- https://youtu.be/WfYxrLaqlN8?si=wKbzuGll1tuc7p-i

---

# 🙏 Acknowledgment

Special thanks to **@official-biswadeb941** for technical guidance and strategic input throughout this lab simulation. His expertise in offensive security, enumeration tactics, and adversarial tooling played a vital role in this project’s successful execution.

---

# 📸 Project Snapshot
Field Details :
Project Name - Wi-Fi Security Audit Using Kali Linux Tools
Domain	Cybersecurity – Wireless Network Penetration Testing
Current Status	✅-  Completed
Completion Date	🗓️-  19th July 2025
Primary Tools	🔧- Wireshark, Kismet, Aircrack-ng
Environment	- Kali Linux OS, External USB Wi-Fi Adapter with monitor mode support

---

# 📁 Directory Structure: wifi-security-audit/

wifi-security-audit/
│
├── README.md                          # Project overview and instructions
├── snapshot/                          # Project snapshot and summaries
│   └── project_snapshot.md            # Includes field, status, tools, etc.
│
├── tools/                             # Tool-specific analysis and commands
│   ├── wireshark/                     # Wireshark-specific captures and filters
│   │   ├── captures/                  # .pcap files saved from audits
│   │   └── filters.txt                # Common filters used
│   │
│   ├── kismet/                        # Kismet scan results
│   │   └── kismet_logs/               # .netxml and .csv logs
│   │
│   └── aircrack-ng/                   # Handshake capture and cracking
│       ├── handshakes/                # .cap files from WPA/WPA2 handshakes
│       ├── wordlists/                 # Wordlists used for cracking
│       └── results/                   # Cracking results/logs
│
├── reports/                           # Final and intermediate reports
│   ├── full_audit_report.md           # Detailed markdown report
│   ├── summary.txt                    # Short summary of results
│   └── recommendations.pdf            # Next phase suggestions
│
├── screenshots/                       # Important screenshots of the audit
│   ├── wireshark_capture.png
│   ├── aircrack_result.png
│   └── kismet_interface.png
│
└── scripts/                           # Optional automation or helper scripts
    ├── monitor_mode.sh                # Enable/disable monitor mode script
    └── scan_and_capture.sh            # Automate scanning & capture steps

---

# 👤 Authors

** 
   1) Shreeya Sarkar-3rd year ECE 
   2) Debasmita Kar- 3rd year ECE
| Name                 | GitHub Username       | Responsibilities                                                                                          |
|----------------------|-----------------------|-----------------------------------------------------------------------------------------------------------|                        | **Shreeya Sarkar**          | `https://github.com/Shreeya-tua`             | - Scanning, identifying, and mapping wireless networks.                                                       |
| **Debasmita Kar**                                 |      -      Simulating attacks to evaluate Wi-Fi security.                                                   |
**
 [LinkedIn](https://www.linkedin.com/in/shreeya-sarkar-3808b3372/) • [GitHub](https://github.com/Shreeya-tua) 






 