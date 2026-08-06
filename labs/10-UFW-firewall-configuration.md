# Lab 10 – UFW Firewall Configuration

---

# Overview

In this lab, I explored how Ubuntu uses UFW (Uncomplicated Firewall) to control incoming and outgoing network traffic.

Rather than simply enabling the firewall, I wanted to understand how Linux administrators configure firewall rules, allow only the services that should be accessible, remove unnecessary rules, explicitly block unwanted traffic, and safely manage firewall configurations without losing remote access.

One thing that stood out during this lab was the importance of allowing SSH before enabling the firewall. Although UFW is designed to simplify firewall management, enabling it without first permitting SSH access could lock an administrator out of a remote server. Working through the configuration on my own VPS made it much easier to understand how firewall rules are applied and managed in a real administration environment.

---

# Objectives

The goals of this lab were to:

- Understand the purpose of UFW
- Check firewall status
- View firewall configuration
- List available application profiles
- Allow SSH access safely
- Enable and disable the firewall
- Allow network traffic on specific ports
- Remove firewall rules
- Deny unwanted traffic
- Understand firewall rule ordering
- Learn basic Linux firewall administration

---

# Environment

- Operating System: Ubuntu Server 24.04 LTS
- Host: Contabo VPS
- Access Method: SSH from macOS Terminal
- Administrator Account: `sysadmin`

---

# Part 1 – Checking Firewall Status

I began by checking whether the firewall was already running.

```bash
sudo ufw status
```

The output showed:

```text
Status: inactive
```

To gather more information, I also checked the detailed firewall configuration.

```bash
sudo ufw status verbose
```

Since UFW had not yet been enabled, the detailed output also confirmed that the firewall was inactive.

Starting with these checks helped me understand the current security state of the server before making any configuration changes.

---

# Part 2 – Listing Available Application Profiles

Next, I explored the predefined application profiles available on the server.

```bash
sudo ufw app list
```

The output showed:

```text
OpenSSH
```

Application profiles make it easier to configure firewall rules for common services without needing to remember individual port numbers.

Since I was connected remotely over SSH, ensuring that OpenSSH was allowed before enabling the firewall was an important step.

---

# Part 3 – Allowing SSH Access

Before enabling UFW, I explicitly allowed incoming SSH connections.

```bash
sudo ufw allow OpenSSH
```

Ubuntu confirmed that the firewall rules had been updated for both IPv4 and IPv6.

Although the firewall was still inactive at this point, creating this rule first ensured that my SSH session would remain accessible after the firewall was enabled.

This reinforced one of the most important practices in Linux administration: always allow remote administrative access before enabling a firewall on a remote server.

---

# Part 4 – Enabling the Firewall

After confirming that SSH access had been allowed, I enabled UFW.

```bash
sudo ufw enable
```

Ubuntu displayed a warning explaining that enabling the firewall could interrupt existing SSH connections.

Since the SSH rule had already been configured, I confirmed the operation.

I then verified the firewall status.

```bash
sudo ufw status
```

The output now showed:

```text
Status: active
```

Viewing the detailed configuration also confirmed the default firewall policy.

```bash
sudo ufw status verbose
```

I observed that:

- Incoming connections were denied by default.
- Outgoing connections were allowed by default.
- Logging was enabled.

This helped me understand the default security posture used by Ubuntu when UFW is active.

---

# Part 5 – Allowing HTTP Traffic

To simulate preparing a server for hosting a website, I allowed HTTP traffic.

```bash
sudo ufw allow 80/tcp
```

Ubuntu successfully added rules for both IPv4 and IPv6.

I then viewed the numbered firewall rules.

```bash
sudo ufw status numbered
```

The output showed:

- OpenSSH
- HTTP (80/tcp)
- IPv6 equivalents for both rules

This demonstrated how additional services can be exposed while keeping all other ports protected.

---

# Part 6 – Removing Firewall Rules

To understand how administrators remove services that are no longer needed, I deleted the HTTP rule.

First, I viewed the numbered rules.

```bash
sudo ufw status numbered
```

Then I removed the HTTP rule.

```bash
sudo ufw delete 2
```

After checking the firewall again, I noticed that the rule numbers had changed automatically.

The remaining IPv6 HTTP rule had shifted from rule 4 to rule 3.

I then removed the remaining IPv6 rule.

```bash
sudo ufw delete 3
```

This exercise demonstrated why administrators should always verify the updated rule numbering before deleting multiple firewall rules.

---

# Part 7 – Denying Network Traffic

Next, I explored how to explicitly deny incoming traffic.

To simulate blocking an unnecessary service, I denied Telnet traffic.

```bash
sudo ufw deny 23/tcp
```

Viewing the firewall rules again confirmed that Telnet traffic was now explicitly denied for both IPv4 and IPv6.

Unlike simply leaving a port closed, an explicit deny rule documents that access to the service should always remain blocked.

---

# Part 8 – Disabling and Re-enabling the Firewall

Finally, I explored how administrators temporarily disable the firewall during maintenance or troubleshooting.

I disabled UFW.

```bash
sudo ufw disable
```

Checking the status confirmed that the firewall was inactive.

```bash
sudo ufw status
```

I then enabled it again.

```bash
sudo ufw enable
```

After checking the status one final time, I confirmed that all previously configured rules had been preserved.

This demonstrated that disabling UFW does not remove firewall rules—it simply stops enforcing them until the firewall is enabled again.

---

# Real-World Scenario

Imagine deploying a new Ubuntu web server for a company.

The server needs to:

- Allow administrators to connect remotely using SSH.
- Allow customers to access a website over HTTP or HTTPS.
- Block all unnecessary incoming traffic.

Using UFW, I configured the firewall to allow only the required services while leaving all other incoming connections blocked by default. I also practised removing unnecessary firewall rules and explicitly denying unwanted traffic, demonstrating how a Linux administrator can reduce a server's attack surface while maintaining secure remote access.

---

# What I Learned

This lab helped me understand that a firewall is much more than simply allowing or blocking ports.

I learned how UFW applies firewall rules, why SSH access should always be configured before enabling the firewall, how firewall rules are ordered, and why rule numbers change after deleting existing entries.

One of the most valuable lessons was understanding the principle of least privilege. Instead of exposing every service, only the ports required by the server should remain accessible.

Working through these exercises on my own VPS gave me practical experience configuring and managing a Linux firewall rather than simply reading about the commands.

---

# Interview Questions

## 1. What is UFW?

UFW (Uncomplicated Firewall) is a user-friendly frontend for managing Linux firewall rules.

---

## 2. Why should SSH be allowed before enabling UFW?

Allowing SSH before enabling the firewall prevents administrators from accidentally locking themselves out of a remote server.

---

## 3. Why are unnecessary ports kept closed?

Closing unused ports reduces the server's attack surface and improves security.

---

## 4. Why do UFW rule numbers change after deleting a rule?

UFW automatically renumbers the remaining rules after a deletion, so administrators should always verify the updated numbering before removing additional rules.

---

## 5. What is the difference between allowing and denying traffic?

Allow rules permit traffic to reach a service, while deny rules explicitly block traffic from reaching that service.

---

# Key Commands Used

```bash
sudo ufw status
sudo ufw status verbose
sudo ufw app list
sudo ufw allow OpenSSH
sudo ufw enable
sudo ufw allow 80/tcp
sudo ufw status numbered
sudo ufw delete
sudo ufw deny 23/tcp
sudo ufw disable
```

---

# Screenshots

The screenshots below capture the key tasks completed throughout this lab.

## 1. Checking Firewall Status

Checking whether UFW was enabled and reviewing the current firewall configuration.

![Checking UFW status](../screenshots/10-ufw-firewall/01-firewall-status.png)

---

## 2. Listing Application Profiles

Viewing the available UFW application profiles before configuring firewall rules.

![Listing UFW application profiles](../screenshots/10-ufw-firewall/02-app-list.png)

---

## 3. Enabling UFW and Allowing SSH

Allowing OpenSSH before enabling the firewall and verifying that remote access would remain available.

![Enabling UFW](../screenshots/10-ufw-firewall/03-enable-firewall.png)

---

## 4. Allowing HTTP Traffic

Adding an HTTP rule to allow web traffic and verifying the updated firewall configuration.

![Allowing HTTP traffic](../screenshots/10-ufw-firewall/04-allow-http.png)

---

## 5. Removing Firewall Rules

Removing HTTP rules and observing how UFW automatically renumbered the remaining firewall rules.

![Removing firewall rules](../screenshots/10-ufw-firewall/05-delete-rules.png)

---

## 6. Denying Traffic and Managing UFW

Creating a deny rule for Telnet, disabling the firewall, and re-enabling it while confirming that all rules remained intact.

![Managing UFW rules](../screenshots/10-ufw-firewall/06-deny-disable-enable.png)

---

# Final Thoughts

This lab gave me a much better understanding of how Linux servers are protected using host-based firewalls.

Before completing the exercises, I knew that UFW could allow and block ports, but I hadn't fully appreciated how administrators safely configure remote access, manage firewall rules over time, and minimise a server's attack surface. Working through the configuration on my own VPS made these concepts much more meaningful than simply reading command examples.

I also found it valuable to see how this lab connects with previous topics such as SSH, networking and system administration. Together, these labs are helping me build a broader understanding of how Linux servers are configured, secured and maintained in real environments.

---

# Next Steps

In the next lab, I'll continue strengthening my Linux system administration skills by exploring additional server administration tasks and security concepts while expanding my hands-on experience with real Linux infrastructure.