# Lab 11 – Fail2Ban Intrusion Prevention

---

# Overview

In this lab, I learned how to protect an Ubuntu server from brute-force login attacks using Fail2Ban.

Unlike a traditional firewall that simply allows or blocks network traffic, Fail2Ban actively monitors authentication logs and automatically blocks IP addresses that repeatedly fail to authenticate. This provides an additional layer of security for internet-facing services such as SSH.

One of the most interesting parts of this lab was working with a live VPS connected to the public internet. Instead of generating artificial log entries, I observed real SSH brute-force attempts from different IP addresses around the world. Seeing Fail2Ban automatically detect those attacks and temporarily ban the offending IP addresses made the purpose of the tool much clearer.

This lab also demonstrated how several of the previous Linux administration labs work together. SSH writes authentication events to the system journal, Fail2Ban reads those logs, and the firewall enforces the temporary bans.

---

# Objectives

The goals of this lab were to:

- Understand the purpose of Fail2Ban
- Install and configure Fail2Ban
- Understand the relationship between SSH, Journald and Fail2Ban
- Configure custom security policies
- Monitor SSH protection
- Verify Fail2Ban runtime configuration
- View banned IP addresses
- Examine real SSH authentication logs
- Understand how automated brute-force attacks are detected and blocked

---

# Environment

- Operating System: Ubuntu Server 24.04 LTS
- Host: Contabo VPS
- Access Method: SSH from macOS Terminal
- Administrator Account: `sysadmin`

---

# Part 1 – Installing Fail2Ban

I first checked whether Fail2Ban was already installed.

```bash
sudo systemctl status fail2ban
```

The service could not be found, confirming that it was not installed on the server.

To verify this further, I checked the installed packages.

```bash
apt list --installed | grep fail2ban
```

No packages were returned.

I updated the package repository before installing Fail2Ban.

```bash
sudo apt update
```

Finally, I installed the package.

```bash
sudo apt install fail2ban
```

After the installation completed successfully, systemd automatically enabled the Fail2Ban service.

---

# Part 2 – Verifying the Service

After installation, I confirmed that the service was running correctly.

```bash
sudo systemctl status fail2ban
```

The output showed:

- Service loaded
- Service enabled
- Active (running)

This confirmed that Fail2Ban was now protecting the server.

I also checked the available jails.

```bash
sudo fail2ban-client status
```

The output showed one active jail:

```text
sshd
```

This confirmed that SSH protection was already enabled by the default Ubuntu configuration.

---

# Part 3 – Exploring the Configuration

Before making any changes, I explored the Fail2Ban configuration directory.

```bash
ls -l /etc/fail2ban
```

This allowed me to identify the main configuration files, including:

- fail2ban.conf
- jail.conf
- jail.d
- filter.d
- action.d

I also inspected the default SSH jail configuration.

```bash
grep "^\[sshd\]" -A 20 /etc/fail2ban/jail.conf
```

Finally, I reviewed Ubuntu's default overrides.

```bash
cat /etc/fail2ban/jail.d/defaults-debian.conf
```

This showed that Ubuntu already enables the SSH jail and uses the systemd backend for log monitoring.

Exploring these files helped me understand how Fail2Ban organizes its configuration without modifying the default files directly.

---

# Part 4 – Creating a Custom Configuration

Rather than editing the default configuration files, I created my own local configuration.

```bash
sudo nano /etc/fail2ban/jail.local
```

I configured the following settings:

```ini
[DEFAULT]
bantime = 10m
findtime = 10m
maxretry = 5
backend = systemd

[sshd]
enabled = true
port = ssh
```

These settings instruct Fail2Ban to:

- monitor failed SSH logins
- allow five failed authentication attempts
- monitor failures within a ten-minute period
- temporarily ban offending IP addresses for ten minutes

Using a separate `jail.local` file ensures that my custom configuration is preserved when the package is updated.

---

# Part 5 – Applying and Verifying the Configuration

After creating the configuration file, I restarted the service.

```bash
sudo systemctl restart fail2ban
```

I confirmed that the service restarted successfully.

```bash
sudo systemctl status fail2ban
```

Rather than assuming the configuration had been applied, I verified the active runtime settings.

```bash
sudo fail2ban-client get sshd maxretry
```

Output:

```text
5
```

Next, I checked the monitoring window.

```bash
sudo fail2ban-client get sshd findtime
```

Output:

```text
600
```

Finally, I verified the ban duration.

```bash
sudo fail2ban-client get sshd bantime
```

Output:

```text
600
```

Although my configuration specified ten minutes, Fail2Ban reports time internally in seconds.

This confirmed that my configuration had been successfully loaded.

---

# Part 6 – Monitoring Active Protection

To see how the SSH jail was performing, I checked its current status.

```bash
sudo fail2ban-client status sshd
```

The output displayed:

- Current failed login attempts
- Total failed login attempts
- Number of currently banned IP addresses
- Total bans issued
- List of banned IP addresses

One thing that surprised me was discovering that my VPS was already receiving continuous brute-force login attempts from the public internet.

Fail2Ban automatically detected these repeated authentication failures and temporarily banned the offending IP addresses without any manual intervention.

Watching the number of failed logins and banned IPs change over time demonstrated that the system was actively protecting the server in real time.

---

# Part 7 – Examining SSH Authentication Logs

To better understand what Fail2Ban was detecting, I inspected the SSH authentication logs.

```bash
sudo journalctl -u ssh --since "30 minutes ago" | tail -30
```

The logs showed multiple failed login attempts against the server.

Examples included:

```text
Failed password for root

Failed password for invalid user wallet

Failed password for invalid user xmr

Failed password for invalid user kusama
```

I also observed connection attempts coming from multiple public IP addresses around the world.

Seeing these real authentication attempts made it much easier to understand how Fail2Ban identifies suspicious behaviour before issuing temporary bans.

---

# What I Learned

This lab helped me understand that server security involves much more than simply installing a firewall.

I learned how Fail2Ban continuously monitors authentication logs, identifies repeated failed login attempts, and automatically blocks malicious IP addresses before they can continue brute-force attacks.

One of the most valuable lessons was seeing how the different Linux components work together. SSH records authentication events, systemd and Journald manage the logs, Fail2Ban analyses those logs, and the firewall enforces the temporary bans.

Working on a live VPS made this experience much more realistic because the attacks I observed were genuine internet scanning activity rather than simulated examples.

---

# Key Commands Used

```bash
sudo apt update
sudo apt install fail2ban
sudo systemctl status fail2ban
sudo systemctl restart fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd
sudo fail2ban-client get sshd maxretry
sudo fail2ban-client get sshd findtime
sudo fail2ban-client get sshd bantime
sudo fail2ban-client get sshd banip
ls -l /etc/fail2ban
cat /etc/fail2ban/jail.d/defaults-debian.conf
grep "^\[sshd\]" -A 20 /etc/fail2ban/jail.conf
sudo journalctl -u ssh --since "30 minutes ago"
```

---

# Screenshots

The screenshots below capture the key tasks completed throughout this lab.

## 1. Installing Fail2Ban

Installing the Fail2Ban package and verifying that the installation completed successfully.

![Installing Fail2Ban](./screenshots/11-fail2ban/01-install-fail2ban.png)

---

## 2. Verifying the Service

Confirming that the Fail2Ban service is running and checking the active SSH jail.

![Fail2Ban service status](./screenshots/11-fail2ban/02-service-status.png)

---

## 3. Exploring the Configuration

Inspecting the Fail2Ban configuration directory and reviewing the default SSH jail configuration.

![Fail2Ban configuration](./screenshots/11-fail2ban/03-configuration.png)

---

## 4. Creating a Custom Jail Configuration

Creating the `jail.local` file and configuring custom security policies for SSH.

![Custom jail.local configuration](./screenshots/11-fail2ban/04-jail-local.png)

---

## 5. Verifying Runtime Configuration

Checking the active Fail2Ban configuration using the fail2ban-client utility.

![Runtime configuration verification](./screenshots/11-fail2ban/05-runtime-configuration.png)

---

## 6. Monitoring Active SSH Protection

Viewing the SSH jail status, failed login attempts, banned IP addresses and overall protection statistics.

![Fail2Ban SSH jail status](./screenshots/11-fail2ban/06-sshd-status.png)

---

## 7. Examining Authentication Logs

Reviewing real SSH authentication logs to observe brute-force login attempts detected on the public VPS.

![SSH authentication logs](./screenshots/11-fail2ban/07-authentication-logs.png)

---

# Final Thoughts

This has been one of the most rewarding Linux administration labs so far.

Unlike previous security exercises that focused on configuring individual components, this lab demonstrated how multiple technologies work together to protect a production server. Seeing real brute-force login attempts against my VPS made the concepts much more meaningful than reading about them in documentation.

I also gained a much better understanding of how Linux administration overlaps with cybersecurity. Configuring Fail2Ban, analysing authentication logs, identifying malicious login attempts, and verifying automatic IP bans are tasks commonly performed by both Linux system administrators and security professionals responsible for protecting servers.

Perhaps the biggest takeaway from this lab is that security is not achieved through a single tool. Instead, it is built by combining secure services, proper logging, firewall rules and automated intrusion prevention into a layered defence.

---

# Next Steps

With the core Linux administration roadmap now complete, the next phase will focus on expanding these skills into enterprise system administration by exploring web servers, containerisation, automation, configuration management, monitoring and cloud infrastructure. These topics will build upon the strong Linux foundation established throughout the first eleven labs.