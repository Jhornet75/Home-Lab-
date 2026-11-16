<img width="169" height="150" alt="download" src="https://github.com/user-attachments/assets/9d241416-52cc-409d-9689-ef280bb47965" />A1 | Splunk Enterprise Deployment and Log Ingestion

Cyber Defense and Analysis — CYDA 420
Infrastructure: Self-hosted Proxmox Homelab
Status: Operational

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

![Uploading download.svg…]<svg font-family="-apple-system-body, ui-sans-serif, -apple-system, system-ui, &quot;Segoe UI&quot;, Helvetica, &quot;Apple Color Emoji&quot;, Arial, sans-serif, &quot;Segoe UI Emoji&quot;, &quot;Segoe UI Symbol&quot;" font-weight="400" data-d-component="svg" fill="currentColor" style="color:rgb(237, 237, 237)" viewBox="0 0 340 302" width="100%" xmlns="http://www.w3.org/2000/svg"><rect x="8" y="5" width="150" height="38" rx="4" fill="#374151" stroke="#9ca3af"/><rect x="182" y="5" width="150" height="38" rx="4" fill="#374151" stroke="#9ca3af"/><text x="83" y="28" font-size="10" text-anchor="middle" fill="#fff">Domain Controller Logs</text><text x="257" y="28" font-size="10" text-anchor="middle" fill="#fff">Boundary Firewall Logs</text><rect x="8" y="65" width="324" height="230" rx="6" fill="#64748b" fill-opacity=".06" stroke="#94a3b8" stroke-dasharray="4 3"/><text x="20" y="82" font-size="10" fill="currentColor">Proxmox / Ubuntu Server VM</text><path d="M83 43 V99 M257 43 V99" stroke="#94a3b8" stroke-width="1.5"/><rect x="18" y="99" width="137" height="42" rx="4" fill="#263445" stroke="#64748b"/><text x="86" y="117" font-size="11" fill="#fff" text-anchor="middle">Universal Forwarder</text><text x="86" y="132" font-size="9" fill="#cbd5e1" text-anchor="middle">Monitored file</text><rect x="185" y="99" width="137" height="42" rx="4" fill="#263445" stroke="#64748b"/><text x="253" y="117" font-size="11" fill="#fff" text-anchor="middle">Splunk Enterprise</text><text x="253" y="132" font-size="9" fill="#cbd5e1" text-anchor="middle">Manual upload</text><path d="M155 120 H185 L179 116 M185 120 L179 124" fill="none" stroke="#0891b2" stroke-width="1.7"/><text x="170" y="110" text-anchor="middle" font-size="8" fill="#0891b2">9997</text><path d="M253 141 V161 H91 V180 M253 161 V180" fill="none" stroke="#94a3b8"/><rect x="20" y="180" width="135" height="48" rx="4" fill="#164e63" stroke="#0891b2"/><text x="87.5" y="199" text-anchor="middle" font-size="11" fill="#fff">cascade_auth</text><text x="87.5" y="215" text-anchor="middle" font-size="10" fill="#bae6fd">5,103 events</text><rect x="185" y="180" width="135" height="48" rx="4" fill="#164e63" stroke="#0891b2"/><text x="252.5" y="199" text-anchor="middle" font-size="11" fill="#fff">cascade_firewall</text><text x="252.5" y="215" text-anchor="middle" font-size="10" fill="#bae6fd">8,984 events</text><path d="M253 228 V240 H170 V251 M87 228 V240 H170" stroke="#94a3b8" fill="none"/><rect x="100" y="251" width="140" height="32" rx="4" fill="#263445" stroke="#64748b"/><text x="170" y="271" font-size="11" text-anchor="middle" fill="#fff">Splunk Web :8000</text></svg>()

Indexed security events


Authentication

Firewall
















Collapsible troubleshooting preview
Issue 05 — Inactive Forwarding ConnectionResolvedProblem: Forwarder configured but inactive.

Investigation:

sudo ss -lntp | grep 9997

Resolution: Restored Splunk Enterprise and verified the forwarding connection.
