# Lab 02 – SSH Hardening and Secure Remote Administration

---

# Objective

Once I had provisioned my Ubuntu VPS, the next thing I wanted to focus on was security.

When I first connected to the server, I was logging in directly as the root user using a password. While that's convenient when setting up a new server, it's not how you would normally administer a production Linux system.

The goal of this lab was to start moving towards a more secure configuration by replacing password-based authentication with SSH keys and reducing unnecessary exposure.

---

# Why This Matters

One thing I quickly realised while working with a real VPS is that it's very different from running Linux inside VirtualBox.

This server has a public IP address, which means anyone on the internet can attempt to connect to it. Because of that, securing SSH isn't just considered good practice—it becomes one of the first things that should be done.

---

# Environment

| Component | Details |
|------------|----------|
| Cloud Provider | Contabo |
| Operating System | Ubuntu Server 24.04.4 LTS |
| Client | macOS Terminal |
| Authentication | OpenSSH |

---

# What I Did

## 1. Verified Existing SSH Keys on macOS

Before creating anything new, I checked whether my Mac already had an SSH key pair.

```bash
ls -la ~/.ssh
```

I found an existing Ed25519 key pair that I could reuse.

To view my public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

---

## 2. Created the SSH Directory

On the server, the `sysadmin` account didn't yet have an `.ssh` directory.

I created one and then secured it.

```bash
mkdir ~/.ssh

chmod 700 ~/.ssh
```

The `700` permission means only the owner can access the directory.

---

## 3. Added My Public Key

I created an `authorized_keys` file and pasted in the public key from my Mac.

```bash
nano ~/.ssh/authorized_keys
```

After saving the file, I secured it.

```bash
chmod 600 ~/.ssh/authorized_keys
```

This prevents other users on the system from reading or modifying the file.

---

## 4. Tested SSH Key Authentication

Before making any security changes, I opened a second terminal and tested logging in.

```bash
ssh sysadmin@<server-ip>
```

The login succeeded immediately without asking for my password, confirming that key-based authentication was working correctly.

I intentionally tested this before disabling any authentication methods. If something had gone wrong, I would still have had another active SSH session available.

---

## 5. Backed Up the SSH Configuration

Before editing the SSH daemon configuration, I created a backup.

```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

I've learned that creating backups before making configuration changes is a habit worth developing early.

---

## 6. Disabled Root Login

Inside the SSH configuration file I changed:

```text
PermitRootLogin no
```

This prevents anyone from logging in directly as the root user.

Instead, administration is performed through the dedicated `sysadmin` account using `sudo`.

---

## 7. Validated the Configuration

Before reloading the SSH service, I checked that the configuration contained no syntax errors.

```bash
sudo sshd -t
```

The command returned no output, which indicates the configuration is valid.

---

## 8. Reloaded SSH

Rather than restarting the service completely, I reloaded it.

```bash
sudo systemctl reload ssh
```

Reloading applies configuration changes without unnecessarily interrupting existing sessions.

---

## 9. Verified the Service

```bash
sudo systemctl status ssh
```

The SSH service remained active after the reload.

---

# Something Interesting I Observed

While checking the SSH logs, I noticed repeated login attempts from unknown IP addresses.

The server was receiving authentication attempts for usernames such as:

- root
- anderson
- gary
- ubuntu
- dst
- odoo

None of these accounts exist on my server.

This was my first time seeing real internet-wide SSH scanning happening on a server I was managing. It reinforced why securing SSH should be one of the very first tasks after deploying a VPS.

---

# Lessons Learned

This lab changed the way I think about SSH.

Previously, SSH was simply the command I used to connect to Linux machines.

Working with a public VPS made me realise that SSH is effectively the front door to the server. The way it's configured has a direct impact on the security of the entire system.

I also learned that making configuration changes safely is just as important as making them correctly. Testing first, validating configuration files, and keeping an active session open are habits I'll continue throughout the rest of this project.

---

# Commands Used

```bash
ls -la ~/.ssh

cat ~/.ssh/id_ed25519.pub

mkdir ~/.ssh

chmod 700 ~/.ssh

nano ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys

ssh sysadmin@<server-ip>

sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

sudo nano /etc/ssh/sshd_config

sudo sshd -t

sudo systemctl reload ssh

sudo systemctl status ssh
```