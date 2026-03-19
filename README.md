☁️ Cloud Honeypot on OpenStack
<p align="center"> <img src="https://img.shields.io/badge/OpenStack-Trusty-red?logo=openstack" alt="OpenStack"> <img src="https://img.shields.io/badge/IDS-Suricata-ff6600?logo=suricata" alt="Suricata"> <img src="https://img.shields.io/badge/SIEM-ELK%20Stack-005571?logo=elastic" alt="ELK"> <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License"> <img src="https://img.shields.io/badge/Python-3.10-blue?logo=python" alt="Python"> <img src="https://img.shields.io/badge/Status-Test%20Environment-lightgrey" alt="Test"> </p><p align="center"> <b>Course Project:</b> NT524.Q11.ANTT – Cloud Computing Architecture and Security<b>Team 8:</b> Huỳnh Đăng Khoa (23520737) · Khiếu Bảo Lâm (23520830) </p>

📖 Table of Contents
🌟 Introduction

✨ Key Features

🏗️ Overall Architecture

🌐 Network Topology

🧩 Core Components

1. Controller Gateway

2. Analyzer (Suricata IDS + Orchestrator)

3. Honeypot Zone

3.1 T-Pot (SSH & HTTP)

3.2 MinIO (Fake S3)

3.3 MySQL (Fake RDS)

4. SIEM Layer (ELK Stack)

⚙️ Automatic Redirection Mechanism

🚀 Deployment Guide

🎬 Demo Scenarios

📊 Achieved Results

📚 References

👥 Authors

🌟 Introduction
In cloud computing, services like object storage (S3), relational databases (RDS), and SSH are frequently misconfigured, leading to rapid compromise. According to research, exposed cloud resources can be discovered and exploited within minutes. Traditional passive defenses (firewalls, signature‑based detection) are insufficient against evolving threats.

This project implements a proactive defense system on an OpenStack private cloud using:

Suricata IDS for real‑traffic analysis.

A network of honeypots emulating popular cloud services.

Automatic traffic redirection (DNAT) to isolate attackers into decoy environments.

ELK Stack for centralized logging, enrichment, and visualization mapped to MITRE ATT&CK.

The system not only protects real assets but also gathers valuable threat intelligence by recording attacker behavior in a controlled environment.

All testing was performed in an isolated lab environment – no real Internet traffic was ever directed to the honeypots. This project is a proof‑of‑concept and should not be deployed on production networks without extensive security review.

✨ Key Features
🔍 Real‑time attack detection using custom Suricata rules with MITRE ATT&CK tagging.

🔄 Automatic traffic redirection (DNAT) based on detection events.

🧪 Multi‑service honeypots:

SSH (Cowrie) – logs all commands and file transfers.

HTTP (Honeytrap) – logs requests, serves fake pages.

S3‑compatible object storage (MinIO) – logs API calls.

MySQL / RDS (custom emulator) – logs connections and SQL queries.

📈 Centralized logging & analysis with ELK Stack (Elasticsearch, Logstash, Kibana) + Filebeat.

🗺️ Geolocation of attackers via GeoIP.

🏷️ MITRE ATT&CK mapping (T1110, T1595, T1530, T1213, etc.).

🧱 Fully deployed on OpenStack using VMs and OVS mirroring.

🏗️ Overall Architecture
![Architecture](architecture.png)

The architecture is divided into four logical zones:

Zone	Role	Technologies
Controller	Entry point for all external traffic; performs DNAT redirection.	Ubuntu, iptables, Linux bridge
Analyzer	Receives mirrored traffic; detects attacks; commands redirection.	Suricata, Python
Honeypot Zone	Isolated network with decoy services.	T‑Pot, MinIO, custom MySQL emulator
SIEM Layer	Aggregates, enriches, and visualizes logs.	Elasticsearch, Logstash, Kibana, Filebeat
Data Flow:

Ingress: Traffic arrives at Controller (floating IP).

Mirroring: A copy is sent to Analyzer via OVS port mirroring.

Detection: Suricata analyzes traffic; on alert, logs to fast.log.

Orchestration: Python script monitors logs, extracts attacker IP, and sends DNAT command to Controller.

Redirection: Controller adds iptables DNAT rule forwarding the attacker's subsequent packets to the appropriate honeypot.

Interaction: Attacker unknowingly interacts with honeypot; actions are logged.

Collection: Filebeat ships logs to Logstash.

Processing: Logstash parses, enriches (GeoIP, MITRE tags), and sends to Elasticsearch.

Visualization: Kibana dashboards display attack data.

🌐 Network Topology
![Network Topology](topology.png)

The network is segmented for security:

External Network – Provides floating IP to Controller.

Management Network – Communication between Analyzer and Controller (SSH).

Honeypot Network (10.10.30.0/24) – Hosts all honeypot VMs, isolated from Internet.

Real Application Network – Hosts genuine services (e.g., OWASP Juice Shop on port 80, real SSH on port 64295).

Each honeypot VM has a single NIC on the Honeypot Network and security groups allowing only necessary inbound ports (from Controller) and outbound to Logstash.

🧩 Core Components
1. Controller Gateway
Role: Acts as the ingress point, performs DNAT redirection based on commands from Analyzer.

Key functions: IP forwarding, NAT masquerading, dynamic iptables updates.

Security: SSH access restricted to management network.

2. Analyzer (Suricata IDS + Orchestrator)
Role: Inspects mirrored traffic, detects attacks, and triggers redirection.

Components:

Suricata with custom rules (including MITRE ATT&CK metadata).

Python script that tails Suricata logs, extracts attacker IP, and sends SSH commands to Controller.

Rules cover: SSH brute‑force, HTTP probing, MySQL brute‑force, MinIO API scanning, etc.

3. Honeypot Zone
3.1 T‑Pot (SSH & HTTP)
Cowrie: Medium‑interaction SSH honeypot; logs credentials, commands, file transfers.

Honeytrap: Low‑interaction HTTP honeypot; logs requests, serves fake pages.

Logs are stored in JSON format.

3.2 MinIO (Fake S3)
MinIO server configured as an S3‑compatible object store.

Pre‑created fake buckets and objects.

Logs all API calls (ListBuckets, GetObject, PutObject) in JSON.

3.3 MySQL (Fake RDS)
Custom Python emulator that mimics MySQL protocol (handshake, auth, query responses).

No real data; responds with fake tables.

Logs connections and SQL queries in JSON.

4. SIEM Layer (ELK Stack)
Filebeat: Installed on all honeypots and Analyzer to ship logs.

Logstash: Pipelines parse logs, enrich with GeoIP, add MITRE tags, and send to Elasticsearch.

Elasticsearch: Stores indexed logs.

Kibana: Dashboards for attack maps, MITRE technique distribution, real‑time events, top attackers.

⚙️ Automatic Redirection Mechanism
Log Monitoring: Python script continuously reads Suricata's fast.log using tail -F.

IP Extraction: Regex extracts attacker IP and alert message.

Attack Classification: Simple keyword matching determines the attack type (SSH, HTTP, MySQL, MinIO).

SSH to Controller: Script connects to Controller via Paramiko (key‑based auth).

Rule Check: Verifies if a DNAT rule for that IP already exists.

DNAT Injection: If not, adds an iptables rule:

text
iptables -t nat -A PREROUTING -s <IP> -p tcp --dport <PORT> -j DNAT --to-destination <HONEYPOT_IP>:<PORT>
Persistence: Rules are saved with iptables-save (optional).

This redirection is transparent to the attacker.

🚀 Deployment Guide
Prerequisites
OpenStack (Yoga+) with Neutron, Nova.

Ubuntu 22.04 images, flavor m1.medium (2 vCPU, 4GB RAM) for VMs.

Security groups configured for required ports.

High‑Level Steps
Create networks: external, management, honeypot, real-app.

Launch VMs: Controller, Analyzer, T‑Pot, MinIO, MySQL, ELK.

Configure Controller: IP forwarding, NAT, iptables.

Set up OVS mirroring from Controller’s internal port to Analyzer.

Install Suricata on Analyzer with custom rules.

Deploy honeypots (T‑Pot, MinIO, custom MySQL).

Install ELK and configure Logstash pipelines.

Install Filebeat on all honeypots and Analyzer.

Run Python orchestrator on Analyzer.

Detailed commands are available in the project documentation.

🎬 Demo Scenarios
Scenario 1: SSH Brute‑Force
Attacker uses Hydra on port 22.

Suricata triggers rule T1110.

Redirector adds DNAT rule; attacker is sent to Cowrie.

Cowrie logs login attempts and commands.

Kibana shows events with MITRE tag T1110.

Scenario 2: HTTP Probing
Attacker requests /admin.

Suricata detects (T1595) and redirects to Honeytrap.

Honeytrap logs request; Kibana tags T1595.

Scenario 3: MinIO (Fake S3) Attack
Attacker scans port 9000, attempts to list buckets.

Suricata detects MinIO API scan (T1595) and redirects to MinIO honeypot.

Attacker sees fake buckets and may upload files.

MinIO logs API calls; Kibana tags T1595 (scanning) and T1530 (data access).

Scenario 4: MySQL (Fake RDS) Attack
Attacker connects to port 3306.

All traffic is DNAT‑ed to MySQL honeypot.

Attacker logs in and runs SQL queries.

Honeypot logs queries; Kibana tags T1110 (brute‑force) and T1213 (data from repositories).

📊 Achieved Results
Redirection latency: < 2 seconds.

Detection accuracy: Successfully identified all test attacks.

Log volume: Processed >50,000 events during testing.

MITRE mapping: Automated tagging for multiple techniques.

Kibana dashboards: Provided clear visibility into attacker behavior and geography.

All testing was performed in an isolated lab environment. The system is a proof‑of‑concept for educational purposes.