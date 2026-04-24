# Phase 3 -- Detection and Monitoring

## 1. Introduction

Phase 3 completes the SOC laboratory project by deploying Splunk as a centralized Security Information and Event Management (SIEM) platform. Building on the infrastructure established in Phase 1 and the attack simulation performed in Phase 2, this phase focuses on centralizing log collection, engineering detection rules, configuring alerts, and building a monitoring dashboard.

The primary objective is to demonstrate that SSH brute-force activity — previously detected only through manual `journalctl` queries — can be identified, alerted on, and visualized automatically through a SIEM pipeline. This transforms the lab from a manual log review exercise into a functional detection and monitoring environment.

---

## 2. Architecture Overview

### Updated Lab Topology

The Phase 3 architecture extends the existing three-node lab by introducing Splunk components across two of the virtual machines.

| Machine | OS | Role | IP Address | Splunk Component |
|---|---|---|---|---|
| Kali-SOC | Kali Linux | Attacker | 192.168.11.129 | None |
| Ubuntu-SOC | Ubuntu Server 24.04 | Target / Log Source / SIEM Host | 192.168.11.128 | Splunk Enterprise |

### Data Flow

The log collection pipeline operates as follows:

1. Kali Linux generates SSH authentication traffic (both legitimate and brute-force) directed at the Ubuntu Server.
2. The Ubuntu Server records all SSH authentication events in the local `journald` logging subsystem.
3. Splunk Enterprise, installed on the Ubuntu Server, monitors the SSH log source (`/var/log/auth.log`) and ingests events in near real-time.
4. Splunk indexes the ingested log data and makes it available for search, alerting, and dashboard visualization.

### Data Flow Diagram

```
Kali Linux (Attacker)
  |
  |--- SSH traffic (port 22) --->
  |
Ubuntu Server (Target + SIEM)
  |
  |--- journald records SSH events
  |--- /var/log/auth.log
  |--- Splunk monitors auth.log
  |
  v
Splunk Web Interface
  |--- Search & Investigation
  |--- Alert Rules
  |--- Dashboard Panels
```

---

## 3. Log Collection Setup

### 3.1 Splunk Enterprise Installation

Splunk Enterprise was installed on the Ubuntu Server using the `.deb` package. The installation was performed under the default configuration with the web interface accessible on port 8000.

Installation steps:

```bash
sudo dpkg -i splunk-9.x.x-linux-amd64.deb
sudo /opt/splunk/bin/splunk start --accept-license
sudo /opt/splunk/bin/splunk enable boot-start
```

After installation, the Splunk Web interface was accessible at `http://192.168.11.128:8000`.

### 3.2 Log Source Configuration

SSH authentication logs on Ubuntu Server are written to `/var/log/auth.log` by the `rsyslog` service. Splunk was configured to monitor this file as a data input.

Configuration was performed through the Splunk Web interface:

1. Navigate to **Settings > Data Inputs > Files & Directories**.
2. Add a new monitored file: `/var/log/auth.log`.
3. Set the source type to `linux_secure`.
4. Assign the index to `main`.

The equivalent configuration entry in `inputs.conf`:

```ini
[monitor:///var/log/auth.log]
disabled = false
sourcetype = linux_secure
index = main
```

### 3.3 Log Verification

After configuring the data input, log ingestion was verified by running a basic search query in Splunk:

```spl
index=main sourcetype=linux_secure
```

This confirmed that SSH authentication events — including both `Failed password` and `Accepted password` entries — were being indexed and were searchable through the Splunk interface.

---

## 4. Detection Engineering

### 4.1 Detection Objective

The primary detection goal for Phase 3 is to identify SSH brute-force activity. In the context of this lab, brute-force activity is defined as five or more failed SSH authentication attempts from a single source IP address within a 10-minute window.

### 4.2 Detection Logic

The detection is based on filtering SSH log events that contain the string `Failed password`, extracting the source IP address from the log entry, and counting the number of failures per IP within the defined time window.

The threshold of five attempts was selected based on the following reasoning:

- The default OpenSSH configuration allows three password attempts per connection before disconnecting the client.
- A legitimate user who mistyped their password would typically produce one to three failed attempts before either succeeding or abandoning the connection.
- Five or more failures within a short window strongly indicates automated or deliberate password guessing activity.

### 4.3 SPL Query

The following Splunk Processing Language (SPL) query implements the detection logic:

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count as failed_attempts by src_ip
| where failed_attempts >= 5
```

Query breakdown:

| Component | Purpose |
|---|---|
| `index=main sourcetype=linux_secure "Failed password"` | Filters events to only SSH authentication failures |
| `rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"` | Extracts the source IP address from the log message using a regular expression |
| `stats count as failed_attempts by src_ip` | Counts the number of failed attempts grouped by each unique source IP |
| `where failed_attempts >= 5` | Filters to only show IPs that exceed the brute-force threshold |

---

## 5. Alert Configuration

### 5.1 Alert Rule

A saved search was created in Splunk with the following parameters:

| Parameter | Value |
|---|---|
| Alert Name | SSH Brute-Force Detection |
| Search Query | See SPL query in Section 4.3 |
| Time Range | Last 10 minutes |
| Schedule | Every 5 minutes |
| Trigger Condition | Number of results greater than 0 |
| Alert Action | Log event (internal Splunk log) |
| Severity | High |

### 5.2 Alert Configuration Steps

1. Execute the SPL query in the Splunk search bar.
2. Select **Save As > Alert**.
3. Configure the alert name, schedule, and trigger condition as specified above.
4. Set the alert action to log the event in the Splunk internal triggered alerts list.
5. Save the alert.

### 5.3 Alert Validation

To validate the alert, the Phase 2 attack simulation was repeated from the Kali Linux machine:

```bash
hydra -l socadmin -P passwords.txt ssh://192.168.11.128
```

After the Hydra scan completed and the alert schedule interval elapsed, the triggered alert appeared in **Activity > Triggered Alerts** within the Splunk Web interface. The alert correctly identified `192.168.11.129` as the source IP exceeding the brute-force threshold.

---

## 6. Attack Simulation

### 6.1 Re-Execution of Brute-Force Attack

The same attack methodology from Phase 2 was repeated to generate fresh authentication events for Splunk to ingest and analyze. The attack was executed from the Kali Linux machine using Hydra.

Command:

```bash
hydra -l socadmin -P passwords.txt ssh://192.168.11.128
```

Wordlist contents (`passwords.txt`):

```text
123456
password
admin
root
toor
kali
ubuntu
socadmin
```

### 6.2 Manual SSH Attempts

In addition to the automated Hydra attack, manual SSH login attempts were performed from Kali Linux to simulate realistic attacker behavior:

```bash
ssh fakeuser@192.168.11.128
```

Each manual attempt resulted in password prompts followed by `Permission denied, please try again.` messages, ultimately ending with `Permission denied (publickey,password).` after the maximum number of attempts per connection.

### 6.3 Log Confirmation

After the attack simulation, the following Splunk query confirmed that the events were ingested:

```spl
index=main sourcetype=linux_secure "Failed password"
| stats count by host
```

The query returned a count consistent with the number of Hydra attempts plus the manual SSH connection attempts.

---

## 7. Dashboard Visualization

### 7.1 Dashboard Overview

A Splunk dashboard was created to provide a centralized view of SSH authentication activity across the lab environment. The dashboard consolidates key metrics into a single interface, enabling rapid identification of anomalous authentication patterns.

### 7.2 Dashboard Panels

The dashboard consists of four panels:

**Panel 1: Total Failed SSH Logins**

Displays the total count of failed SSH authentication attempts across all source IPs.

```spl
index=main sourcetype=linux_secure "Failed password"
| stats count as total_failed
```

**Panel 2: Failed Logins by Source IP**

Displays a table showing the number of failed login attempts grouped by source IP address, sorted in descending order.

```spl
index=main sourcetype=linux_secure "Failed password"
| rex "from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| stats count as failed_attempts by src_ip
| sort -failed_attempts
```

**Panel 3: Successful Logins**

Displays a table of all successful SSH authentication events, including the username, source IP, and timestamp.

```spl
index=main sourcetype=linux_secure "Accepted password"
| rex "for (?<username>\w+) from (?<src_ip>\d+\.\d+\.\d+\.\d+)"
| table _time, username, src_ip
| sort -_time
```

**Panel 4: Authentication Events Over Time**

Displays a time chart showing the volume of both failed and successful authentication events over time, enabling visual identification of attack windows.

```spl
index=main sourcetype=linux_secure ("Failed password" OR "Accepted password")
| eval event_type=if(like(_raw, "%Failed password%"), "Failed", "Accepted")
| timechart count by event_type
```

### 7.3 Dashboard Creation Steps

1. Navigate to **Dashboards > Create New Dashboard**.
2. Set the dashboard title to "SOC Lab - SSH Authentication Monitor".
3. Add each panel using the SPL queries documented above.
4. Select appropriate visualization types (single value for Panel 1, table for Panels 2 and 3, line chart for Panel 4).
5. Save the dashboard.

---

## 8. Results and Observations

### 8.1 Key Findings

The Phase 3 implementation produced the following results:

1. **Log ingestion confirmed.** Splunk successfully ingested SSH authentication events from `/var/log/auth.log` in near real-time. Events appeared in search results within seconds of being generated.

2. **Detection rule validated.** The brute-force detection SPL query correctly identified `192.168.11.129` (Kali Linux) as the sole source IP exceeding the five-attempt threshold. No false positives were observed during testing.

3. **Alert triggered successfully.** The saved search alert fired after the Hydra brute-force scan was re-executed, confirming that the alerting pipeline functions correctly under the defined schedule and threshold.

4. **Dashboard operational.** All four dashboard panels rendered correctly and displayed data consistent with the attack simulation. The time chart panel clearly showed the spike in failed authentication events during the Hydra scan window.

### 8.2 Attacker Profile (Based on Splunk Analysis)

| Attribute | Value |
|---|---|
| Source IP | 192.168.11.129 |
| Target Service | SSH (port 22) |
| Target Account | socadmin |
| Failed Attempts | 7+ (across Hydra and manual attempts) |
| Successful Login | 1 (after password was set to a value in the wordlist) |
| Attack Tool | Hydra (automated), manual SSH client |

---

## 9. Limitations

The following limitations are acknowledged for the Phase 3 implementation:

1. **Single log source.** Only SSH authentication logs from the Ubuntu Server are ingested. A production SOC environment would collect logs from multiple sources including firewalls, endpoints, DNS servers, and application logs.

2. **Local SIEM deployment.** Splunk Enterprise is installed on the same machine that serves as the attack target. In a production architecture, the SIEM would be deployed on a dedicated server or cluster to ensure separation of concerns and adequate resource allocation.

3. **No log forwarding infrastructure.** The Splunk Universal Forwarder was not deployed. Log collection relies on local file monitoring rather than agent-based forwarding from remote hosts.

4. **Static threshold.** The brute-force detection threshold (five failed attempts) is hardcoded. A production implementation would incorporate adaptive thresholds or baseline deviation analysis.

5. **No incident response workflow.** The alert action is limited to internal Splunk logging. No external notification channels (email, webhook, ticketing system) are configured.

6. **Lab-scale data volume.** The volume of log data generated in this lab is minimal compared to production environments. The detection logic has not been tested under high-volume conditions.

---

## 10. Conclusion

Phase 3 successfully demonstrates the deployment and configuration of Splunk Enterprise as a SIEM platform within the SOC laboratory environment. The phase achieved its core objectives: centralizing log collection from the Ubuntu Server, engineering a detection rule for SSH brute-force activity, configuring an automated alert, and building a monitoring dashboard.

The end-to-end pipeline — from attack generation on Kali Linux, through log recording on Ubuntu Server, to detection and visualization in Splunk — validates the fundamental workflow of a Security Operations Center. While the implementation operates at lab scale with acknowledged limitations, it demonstrates the core principles of log management, detection engineering, and security monitoring that are foundational to SOC operations.

The SOC Lab project, across all three phases, progresses from raw infrastructure deployment through active attack simulation to automated detection and monitoring, mirroring the structured approach used in professional security engineering environments.
