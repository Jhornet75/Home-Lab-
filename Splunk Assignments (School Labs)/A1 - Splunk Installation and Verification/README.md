Splunk Enterprise Deployment and Log Ingestion

Project: A1 — Security Monitoring Infrastructure
Course: CYDA 420 — Applied Network Security Monitoring
Environment: Self-hosted Proxmox Homelab

Project Overview

For this project, I deployed Splunk Enterprise within my Proxmox homelab to establish a centralized security monitoring environment. My goal was to gain hands-on experience with SIEM infrastructure, understand how security logs are collected and indexed, and establish a working environment for future threat detection and incident investigation labs.

Rather than using a preconfigured cloud environment, I installed and configured Splunk on a dedicated Ubuntu Server virtual machine.

Technologies Used

Technology

Purpose

Proxmox VE

Virtualization platform

Ubuntu Server 24.04

SIEM hosting environment

Splunk Enterprise

Centralized security monitoring

Splunk Universal Forwarder

Log collection and forwarding

Linux CLI

Installation and troubleshooting

SPL

Log searching and analysis

TCP 8000

Splunk web interface

TCP 9997

Splunk log receiver

Architecture

The monitoring environment consists of a dedicated Ubuntu Server VM hosting Splunk Enterprise and a Universal Forwarder. Authentication logs are forwarded to Splunk, while firewall logs were manually uploaded through the web interface.

Implementation

1. Splunk Deployment

Provisioned an Ubuntu Server virtual machine in Proxmox.

Installed Splunk Enterprise 10.6.1.1.

Configured administrative access.

Started and verified the Splunk service.

Established access to the web interface through TCP port 8000.

2. Log Forwarding

Installed Splunk Universal Forwarder.

Configured TCP port 9997 for receiving logs.

Configured a monitored authentication log file.

Verified the active forwarding connection.

3. Index and Source Configuration

Created separate indexes to organize authentication and firewall events.

Log Source

Index

Sourcetype

Domain Controller

cascade_auth

cascade:dc:security

Boundary Firewall

cascade_firewall

cascade:fw:boundary

4. Log Ingestion and Verification

Verified that both log sources were successfully indexed and searchable through Splunk using SPL queries.

Separating these sources into dedicated indexes provides a foundation for future authentication monitoring, network traffic analysis, and security event correlation.

Challenges and Troubleshooting

Throughout this project, I encountered several issues relating to service availability, network connectivity, log forwarding, and data ingestion.

To troubleshoot these issues, I verified the VM's network configuration, checked the status of Splunk services, reviewed listening ports, and confirmed that the Universal Forwarder was communicating with Splunk Enterprise.

Working through these issues helped me better understand the relationship between the operating system, network infrastructure, and SIEM platform.

Results

Operational self-hosted Splunk Enterprise deployment.

Active Universal Forwarder connection.

Dedicated authentication and firewall indexes.

Successfully ingested security logs.

Searchable security events through SPL.

Skills Developed

SIEM administration

Linux server administration

Virtualization

Log collection and forwarding

Network troubleshooting

SPL fundamentals

Security monitoring infrastructure

Technical documentation

Limitations

The environment uses educational security datasets rather than live production infrastructure. Direct visibility into OT devices was not established, leaving monitoring gaps that would require additional log sources.

AI Assistance

I used AI to help me move through the lab, better understand the configurations, and troubleshoot issues I came across. AI also helped me organize and document the project. I performed the configurations, ran the commands, and verified the results within my own environment.
