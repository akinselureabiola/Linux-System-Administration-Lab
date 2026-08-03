# Enterprise Linux Administration Lab Architecture

## Overview

This project documents my journey of learning Linux system administration by managing a real Ubuntu Server hosted on a Contabo VPS.

I've previously worked with Linux in virtual machines and completed several hands-on Linux labs, but I wanted to take the next step by working with a real server that is accessible over the internet. My goal is to build practical experience that reflects the kind of work carried out by Linux System Administrators and IT Infrastructure Engineers.

As I progress, I'll document each configuration, challenge, and troubleshooting step so this repository serves as both a learning journal and a professional portfolio.

---

## Project Objectives

Through this project, I plan to gain practical experience in the following areas:

- Linux System Administration
- Ubuntu Server Administration
- SSH Configuration and Remote Access
- Linux Networking
- DNS Configuration
- Firewall Management with UFW
- Web Server Administration using Nginx
- Bash Scripting and Automation
- System Monitoring and Log Analysis
- Security Hardening
- Troubleshooting
- Infrastructure Documentation

---

## Planned Infrastructure


                 Internet
                     │
                     ▼
             Contabo VPS
        Ubuntu Server 24.04 LTS
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
      SSH                      Nginx
        │                         │
        ▼                         ▼
Remote Administration      Hosted Website
        │
        ▼
Firewall (UFW)
        │
        ▼
System Monitoring
        │
        ▼
Automation

---

## Services to be Configured

| Service | Purpose |
|----------|---------|
| SSH | Secure remote administration |
| UFW | Firewall configuration and management |
| Nginx | Web server hosting |
| Fail2Ban | Protect the server against brute-force attacks |
| Cron | Schedule recurring administrative tasks |
| Systemd | Manage Linux services |
| Certbot | Configure SSL certificates |
| Logrotate | Manage and rotate system logs |

---

## Skills I Want to Develop

By the end of this project, I want to be confident carrying out common Linux administration tasks such as:

- Managing Linux servers
- Configuring networks and DNS
- Securing a Linux server
- Deploying and managing web services
- Troubleshooting system and network issues
- Monitoring server performance
- Automating repetitive administrative tasks
- Writing clear technical documentation

---

## Long-Term Goal

The aim of this project is to build practical Linux administration experience that goes beyond working in virtual machines.

By documenting each stage of the journey, I hope to create a portfolio that demonstrates my ability to administer Linux servers in a way that reflects real-world IT Support, System Administration, and Cloud Infrastructure environments.