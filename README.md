# SOC Laboratory Project

A structured multi-phase Security Operations Center (SOC) laboratory designed to simulate real-world security monitoring, detection, and incident response workflows within a controlled virtualized environment.

---

## Project Overview

This project documents the design and implementation of a three-node SOC lab environment built using VMware virtualization. The objective is to establish a stable infrastructure baseline before progressively integrating security monitoring, logging, and defensive controls in subsequent phases.

The lab consists of:

- **Kali-SOC** – Attacker / Security Analyst Node  
- **Windows-SOC** – Endpoint / Victim Machine  
- **Ubuntu-SOC** – SOC Server (Future SIEM Node)

Phase 1 focuses strictly on infrastructure deployment, network validation, and baseline snapshot management.

---

## Architecture Summary

- Hypervisor: VMware Workstation  
- Network Model: NAT  
- IP Assignment: DHCP  
- Connectivity: Internet + Inter-VM validated  
- Resource Allocation: 4GB RAM, 2 CPU cores, 40GB storage per VM  

---

## Architecture Diagram

![SOC Architecture](diagrams/soc-architecture.png)

---

## Phase Structure

| Phase | Description | Status |
|-------|-------------|--------|
| Phase 1 | Infrastructure Deployment & Baseline Configuration | Completed |
| Phase 2 | Security Tooling & Log Aggregation | Planned |
| Phase 3 | Detection & Monitoring Integration | Planned |
| Phase 4 | Attack Simulation & Incident Analysis | Planned |

Detailed documentation for Phase 1 is available here:

👉 `docs/phase-1.md`

---

## Design Philosophy

The project follows a phased engineering approach:

- Separate infrastructure from security configuration  
- Establish validated baselines before introducing complexity  
- Preserve rollback states using structured snapshots  
- Maintain clean documentation for each implementation stage  

---

## Current Status

Phase 1 has been successfully completed.  
All virtual machines are operational, network-validated, updated, and preserved through named snapshots.

Future phases will introduce SIEM integration, log forwarding, detection rules, and adversary simulation workflows.

---

## Repository Structure

```text
SOC-LAB-DOCUMENTATION/
├── README.md                        # Project overview, architecture summary, and phase tracker
├── docs/
│   └── phase-1.md                   # Detailed documentation for Phase 1 infrastructure deployment
└── diagrams/
    └── soc-architecture.png         # Visual diagram of the three-node SOC lab topology
```
