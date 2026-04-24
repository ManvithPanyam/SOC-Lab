# SOC Laboratory Project
## Phase 1 – Infrastructure Deployment and Baseline Configuration

**Environment Type:** Virtualized SOC Laboratory  
**Deployment Model:** Three-Node NAT-Based Architecture  
**Primary Objective:** Establish validated infrastructure baseline prior to security tooling integration  
**Status:** Completed

---

## Phase 1 Overview

Phase 1 establishes the foundational infrastructure of a controlled Security Operations Centre (SOC) laboratory environment. The objective of this phase is to deploy, configure, and validate a three-node virtual architecture that will serve as the operational baseline for subsequent security tooling and monitoring integration.

The lab simulates a practical SOC topology consisting of:

- A security analyst / attacker workstation (Kali-SOC)
- A Windows endpoint representing a monitored asset
- A Linux-based SOC server designated for future SIEM deployment

This phase focuses strictly on infrastructure readiness, network validation, controlled baseline configuration, and snapshot preservation. No security monitoring, log aggregation, intrusion detection, or system hardening mechanisms are introduced at this stage.

The outcome of Phase 1 is a stable, validated infrastructure layer upon which future defensive and monitoring capabilities can be systematically implemented.

---

## Objectives

- Deploy three role-based virtual machines representing core SOC lab components.
- Allocate consistent and controlled hardware resources across all nodes.
- Implement NAT-based networking with verified internal and external connectivity.
- Validate inter-node communication within the isolated subnet.
- Enable remote administration capability via OpenSSH on the Ubuntu server.
- Apply baseline system updates to ensure package consistency.
- Preserve validated system states through structured snapshot creation.

---

## Lab Environment Details

| Parameter              | Value                          |
|------------------------|-------------------------------|
| Hypervisor             | VMware Workstation             |
| Network Model          | NAT (Network Address Translation) |
| IP Assignment          | DHCP                           |
| Internet Connectivity  | Verified                       |
| Inter-VM Communication | Verified via ICMP ping         |
| Host Operating System  | Windows                        |
| Lab Phase              | Phase 1 – Baseline Deployment  |

---

## Virtual Machine Configuration

Three virtual machines were deployed, each assigned a specific functional role within the lab.

### Kali-SOC – Attacker / Security Analyst Node

| Parameter     | Value             |
|---------------|-------------------|
| Operating System | Kali Linux     |
| Role          | Attacker / Security Analyst |
| RAM           | 4 GB              |
| CPU Cores     | 2                 |
| Storage       | 40 GB             |
| Network       | VMware NAT        |

### Windows-SOC – Endpoint / Victim Machine

| Parameter     | Value                  |
|---------------|------------------------|
| Operating System | Windows             |
| Role          | Endpoint / Victim Node |
| RAM           | 4 GB                   |
| CPU Cores     | 2                      |
| Storage       | 40 GB                  |
| Network       | VMware NAT             |

### Ubuntu-SOC – SOC Server (Future SIEM Node)

| Parameter     | Value                          |
|---------------|-------------------------------|
| Operating System | Ubuntu Server 24.04 LTS    |
| Role          | SOC Server / Future SIEM Node  |
| RAM           | 4 GB                           |
| CPU Cores     | 2                              |
| Storage       | 40 GB                          |
| Network       | VMware NAT                     |

---

## Network Configuration

All three virtual machines are connected through VMware's NAT network adapter. This configuration allows each VM to share the host machine's IP address for outbound internet traffic while remaining logically isolated from the external network. DHCP is handled by VMware's virtual DHCP server, which dynamically assigns IP addresses to each VM within the NAT subnet.

Internet connectivity was confirmed on all nodes following deployment. Inter-VM communication was validated by performing ICMP ping tests between each machine, confirming that all nodes can reach one another within the NAT subnet.

```bash
# Example: Verify connectivity from Kali-SOC to Ubuntu-SOC
ping <Ubuntu-SOC-IP>

# Example: Verify connectivity from Kali-SOC to Windows-SOC
ping <Windows-SOC-IP>
```

> **Note:** IP addresses are dynamically assigned via DHCP and may change between sessions. Static IP assignment is not configured in this phase.

---

## SSH Installation & Configuration

OpenSSH Server was enabled on Ubuntu-SOC during the operating system installation process. No post-installation modifications were made to the SSH configuration.

| Parameter           | Value                        |
|---------------------|------------------------------|
| SSH Server          | OpenSSH (installed at setup) |
| Default Port        | 22                           |
| Root Login          | Disabled (Ubuntu default)    |
| Configuration File  | `/etc/ssh/sshd_config` (unmodified) |
| UFW / Firewall      | Not configured               |

SSH connectivity can be tested using the following command from any node within the NAT subnet:

```bash
ssh <username>@<Ubuntu-SOC-IP>
```

The SSH service status can be verified on the Ubuntu-SOC node with:

```bash
sudo systemctl status ssh
```

No hardening measures, port changes, or access restrictions were applied to the SSH service during this phase. These configurations are deferred to a subsequent phase.

---

## Security Posture Assessment (Baseline)

At the conclusion of Phase 1, the environment reflects a controlled post-installation baseline state. No active hardening measures, firewall rules, or security monitoring controls have been introduced. This deliberate decision ensures that future security configurations can be applied incrementally and documented with clarity.

The following table documents the configuration status of relevant security controls.

| Control                        | Status                                      |
|-------------------------------|---------------------------------------------|
| Root SSH Login                | Disabled (Ubuntu installation default)      |
| UFW (Uncomplicated Firewall)  | Not configured                              |
| SSH Port                      | 22 (default, unmodified)                    |
| SSH Hardening                 | Not applied                                 |
| System Updates                | Applied (`apt update && apt upgrade`)        |
| SIEM / Log Aggregation        | Not deployed                                |
| Intrusion Detection           | Not deployed                                |
| Endpoint Security (Windows)   | Not configured                              |

The baseline security posture is intentionally minimal. This is expected for Phase 1, where the primary goal is infrastructure deployment rather than security hardening. Hardening and monitoring controls will be introduced in later phases.

---

## Snapshot Strategy

A structured snapshot methodology was implemented to preserve validated system states at defined configuration milestones. This approach ensures rapid recovery capability, reduces rebuild time in the event of misconfiguration, and maintains phase integrity as the lab evolves.

### Snapshot Checkpoints

| Snapshot Name           | VM           |
|-------------------------|--------------|
| `Kali-Clean-Install`    | Kali-SOC     |
| `Windows-Clean-Install` | Windows-SOC  |
| `Ubuntu-Clean-Base`     | Ubuntu-SOC   |

Snapshots were taken after:

- Operating system installation
- Network validation
- System updates

The final snapshot state reflects all three completed milestones.

---

## Challenges & Troubleshooting

No configuration errors or network inconsistencies were observed during deployment. All validation checks completed successfully.

---

## Learning Outcomes

- Gained practical experience deploying multiple virtual machines within a VMware NAT network topology.
- Understood the role and behaviour of VMware's virtual DHCP server in assigning addresses within a NAT subnet.
- Established familiarity with the Ubuntu Server 24.04 LTS installation process, including the OpenSSH setup option.
- Reinforced the importance of snapshot management as a baseline recovery strategy in lab environments.
- Developed an understanding of the default security posture of freshly installed Linux and Windows systems prior to any hardening.

---

## Conclusion

Phase 1 successfully delivers a validated three-node SOC laboratory architecture. All systems are operational, network-verified, updated, and preserved through structured snapshot management. The infrastructure layer is stable and intentionally minimal, providing a clean baseline for the introduction of monitoring, detection, and hardening controls in subsequent phases.

By isolating infrastructure deployment from security configuration, this phase establishes architectural discipline and documentation clarity — two critical principles in professional security engineering environments.
