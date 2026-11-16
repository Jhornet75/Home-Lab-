<h1 align="center">
A1 | Splunk Enterprise<br>
Deployment and Log Ingestion
</h1>

<h3 align="center">
Cyber Defense and Analysis — CYDA 420
</h3>

<p align="center">
<strong>Infrastructure:</strong> Self-hosted Proxmox Homelab
<br>
<strong>Status:</strong> Operational
</p>

<hr>


<p align="center">
  <img width="169" height="150" alt="download" src="https://github.com/user-attachments/assets/9d241416-52db-9689-ef280bb47965" />
</p>


01. Project Overview

For this project, I deployed Splunk Enterprise within my Proxmox homelab to establish a centralized security monitoring environment.

The purpose of this project was to gain hands-on experience with SIEM deployment, Linux administration, log forwarding, data ingestion, and security event analysis.

Throughout the implementation, I encountered several issues involving network connectivity, installation packages, service availability, and forwarding configurations. This documentation covers the implementation process, the issues I encountered, and how I worked through them.

02. Deployment Results

Component

Result

Splunk Enterprise

Operational

Universal Forwarder

Active

Dedicated indexes

2

Authentication events

5,103

Firewall events

8,984

Total indexed events

14,087

pie showData
    title Indexed Security Events
    "Authentication" : 5103
    "Firewall" : 8984

03. Infrastructure Architecture

flowchart LR
    subgraph Sources["Security Log Sources"]
        DC["Domain Controller Logs"]
        FW["Boundary Firewall Logs"]
    end

    subgraph Proxmox["Proxmox VE"]
        subgraph Ubuntu["Ubuntu Server VM"]
            UF["Universal Forwarder"]
            SE["Splunk Enterprise"]
            AUTH[("cascade_auth")]
            FIRE[("cascade_firewall")]

            UF -->|"TCP 9997"| SE
            SE --> AUTH
            SE --> FIRE
        end
    end

    DC -->|"Monitored File"| UF
    FW -->|"Manual Upload"| SE
    SE -->|"TCP 8000"| WEB["Splunk Web Interface"]

    classDef server fill:#263445,stroke:#64748b,color:#ffffff
    classDef storage fill:#164e63,stroke:#0891b2,color:#ffffff
    classDef source fill:#374151,stroke:#9ca3af,color:#ffffff

    class UF,SE,WEB server
    class AUTH,FIRE storage
    class DC,FW source

Technology Stack

Component

Technology

Virtualization

Proxmox VE

Operating System

Ubuntu Server 24.04

SIEM

Splunk Enterprise

Log Collection

Splunk Universal Forwarder

System Administration

Bash

Search Language

SPL

Network Configuration

Protocol

Port

Purpose

TCP

8000

Splunk Web

TCP

8089

Enterprise management

TCP

8090

Forwarder management

TCP

9997

Log receiving

04. Implementation

Phase 1 — Virtual Machine Deployment

I began by creating a dedicated Ubuntu Server virtual machine within Proxmox. Separating Splunk from the other systems within my homelab provided a dedicated environment for security monitoring.

I verified the network configuration using the Linux terminal.

ip addr

Phase 2 — Splunk Enterprise Installation

I downloaded the Splunk Enterprise installation package and installed it through the Linux terminal.

The following commands represent the installation workflow.

ls -lh ~/splunk*.deb
sudo dpkg -i ~/splunk*.deb

Splunk Enterprise installation directory:

/opt/splunk/
├── bin/
│   └── splunk
├── etc/
│   ├── apps/
│   └── system/
│       └── local/
└── var/
    ├── log/
    └── lib/

Directory

Purpose

bin/

Executables and management commands

etc/apps/

Installed applications

etc/system/local/

Local configuration files

var/log/

Internal Splunk logs

var/lib/

Indexed data and runtime storage

After installation, I started Splunk and configured administrative access.

sudo -u splunk /opt/splunk/bin/splunk start --accept-license

I accessed the web interface through TCP port 8000.

http://<VM-IP>:8000

Phase 3 — Universal Forwarder Installation

I installed Splunk Universal Forwarder on the same Ubuntu virtual machine.

/opt/splunkforwarder/
├── bin/
├── etc/
│   └── system/
│       └── local/
└── var/
    └── log/

The Universal Forwarder was responsible for monitoring and forwarding authentication logs to Splunk Enterprise.

Since both applications were running on the same virtual machine, their management ports needed to be separated.

Phase 4 — Receiving Configuration

Within Splunk Enterprise, I navigated to:

Settings
└── Forwarding and receiving
    └── Configure receiving
        └── TCP 9997

I enabled port 9997 and configured the Universal Forwarder to communicate with the local Splunk Enterprise instance.

sudo /opt/splunkforwarder/bin/splunk \
add forward-server 127.0.0.1:9997

I verified the forwarding connection using:

sudo /opt/splunkforwarder/bin/splunk list forward-server

Successful forwarding configuration:

Active forwards:
    127.0.0.1:9997

Configured but inactive forwards:
    None

Phase 5 — Index Creation

I created two dedicated indexes to separate authentication activity from firewall traffic.

Index

Sourcetype

Purpose

cascade_auth

cascade:dc:security

Authentication monitoring

cascade_firewall

cascade:fw:boundary

Network traffic monitoring

Splunk Web
└── Settings
    └── Indexes
        ├── cascade_auth
        └── cascade_firewall

Phase 6 — Authentication Log Collection

I transferred the domain controller security log to the Ubuntu virtual machine and prepared a dedicated directory for log collection.

/var/log/cascade/
└── cascade_dc01_security.log

Representative file preparation commands:

sudo mkdir -p /var/log/cascade

sudo cp ~/cascade_dc01_security.log \
/var/log/cascade/

sudo chmod 644 \
/var/log/cascade/cascade_dc01_security.log

Next, I configured the Universal Forwarder to monitor the authentication log.

sudo /opt/splunkforwarder/bin/splunk add monitor \
/var/log/cascade/cascade_dc01_security.log \
-index cascade_auth \
-sourcetype cascade:dc:security

Phase 7 — Firewall Log Ingestion

The firewall log was manually uploaded through the Splunk web interface.

Splunk Web
└── Settings
    └── Add Data
        └── Upload
            └── cascade_fw01_boundary.log

Configuration

Value

Source

cascade_fw01_boundary.log

Sourcetype

cascade:fw:boundary

Index

cascade_firewall

Collection Method

Manual upload

This allowed me to work with two different methods of collecting security logs.

05. Search and Verification

Authentication Events

index=cascade_auth
| stats count by source sourcetype

Firewall Events

index=cascade_firewall
| stats count by source sourcetype

Combined Log Source Verification

index=cascade_auth OR index=cascade_firewall
| stats count by index source sourcetype

Collection Workflow

flowchart TD
    A["Collect Log Data"] --> B["Assign Sourcetype"]
    B --> C["Assign Index"]
    C --> D["Search Using SPL"]
    D --> E{"Events Available?"}
    E -->|Yes| F["Verify Event Count"]
    E -->|No| G["Review Input and Service Configuration"]
    G --> A
    F --> H["Security Monitoring"]

Both security datasets were successfully indexed and searchable, establishing a foundation for future security monitoring and threat detection labs.

06. Troubleshooting Journal

This was not a flawless deployment. Several configurations required troubleshooting before I was able to establish a working Splunk environment.

<details> <summary><strong>Issue 01 — Unable to Access Splunk Web</strong></summary>

Problem: The virtual machine was running, but the Splunk web interface was inaccessible.

Investigation: I reviewed the VM's network configuration and discovered that I had incorrectly entered its IP address.

Resolution: Corrected the IP address and restored access.

Lesson Learned: Verify basic network configurations before investigating more complicated application issues.

</details>

<details> <summary><strong>Issue 02 — Splunk Not Running After Restart</strong></summary>

Problem: After restarting the virtual machine, I was unable to access Splunk.

Investigation: I checked whether the Splunk service was running.

sudo /opt/splunk/bin/splunk status

Resolution: Started Splunk and verified application availability.

sudo -u splunk /opt/splunk/bin/splunk start

Lesson Learned: A running virtual machine does not necessarily mean that the applications within it are operational.

</details>

<details> <summary><strong>Issue 03 — Installation Package and Directory Errors</strong></summary>

Problem: During installation, I encountered download errors and messages indicating that files or directories could not be found.

Investigation: I reviewed the package location and installation status.

Resolution: Verified the required installation files and continued with the Universal Forwarder installation.

Lesson Learned: Verify file paths, package availability, and installation directories before executing commands.

</details>

<details> <summary><strong>Issue 04 — Management Port Conflict</strong></summary>

Problem: Splunk Enterprise and the Universal Forwarder were operating on the same virtual machine and required separate management ports.

Resolution: Separated the management port configurations.

Service

Port

Splunk Enterprise

8089

Universal Forwarder

8090

Lesson Learned: Multiple services running on the same system require properly coordinated network configurations.

</details>

<details> <summary><strong>Issue 05 — Forwarding Destination Configured but Inactive</strong></summary>

Problem: The Universal Forwarder was running, but the forwarding destination was inactive.

Investigation: I checked whether TCP port 9997 was listening.

sudo ss -lntp | grep 9997

The command initially returned no output.

Further troubleshooting showed that Splunk Enterprise was not running correctly.

Resolution: Restored Splunk Enterprise availability and verified the receiving port and forwarding connection.

sudo /opt/splunkforwarder/bin/splunk list forward-server

Lesson Learned: A configured forwarding destination does not guarantee that the receiving service is available.

</details>

<details> <summary><strong>Issue 06 — Difficulty Locating Authentication Logs</strong></summary>

Problem: I had difficulty locating the correct domain controller security events within Splunk.

Investigation: I reviewed the monitored file path, source, sourcetype, index, and forwarding configuration.

Resolution: Verified that the authentication dataset was indexed and searchable.

Lesson Learned: Understanding the relationship between sources, sourcetypes, and indexes is essential when troubleshooting data ingestion.

</details>

Troubleshooting Methodology

flowchart TD
    A["Service Unavailable"] --> B["Verify Network Connectivity"]
    B --> C["Check Application Status"]
    C --> D["Verify Listening Ports"]
    D --> E["Review Forwarding Configuration"]
    E --> F["Verify File Paths and Permissions"]
    F --> G["Search Indexed Events"]
    G --> H{"Issue Resolved?"}
    H -->|No| C
    H -->|Yes| I["Document Results"]

Troubleshooting Summary

Issue

Outcome

Incorrect IP address

Resolved

Splunk service unavailable

Resolved

Installation package errors

Resolved

Management port conflict

Resolved

Inactive forwarding connection

Resolved

Authentication log identification

Resolved

07. Technical Skills Developed

Domain

Practical Experience

Virtualization

Deployment of dedicated monitoring infrastructure

Linux Administration

File management, permissions, and service administration

Networking

Connectivity, port configuration, and troubleshooting

SIEM Administration

Splunk Enterprise and Universal Forwarder deployment

Log Management

Collection, indexing, and source configuration

Security Analytics

SPL searches and event verification

Documentation

Infrastructure and troubleshooting documentation

08. Monitoring Limitations

This environment uses educational security datasets rather than live production infrastructure.

Direct visibility into operational technology devices was not established. Additional log sources would be required to detect activity occurring entirely within the OT network.

The commands included in this documentation represent the implementation and troubleshooting workflow rather than a complete historical terminal recording.

09. Supporting Documentation

The original academic report contains the screenshots, configuration evidence, infrastructure documentation, and search results collected throughout this project.

View A1 Technical Report

10. AI Transparency

I used ChatGPT to help me move through the lab and better understand what was being asked of me. AI helped me troubleshoot issues I came across, understand commands, figure out what screenshots I needed, and organize my documentation.

I performed the configurations, ran the searches, and collected the results within my own Splunk environment.
