<div align="center">

# ☁️ Cloud Honeypot on OpenStack

<p>
  <img src="https://img.shields.io/badge/OpenStack-Yoga-red?logo=openstack&logoColor=white&style=for-the-badge">
  <img src="https://img.shields.io/badge/IDS-Suricata-ff6600?logo=suricata&logoColor=white&style=for-the-badge">
  <img src="https://img.shields.io/badge/SIEM-ELK%20Stack-005571?logo=elastic&logoColor=white&style=for-the-badge">
  <img src="https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white&style=for-the-badge">
</p>
<p>
  <img src="https://img.shields.io/badge/Status-Lab%20Environment-lightgrey?style=for-the-badge">
  <img src="https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge">
</p>

### 🛡️ Cloud-Based Proactive Defense using IDS + Honeypot + SIEM

**NT524 – Cloud Computing Architecture & Security**

**Team 8:** Huỳnh Đăng Khoa · Khiếu Bảo Lâm

</div>

---

## 📖 Table of Contents

- [🌟 Introduction](#-introduction)
- [🎯 Objectives](#-objectives)
- [✨ Key Features](#-key-features)
- [🏗️ Architecture Overview](#-architecture-overview)
- [🌐 Network Topology](#-network-topology)
- [🔄 Data Flow](#-data-flow)
- [🧠 Decision Logic](#-decision-logic)
- [🧩 Core Components](#-core-components)
- [🧪 Honeypot Design](#-honeypot-design)
- [⚙️ Automatic Redirection](#-automatic-redirection-mechanism)
- [📊 SIEM & Log Processing](#-siem--log-processing)
- [🎬 Demo Scenarios](#-demo-scenarios)
- [📈 Results](#-results)

---

## 🌟 Introduction

In modern cloud environments, services such as **SSH**, **web applications**, **object storage (S3)**, and **databases (RDS)** are frequent targets of cyberattacks. These services are often exposed due to misconfiguration, weak credentials, or improper access control.

> 💡 Studies show that publicly exposed cloud resources can be **discovered and exploited within minutes** using automated scanning tools.

Traditional defensive mechanisms such as firewalls or passive intrusion detection systems are no longer sufficient to handle evolving threats. Instead of only blocking attackers, a more effective approach is to **observe and understand their behavior**.

This project proposes a **proactive defense architecture** by combining intrusion detection with a honeypot system. When suspicious activity is detected, the attacker is **not blocked immediately** but is transparently redirected into a controlled, simulated environment.

This approach allows the system to:
- 🔒 Protect real services from compromise
- 🧠 Collect valuable threat intelligence
- 🔬 Analyze attacker techniques in depth

---

## 🎯 Objectives

| # | Goal |
|---|------|
| 🔍 | Detect attacks in real time using IDS |
| 🔄 | Automatically redirect attackers |
| 🧪 | Simulate real cloud services (SSH, HTTP, S3, MySQL) |
| 📊 | Centralize logs using SIEM |
| 🏷️ | Map attacks to MITRE ATT&CK |

---

## ✨ Key Features

- ⚡ **Real-time detection** with Suricata IDS
- 🔀 **Automatic DNAT redirection** — transparent to attackers
- 🎭 **Multi-service honeypot** environment (SSH, HTTP, S3, MySQL)
- 📦 **Centralized logging** via ELK Stack
- 🌍 **GeoIP-based** attacker tracking
- 🗺️ **MITRE ATT&CK** framework mapping
- ☁️ **Fully deployed** on OpenStack

---

## 🏗️ Architecture Overview

<div align="center">
  <img src="architecture.png" width="80%" alt="Architecture Diagram">
</div>

### System Layers

| Layer | Description |
|-------|-------------|
| 🎛️ **Controller** | Entry point, NAT, redirection logic |
| 🔬 **Analyzer** | Traffic inspection & threat detection |
| 🍯 **Honeypot Zone** | Isolated fake cloud services |
| 📊 **SIEM Layer** | Centralized logging & visualization |

---

## 🌐 Network Topology

<div align="center">
  <img src="topology.png" width="80%" alt="Network Topology">
</div>

### Network Segments

| Network | Role |
|---------|------|
| 🌐 **External Network** | Public internet access |
| 🔧 **Management Network** | Control plane traffic (SSH) |
| 🍯 **Honeypot Network** | Isolated fake service zone |
| 🔒 **Real App Network** | Protected production services |

---

## 🔄 Data Flow

```
Internet → Controller → Analyzer → Detection → Redirect → Honeypot → ELK
```

### Step-by-Step Flow

```
1. 🌐  Traffic enters Controller
2. 🪞  Mirrored to Analyzer
3. 🔍  Suricata analyzes packets
4. 🚨  Attack detected → alert generated
5. 🐍  Python script processes the alert
6. 🔀  Controller updates NAT rules
7. 🍯  Attacker redirected to Honeypot
8. 📦  All logs shipped to ELK
```

---

## 🧠 Decision Logic

| Traffic Type | Action |
|--------------|--------|
| ✅ Legitimate | Forward to real service |
| 🚨 Malicious | Redirect to honeypot |

### Strategy

> **Real services** run on **non-standard ports**
> **Honeypots** listen on **default ports**

```
👉 Attackers are naturally trapped — no magic needed.
```

---

## 🧩 Core Components

### 🔷 Suricata IDS
- Deep packet inspection (DPI)
- Detects brute-force, port scanning, and exploitation attempts
- Generates structured EVE JSON alerts

### 🔷 Python Orchestrator
- Reads IDS logs in real-time using `tail -f` or inotify
- Extracts attacker IP from alert events
- Triggers DNAT redirection via API or shell commands

### 🔷 Controller Node
- Manages OpenStack router / iptables NAT rules
- Controls all inbound/outbound traffic flow
- Single point of redirection enforcement

---

## 🧪 Honeypot Design

| Service | Description |
|---------|-------------|
| 🔐 **SSH** | Fake login shell + full command logging |
| 🌐 **HTTP** | Fake web server with trap endpoints |
| ☁️ **S3 (MinIO)** | Fake cloud object storage |
| 🗄️ **MySQL** | Fake database with query capture |

---

## ⚙️ Automatic Redirection Mechanism

### Workflow

```
📡 Monitor IDS logs
    ↓
🔎 Extract attacker IP from alert
    ↓
📋 Check for existing DNAT rules
    ↓
✏️  Apply new DNAT rule via iptables/OpenStack
    ↓
🍯 All future traffic → Honeypot
```

| Property | Value |
|----------|-------|
| ✅ Automation | Fully automated |
| ⚡ Latency | < 2 seconds |
| 🔁 Persistence | Rules survive session |

---

## 📊 SIEM & Log Processing

### Pipeline

```
Filebeat → Logstash → Elasticsearch → Kibana
```

### Features

- 📦 **Centralized logging** from all honeypot nodes
- 🗺️ **MITRE ATT&CK** technique mapping per event
- 📊 **Real-time Kibana dashboards** with GeoIP visualization
- 🔎 **Full-text search** across 50,000+ events

---

## 🎬 Demo Scenarios

> **Full flow:** 🔁 Detection → Redirection → Deception → Logging → Visualization

---

### 🔐 SSH Brute-force Attack

1. Attacker runs automated brute-force tool
2. Suricata detects abnormal login attempts
3. Python script triggers DNAT redirection
4. Attacker is silently moved to SSH honeypot

**Result:**
- Fake login succeeds (attacker thinks they're in)
- All commands executed are captured and logged

---

### 🌐 HTTP Reconnaissance

1. Attacker scans endpoints (`/admin`, `/login`, `/.env`)
2. Detected as web reconnaissance / directory traversal
3. Redirected to fake web server

**Result:**
- Full request payload + User-Agent logged
- Attacker explores decoy endpoints

---

### ☁️ S3 / Object Storage Attack

1. Attacker scans for open MinIO buckets
2. Attempts to list or access objects

**Result:**
- Redirected to fake S3 instance
- All upload/download/list operations captured

---

### 🗄️ MySQL Attack

1. Attacker connects to port `3306`
2. Attempts authentication + SQL queries

**Result:**
- Redirected to fake database
- All SQL queries captured and indexed in ELK

---

### 🎥 Demo Video

<div align="center">

[![Watch Full Demo](https://img.shields.io/badge/🎬%20Watch%20Full%20Demo-Drive-blue?style=for-the-badge&logo=google-drive)](https://drive.google.com/drive/folders/1-ASvfGYQs8oLcXdMXsbLzcvJfMdoVuuf?usp=drive_link)

</div>

---

## 📈 Results

| Metric | Value |
|--------|-------|
| ⏱️ Redirection Latency | **< 2 seconds** |
| 📊 Events Processed | **50,000+** |
| 🎯 Detection Accuracy | **High** |
| 📊 Visualization | **Kibana Dashboards** |

---

<div align="center">

**NT524 · Cloud Computing Architecture & Security**
**Team 8 · 2024**

</div>
