Narito ang isang pormang-GitHub na `README.md` na hango sa mga clean at professional homelab repository layout (tulad ng mga estruktura ng mga n8n/automation repos), na naka-angkop nang eksakto sa ating Part 2 case study:

```markdown
# 🤖 Mini Self-Healing Infrastructure & Dynamic AIOps Remediation (Part 2)

> Part of the **Pinoy Tech Share** technical case study series on building local, privacy-first, self-healing enterprise automation.

[![n8n](https://img.shields.io/badge/n8n-Orchestration-orange?style=flat-square&logo=n8n)](https://n8n.io)
[![Zabbix](https://img.shields.io/badge/Zabbix-Monitoring-red?style=flat-square&logo=zabbix)](https://www.zabbix.com)
[![Ollama](https://img.shields.io/badge/Ollama-Local%20AI-blue?style=flat-square)](https://ollama.com)
[![Proxmox](https://img.shields.io/badge/Proxmox-LXC-orange?style=flat-square&logo=proxmox)](https://www.proxmox.com)

---

## 📌 Overview

This repository and documentation cover **Part 2** of our local AIOps automation pipeline. Moving beyond passive logging and security digests (Part 1), this implementation bridges the gap between monitoring telemetry and active, closed-loop system recovery using local artificial intelligence and n8n orchestration.

---

## 🏗️ Architecture Flow


```

[ Zabbix (Telemetry) ]
│ (HTTP Webhook POST)
▼
[ n8n on Proxmox LXC ] ──> [ Ollama (Local LLM) ] ──> [ Safe Execution Layer (pct start/stop) ]

```

---

## ⚙️ Component Stack

*   **Telemetry & Event Source (Zabbix):** Monitors resource health, container states, and performance thresholds, dispatching rich JSON payloads via webhooks upon anomaly detection.
*   **Orchestration Engine (n8n on Proxmox LXC):** Hosted locally inside a dedicated Linux Container with explicit `systemd` network bindings (`0.0.0.0`) and static local routing (`[Local IP of n8n]:5678`) for seamless internal communication.
*   **Cognitive Processing (Local LLM via Ollama):** Evaluates unstructured alert messages locally, translating raw error context into clean, structured execution parameters without recurring cloud API costs.
*   **Execution Layer:** Interfaces directly with host environments or container APIs to execute corrective routines.

---

## 🛡️ Safety & Governance: The "Start/Stop" Rule

To mitigate the inherent risks of small-parameter model hallucinations and unintended system mutations, this setup strictly enforces:

*   **Non-Destructive Operations Only:** The AI's operational scope is locked exclusively to basic `start` or `stop` lifecycle routines for non-critical LXC containers or VMs (e.g., executing `pct start 108`).
*   **Zero Configuration Mutation:** Commands that modify system configurations, storage pools, or network bridges are blocked entirely during initial test phases.

---

## 🚀 Implementation Workflow

1.  **Webhook Ingestion:** When a service crashes or a container stops unexpectedly, Zabbix fires a structured POST request containing event metadata and error logs to the n8n webhook endpoint.
2.  **Cognitive Root-Cause Parsing:** The payload is passed to the local LLM node. The model analyzes the error signature and outputs a deterministic JSON object containing the exact remediation command required (e.g., `{"actionCommand": "pct start 108"}`).
3.  **Command Validation & Execution:** The orchestration workflow extracts the payload dynamically and passes it to the command execution module, closing the incident recovery cycle.

---

## 📖 Series Navigation

*   **Part 1:** Zero-Cloud AIOps & Security Digest Engine *(Passive telemetry & local analysis)*
*   **Part 2:** [Mini Self-Healing Infrastructure & Dynamic AIOps Remediation](https://pinoytechshare.blogspot.com/2026/09/part-2-mini-self-healing-infrastructure.html) *(Active closed-loop execution)*
*   **Part 3:** Human-in-the-Loop (HITL) Governance Gates *(Upcoming)*

---

## 📄 License

Shared under the [MIT License](LICENSE) for educational and homelab engineering reference.

```
