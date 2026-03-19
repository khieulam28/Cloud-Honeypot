☁️ Cloud Honeypot on OpenStack
<p align="center"> <img src="https://img.shields.io/badge/OpenStack-Yoga-red?logo=openstack"> <img src="https://img.shields.io/badge/IDS-Suricata-ff6600?logo=suricata"> <img src="https://img.shields.io/badge/SIEM-ELK%20Stack-005571?logo=elastic"> <img src="https://img.shields.io/badge/Python-3.10-blue?logo=python"> <img src="https://img.shields.io/badge/Status-Lab%20Environment-lightgrey"> <img src="https://img.shields.io/badge/License-MIT-yellow.svg"> </p> <p align="center"> <b>Course Project:</b> NT524 – Cloud Computing Architecture & Security <br> <b>Team 8:</b> Huỳnh Đăng Khoa · Khiếu Bảo Lâm </p>
📖 Table of Contents

Introduction

Objectives

Key Features

Architecture Overview

Network Topology

Data Flow

Decision Logic

Core Components

Honeypot Design

Automatic Redirection Mechanism

SIEM & Log Processing

Demo Scenarios

Results

Disclaimer

🌟 Introduction

In modern cloud environments, services such as SSH, web applications, object storage (S3), and databases (RDS) are frequent targets of cyberattacks. These services are often exposed due to misconfiguration, weak credentials, or improper access control.

Studies show that publicly exposed cloud resources can be discovered and exploited within minutes using automated scanning tools.

Traditional defensive mechanisms such as firewalls or passive intrusion detection systems are no longer sufficient to handle evolving threats. Instead of only blocking attackers, a more effective approach is to observe and understand their behavior.

This project proposes a proactive defense architecture by combining intrusion detection with a honeypot system. When suspicious activity is detected, the attacker is not blocked immediately but is transparently redirected into a controlled, simulated environment.

This approach allows the system to:

Protect real services from compromise

Collect valuable threat intelligence

Analyze attacker techniques in depth

🎯 Objectives

The system is designed with the following goals:

1. Real-time attack detection

Analyze network traffic using an IDS to identify malicious behavior such as brute-force attacks, scanning, and service exploitation.

2. Automatic traffic redirection

Redirect malicious traffic to honeypots instead of allowing access to real services.

3. Cloud service simulation

Deploy multiple honeypots that mimic real-world cloud services, including:

SSH servers

Web applications

Object storage (S3)

Relational databases

4. Centralized log collection and analysis

Collect logs from multiple sources and process them using a SIEM system.

5. MITRE ATT&CK mapping

Classify attacker behavior based on standardized attack techniques.

✨ Key Features

Real-time intrusion detection using Suricata IDS

Automatic DNAT-based redirection of malicious traffic

Multi-service honeypot environment

Centralized logging with ELK Stack

GeoIP-based attacker tracking

MITRE ATT&CK technique mapping

Fully deployed on OpenStack private cloud

🏗️ Architecture Overview

The system is divided into four main layers:

1. Controller Gateway

Acts as the entry point for all incoming traffic. It is responsible for routing traffic, performing NAT operations, and enforcing redirection rules.

2. Analyzer (Detection Layer)

This component analyzes mirrored traffic using Suricata IDS. It detects attacks and generates alerts that trigger automated responses.

3. Honeypot Zone

An isolated network containing decoy services designed to simulate real cloud environments and capture attacker interactions.

4. SIEM Layer

Handles log aggregation, processing, storage, and visualization using the ELK Stack.

🌐 Network Topology

The system is segmented into multiple isolated networks:

External Network
Provides public access via floating IP assigned to the Controller.

Management Network
Used for secure communication between Analyzer and Controller.

Honeypot Network
Hosts all honeypot instances and is isolated from external access.

Real Application Network
Contains legitimate services accessible only to valid users.

This segmentation ensures that attackers cannot directly access real services.

🔄 Data Flow

The system operates through the following workflow:

Incoming traffic reaches the Controller via a floating IP.

The Controller mirrors traffic to the Analyzer.

Suricata analyzes the mirrored traffic in real time.

When an attack is detected, an alert is generated.

A Python script processes the alert and extracts attacker information.

The script sends a command to the Controller.

The Controller updates its NAT rules.

The attacker is redirected to the honeypot.

All attacker interactions are logged and forwarded to the SIEM system.

🧠 Decision Logic

The system differentiates between legitimate and malicious traffic:

Legitimate traffic → forwarded to real services

Malicious traffic → redirected to honeypots

A key strategy used is:

Real services are deployed on non-standard ports

Honeypots listen on default ports (e.g., 22, 80, 3306)

This increases the likelihood that attackers interact with honeypots instead of real systems.

🧩 Core Components
🔹 Suricata IDS

Suricata performs deep packet inspection and detects threats based on custom rules. It operates passively by analyzing mirrored traffic without affecting performance.

It can detect:

SSH brute-force attacks

HTTP scanning and probing

Database attacks

Object storage enumeration

🔹 Python Orchestrator

The orchestrator script automates system response:

Continuously monitors IDS logs

Extracts attacker IP addresses

Identifies attack types

Sends remote commands to the Controller

Updates firewall rules dynamically

🔹 Controller

The Controller acts as a network gateway and enforcement point. It applies NAT rules and redirects traffic based on instructions from the Analyzer.

🧪 Honeypot Design
1. SSH Honeypot

Simulates an SSH server and allows attackers to log in using fake credentials. It records all login attempts and commands executed.

2. HTTP Honeypot

Simulates a web server and captures requests, payloads, and scanning activities.

3. Fake S3 (MinIO)

Mimics an object storage service with fake buckets and files. It logs all API interactions such as listing and uploading objects.

4. MySQL Honeypot

Simulates a database server and responds with fake data. It captures login attempts and SQL queries executed by attackers.

⚙️ Automatic Redirection Mechanism

The redirection process works as follows:

IDS logs are continuously monitored

Attacker IP is extracted

Existing rules are checked to avoid duplication

A new redirection rule is applied if necessary

All future traffic from the attacker is redirected

This process is fully automated and operates in near real-time.

📊 SIEM & Log Processing
Log Collection

Logs are collected from:

Suricata IDS

Honeypot services

Processing

Logstash parses and enriches data by:

Extracting fields

Adding GeoIP information

Mapping MITRE ATT&CK techniques

Storage

Elasticsearch stores logs in a structured format for fast querying.

Visualization

Kibana provides dashboards for:

Attack visualization

Real-time monitoring

Threat analysis

Attacker tracking

🎬 Demo Scenarios

This section demonstrates how the system behaves under different attack scenarios. Each scenario follows the same pipeline:

Detection → Alert → Redirection → Deception → Logging → Visualization

The goal is not only to block attacks, but to observe attacker behavior in a controlled environment.

🔐 Scenario 1: SSH Brute-force Attack
🧪 Description

In this scenario, an attacker attempts to gain unauthorized access to the system by performing a brute-force attack on the SSH service.

The system is designed to:

Detect abnormal login attempts

Redirect the attacker to a fake SSH server (honeypot)

Record all attacker activities

⚙️ Attack Execution

The attacker uses automated tools (e.g., Hydra) to repeatedly attempt login with different username/password combinations against the public IP of the system.

This generates:

High-frequency connection attempts

Repeated authentication failures

Suspicious traffic patterns

🔍 Detection Phase

The Analyzer receives mirrored traffic and processes it using Suricata.

Suricata identifies:

Multiple failed login attempts

High request rate within a short time window

An alert is generated and tagged with:

MITRE ATT&CK: T1110 (Brute Force)

🔄 Redirection Phase

Once the attack is confirmed:

The system extracts the attacker’s IP address

A redirection rule is dynamically applied at the Controller

All subsequent SSH traffic from that IP is redirected

👉 Instead of reaching the real SSH service, the attacker is now interacting with a honeypot.

🎭 Deception Phase

The attacker successfully logs in using weak credentials and believes they have compromised the system.

However:

The environment is completely simulated

No real system access is granted

The honeypot mimics a real Linux system and allows:

Command execution

File interaction

📊 Logging & Analysis

All attacker actions are recorded:

Username and password attempts

Commands executed (e.g., whoami, ls, cat /etc/passwd)

Session duration and behavior

These logs are:

Converted into structured JSON

Sent to the SIEM system

Visualized in Kibana dashboards

🌐 Scenario 2: HTTP Scanning & Probing
🧪 Description

This scenario simulates an attacker performing reconnaissance by probing web endpoints.

⚙️ Attack Execution

The attacker sends crafted HTTP requests such as:

Accessing sensitive endpoints (/admin, /login)

Sending unusual payloads

Using automated scanning tools

🔍 Detection Phase

Suricata detects abnormal behavior based on:

Suspicious URL patterns

Non-human browsing behavior

Known scanning signatures

MITRE classification:

T1595 (Active Scanning)

🔄 Redirection Phase

After detection:

The attacker is silently redirected

Traffic is no longer forwarded to the real web application

🎭 Deception Phase

The attacker is now interacting with a fake web service:

Receives realistic HTTP responses

Believes the system is vulnerable

Continues probing deeper

📊 Logging & Analysis

The system records:

Requested URLs

HTTP methods (GET, POST, etc.)

Payloads and parameters

User-Agent (tools used by attacker)

☁️ Scenario 3: Object Storage (Fake S3 / MinIO Attack)
🧪 Description

This scenario simulates attacks targeting cloud object storage services, which are often misconfigured and publicly exposed.

⚙️ Attack Execution

The attacker:

Scans for open S3-compatible endpoints

Attempts to brute-force credentials

Tries to list buckets or access stored data

🔍 Detection Phase

Suricata detects:

API probing behavior

Repeated authentication attempts

MITRE classification:

T1595 (Scanning)

T1530 (Data from Cloud Storage)

🔄 Redirection Phase

Once flagged:

The attacker is redirected to a fake MinIO service

The real storage system is completely isolated

🎭 Deception Phase

The honeypot provides:

Fake buckets

Fake sensitive files

Weak credentials for easy access

The attacker may:

List buckets

Download files

Upload malicious content

📊 Logging & Analysis

Captured data includes:

API calls (ListBucket, GetObject, PutObject)

Access credentials used

Uploaded files

Interaction patterns

🗄️ Scenario 4: Database Attack (Fake MySQL / RDS)
🧪 Description

This scenario demonstrates attacks targeting publicly exposed database services.

⚙️ Attack Execution

The attacker:

Connects to port 3306

Attempts login with common credentials

Executes SQL queries

🔍 Detection Phase

Unlike other services:

👉 All external MySQL connections are considered malicious

No real database is exposed.

🔄 Redirection Phase

All database traffic is automatically redirected

No access to real data is ever granted

🎭 Deception Phase

The attacker interacts with a simulated MySQL server:

Receives realistic responses

Sees fake tables and data

Believes they have compromised the database

📊 Logging & Analysis

The system logs:

Login attempts

Executed SQL queries

Data extraction attempts

MITRE classification:

T1110 (Brute Force)

T1213 (Data from Information Repositories)

🎥 Demo Video
<p align="center"> <a href="https://drive.google.com/drive/folders/1-ASvfGYQs8oLcXdMXsbLzcvJfMdoVuuf?usp=drive_link"> <img src="https://img.shields.io/badge/🎬 Watch%20Demo-Click%20Here-blue?style=for-the-badge"> </a> </p>

📌 The demo showcases real attack simulations, automatic redirection, and attacker interaction with honeypots.