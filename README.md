# SOC Lab Project – End-to-End Threat Detection

A structured, multi-phase Security Operations Center (SOC) lab project designed to simulate real-world cyber attacks and SOC detection workflows in a controlled virtualized environment.

---

## Overview

This repository documents a SOC lab environment built using VMware Workstation. The objective is to establish a stable infrastructure baseline and then progressively implement logging, monitoring, and detection activities through structured phases.

The lab consists of:

- **Kali-SOC** – Attacker / Security Analyst Node  
- **Windows-SOC** – Endpoint / Victim Machine  
- **Ubuntu-SOC** – SOC Server (Future SIEM Node)

Phase 1 focuses strictly on infrastructure deployment, network validation, and baseline snapshot management.

---

## Architecture

Simple flow (Phase 2):

Kali Linux (Attacker) → Ubuntu Server (Target) → Logs → Analysis

### Architecture Summary

- Hypervisor: VMware Workstation  
- Network Model: NAT  
- IP Assignment: DHCP  
- Connectivity: Internet + Inter-VM validated  
- Resource Allocation: 4GB RAM, 2 CPU cores, 40GB storage per VM  


### Architecture Diagram

![SOC Architecture](diagrams/soc-architecture.png)

---

## Lab Setup

| Item | Value |
|---|---|
| Virtualization | VMware Workstation |
| Network | NAT |
| Kali IP | 192.168.11.129 |
| Ubuntu IP | 192.168.11.128 |

## Phase Documentation

- Phase 1: [docs/phase-1.md](docs/phase-1.md)
- Phase 2: [docs/phase-2.md](docs/phase-2.md)

## Phase 2 – SIEM Setup and Log Collection

### Reconnaissance (Nmap)

Nmap was used from Kali Linux to identify exposed services on the Ubuntu Server target.

![Nmap scan showing open SSH service discovery](documentation/images/phase-2/nmap_scan.png)

Caption: Nmap: open SSH port discovery.

### Brute Force Attack (Hydra)

Hydra was used to simulate SSH password guessing against the Ubuntu Server target account.

![Hydra output for SSH brute-force attempt](documentation/images/phase-2/hydra_attack.png)

Caption: Hydra: brute-force attack attempt.

### Failed Login Detection

Failed SSH authentication attempts were identified on Ubuntu Server using `journalctl` queries against the SSH service logs.

![Journald output showing multiple failed SSH logins](documentation/images/phase-2/failed_logs.png)

Caption: Failed logs: multiple failed login attempts.

### Successful Compromise

After setting a known password for the target account and re-running Hydra, a successful SSH login was confirmed in the Ubuntu SSH logs.

![Journald output showing accepted SSH login](documentation/images/phase-2/successful_login.png)

Caption: Successful login: confirmed system compromise.

## SOC Analysis

- Brute-force detection: multiple failed SSH authentication attempts were observed and counted.
- Attacker IP identification: log review identified a single source IP (192.168.11.129) for the failed attempts.
- Successful compromise detection: an `Accepted` SSH log entry confirmed successful authentication.

## Key Learnings

- Validated connectivity and attack paths between lab hosts.
- Used reconnaissance to identify the exposed service and its version.
- Generated authentication events and verified that logs were recorded on the target system.
- Performed basic log triage to quantify failed attempts and identify the source of activity.

## Future Work

- SIEM integration
- Alerts
- Dashboards

## Project Status

| Phase | Status |
|---|---|
| Phase 1 | Completed |
| Phase 2 | Completed |
| Phase 3 | Upcoming |

---

## Phase Structure

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1 | Infrastructure Deployment & Baseline Configuration | Completed |
| Phase 2 | Security Tooling & Log Aggregation | Completed |
| Phase 3 | Detection & Monitoring Integration | Planned |
| Phase 4 | Attack Simulation & Incident Analysis | Planned |

---

## Design Philosophy

The project follows a phased engineering approach:

- Separate infrastructure from security configuration  
- Establish validated baselines before introducing complexity  
- Preserve rollback states using structured snapshots  
- Maintain clean documentation for each implementation stage  

---

## Current Status

Phase 1 (infrastructure) and Phase 2 (SIEM setup and log collection) have been successfully completed.  
The lab environment is operational, network-validated, and is now capable of local log ingestion and basic monitoring through manual log review.

Future phases will focus on detection, analysis, and attack simulation.

---

## Repository Structure

```text
SOC-LAB-DOCUMENTATION/
├── README.md                        # Project overview, architecture summary, and phase tracker
├── documentation/
│   └── images/
│       └── phase-2/                 # Phase 2 screenshots for README
├── docs/
│   ├── phase-1.md                   # Detailed documentation for Phase 1 infrastructure deployment
│   └── phase-2.md                   # Detailed documentation for Phase 2 – SIEM setup and log collection
└── diagrams/
    └── soc-architecture.png         # Visual diagram of the three-node SOC lab topology
```
