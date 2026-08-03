# Lab 01 – Provisioning an Ubuntu VPS on Contabo

---

# Objective

The objective of this lab was to provision a cloud-hosted Ubuntu Server, establish a secure SSH connection from macOS, validate the operating system, verify network connectivity, and document the initial state of the server before performing any configuration or security hardening.

---

# Environment

| Component | Details |
|-----------|---------|
| Cloud Provider | Contabo |
| VPS Plan | Cloud VPS 20 |
| Operating System | Ubuntu Server 24.04.4 LTS |
| Region | European Union (Germany) |
| Storage | 200 GB SSD |
| CPU | 6 vCPU |
| Memory | 12 GB RAM |
| Client Machine | macOS |
| Connection Method | SSH |

---

# Lab Steps

## 1. Selected the VPS Plan

After comparing the available Contabo VPS offerings, I selected the Cloud VPS 20 plan with monthly billing.

Reasons for the selection included:

- Suitable resources for Linux administration labs
- Ubuntu 24.04 LTS availability
- Germany-based data centre for lower latency
- Sufficient compute resources for future Docker, networking, and deployment labs

---

## 2. Provisioned Ubuntu Server

After completing the order, Contabo provisioned the VPS and assigned a public IPv4 address.

The server was deployed with Ubuntu Server 24.04.4 LTS.

---

## 3. Connected via SSH

From macOS, I connected to the server using:

```bash
ssh root@<server-ip>
```

During the first connection, SSH displayed the server fingerprint for verification before adding it to the local `known_hosts` file.

After authenticating with the root password, I successfully logged into the server.

---

## 4. Verified the Operating System

The following commands were executed:

```bash
hostnamectl
lsb_release -a
uname -a
```

These confirmed:

- Ubuntu 24.04.4 LTS
- Linux kernel 6.8
- x86_64 architecture
- KVM virtualization

---

## 5. Initial System Validation

The following commands were executed:

```bash
whoami
pwd
df -h
free -h
lscpu
```

These checks verified:

- Current logged-in user
- Working directory
- Available disk space
- Memory allocation
- CPU configuration
- Virtual hardware information

---

## 6. Network Connectivity

Connectivity testing included:

```bash
ping google.com
```

Results:

- DNS resolution successful
- Internet connectivity confirmed
- 0% packet loss
- Average latency approximately 13 ms

---

## 7. Lessons Learned

This lab reinforced several important Linux administration concepts.

Provisioning a server is only the first step. Before installing software or making configuration changes, it is important to validate the operating system, hardware resources, storage, and network connectivity.

I also gained a better understanding of SSH authentication, Linux system information commands, and how cloud-hosted virtual machines differ from local virtual machines.

---

# Commands Used

```bash
ssh root@<server-ip>

hostnamectl

lsb_release -a

uname -a

whoami

pwd

df -h

free -h

lscpu

ping google.com
```

---

# Screenshots

- Contabo VPS dashboard
- Successful SSH login
- `hostnamectl` output
- `df -h`
- `free -h`
- `lscpu`
- `ping google.com`

---

# Troubleshooting

### Mistyped Command

```bash
whoiam
```

Correction:

```bash
whoami
```

---

### Incorrect Network Command

```bash
ipaddr
```

Correction:

```bash
ip addr
```

---

### Incorrect Public IP Command

```bash
curl ipconfig.me
```

Correction:

```bash
curl ifconfig.me
```

---

# Next Steps

The next phase of this lab will focus on securing the server by:

- Creating a non-root administrative user
- Configuring SSH key authentication
- Disabling root login
- Enabling the UFW firewall
- Applying system updates
- Verifying secure remote access

These tasks will prepare the server for future Linux administration, Docker, networking, and deployment labs.