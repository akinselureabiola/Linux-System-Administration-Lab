# Lab 06 – Package Management (APT)

---

# Overview

In this lab, I explored how software is managed on an Ubuntu server using the Advanced Package Tool (APT).

Rather than simply installing applications, I wanted to understand the complete package management process—from checking for updates and searching for packages to installing, verifying, removing, and cleaning up software.

One thing I found interesting was that even though my server was already fully up to date, there were still plenty of opportunities to explore how APT works. Working through each command on a live Ubuntu VPS gave me a much better understanding of how Linux administrators keep systems updated and maintained.

---

# Objectives

The goals of this lab were to:

- Refresh the package repository information
- Check for available software updates
- Search for packages
- View detailed package information
- Install software using APT
- Verify installed packages
- Remove packages safely
- Clean unused packages and package cache
- Understand the package management workflow used on Ubuntu

---

# Environment

- Operating System: Ubuntu Server 24.04 LTS
- Host: Contabo VPS
- Access Method: SSH from macOS Terminal
- Administrator Account: `sysadmin`

---

# Part 1 – Updating Package Information

Before installing or updating any software, I refreshed the package index.

```bash
sudo apt update
```

The command completed successfully and reported that all packages on the server were already up to date.

To double-check, I listed any available upgrades.

```bash
sudo apt list --upgradable
```

No packages were returned, confirming that the system was fully updated.

---

# Part 2 – Searching for Packages

Next, I explored how to search for software available in Ubuntu's repositories.

For example:

```bash
apt search tree
```

This returned several matching packages.

To learn more about the `tree` package, I displayed its detailed information.

```bash
apt show tree
```

This provided useful details including:

- Version
- Description
- Dependencies
- Download size
- Installed size
- Repository source

Seeing this information helped me understand how Linux packages are documented before installation.

---

# Part 3 – Installing a Package

To practise package installation, I installed the `tree` utility.

```bash
sudo apt install tree
```

After installation completed, I confirmed that the package was installed correctly.

```bash
tree --version
```

I also verified it using:

```bash
dpkg -l tree
```

Both commands confirmed that the installation had completed successfully.

---

# Part 4 – Using the Installed Package

Rather than installing software without testing it, I wanted to confirm it actually worked.

I used the `tree` command to display the directory structure of my home folder.

```bash
tree ~
```

I also limited the output depth.

```bash
tree -L 2 ~
```

This produced a much cleaner view of my directory structure and showed why the utility is useful when navigating larger projects.

---

# Part 5 – Removing a Package

After confirming how the package worked, I removed it.

```bash
sudo apt remove tree
```

Linux displayed the packages that would be removed before asking for confirmation.

Once the removal completed, I verified that the package was no longer available.

```bash
tree --version
```

This confirmed that the software had been successfully removed from the system.

---

# Part 6 – Cleaning Up

After removing the package, I cleaned up unnecessary files left behind.

First, I removed packages that were no longer required.

```bash
sudo apt autoremove
```

Then I cleaned the package cache.

```bash
sudo apt autoclean
```

At first, seeing many lines beginning with `Del` was a little confusing because it looked as though important software such as OpenSSH and Python was being deleted.

After looking into it, I realised these entries referred only to outdated package files stored in APT's local cache. The installed software itself remained untouched. It was a good reminder that understanding command output is just as important as knowing which command to run.

---

# What I Learned

This lab helped me understand that package management involves much more than simply installing software.

I learned how Linux keeps track of available packages, where software comes from, how to inspect packages before installing them, and how to keep the system clean by removing unnecessary files afterwards.

One thing that stood out was the importance of reading command output carefully. Seeing the `Del` entries during `apt autoclean` initially looked alarming, but investigating the output helped me understand that only cached package files were being removed.

Working through each step on my own VPS made the package management process feel much more practical than simply reading the documentation.

---

# Key Commands Used

```bash
sudo apt update
sudo apt list --upgradable
apt search
apt show
sudo apt install
dpkg -l
tree
sudo apt remove
sudo apt autoremove
sudo apt autoclean
```

---

# Screenshots

The screenshots below capture the key stages of this lab.

## 1. Updating Package Information

Refreshing the package index and confirming that the server was already fully up to date.

```text
sudo apt update
sudo apt list --upgradable
```

**Screenshot**

```
../screenshots/06-package-management/01-system-update.png
```

---

## 2. Searching for Package Information

Searching the repositories and viewing detailed information about the `tree` package before installation.

```text
apt search tree
apt show tree
```

**Screenshot**

```
../screenshots/06-package-management/02-package-search.png
```

---

## 3. Installing and Verifying the Package

Installing the package and confirming that it was successfully installed.

```text
sudo apt install tree
tree --version
dpkg -l tree
```

**Screenshot**

```
../screenshots/06-package-management/03-package-installation.png
```

---

## 4. Testing the Installed Package

Using the `tree` command to display the directory structure of the home directory.

```text
tree ~
tree -L 2 ~
```

**Screenshot**

```
../screenshots/06-package-management/04-using-tree-command.png
```

---

## 5. Removing the Package and Cleaning Up

Removing the package, deleting unused dependencies, and cleaning the local package cache.

```text
sudo apt remove tree
sudo apt autoremove
sudo apt autoclean
```

**Screenshot**

```
../screenshots/06-package-management/05-package-removal.png
```

---

# Final Thoughts

Although package management seems straightforward at first, I found there is much more happening behind the scenes than I initially realised.

This lab helped me understand how Ubuntu manages software throughout its lifecycle—from discovering packages and installing them to removing software cleanly and maintaining the package cache.

More importantly, it reinforced the habit of paying attention to command output instead of assuming what a command has done. That small detail turned out to be one of the most valuable lessons from this exercise.

---

# Next Steps

In the next lab, I'll move on to managing services with **systemd**, where I'll learn how Linux starts, stops, enables, and monitors the background services that keep a server running.