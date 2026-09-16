# Phase 9 — Monitoring & Operations

> **Think like a Systems Engineer.**

This phase focuses on two core operational responsibilities of a Linux Systems Administrator:

1. **Monitoring** — understanding the current health and performance of a Linux server.
2. **Backup & Restore** — protecting data, automating backups, and proving that data can actually be restored.

The labs were completed on an Ubuntu 24.04.4 LTS server running on a Contabo VPS.

---

## Environment

| Item | Details |
|---|---|
| OS | Ubuntu 24.04.4 LTS |
| Hostname | `vmd200746` |
| User | `sysadmin` |
| CPU | 6 CPUs |
| RAM | ~11–12 GiB |
| Root Disk | ~193 GB |
| Swap | 0 |
| Web Server | Nginx 1.24.0 |
| Network Interface | `eth0` |
| IPv4 | `173.249.47.239` |
| IPv6 | `2a02:c207:3020:746::1` |

---

# Lab 23 — System Monitoring

## Topics Covered

- Disk usage
- Memory
- CPU
- Network
- Performance troubleshooting

---

## 1. CPU Monitoring with `top`

The first step in troubleshooting a Linux server is understanding its current resource usage.

```bash
top
```

The server was initially healthy:

- CPU was approximately **99.8% idle**
- Load average was very low
- I/O wait was approximately **0%**
- Memory availability was high
- No swap was being used

### Important CPU fields

| Field | Meaning |
|---|---|
| `us` | User-space CPU usage |
| `sy` | Kernel/system CPU usage |
| `id` | CPU idle time |
| `wa` | CPU waiting for I/O |
| `ni` | User processes with adjusted priority |
| `hi` | Hardware interrupts |
| `si` | Software interrupts |
| `st` | CPU time stolen by the hypervisor |

---

## 2. Controlled CPU Load Test

To understand how CPU pressure appears in monitoring tools, a controlled workload was created:

```bash
yes > /dev/null
```

The process consumed approximately **100% CPU**.

Because the server has 6 CPUs, a single process using 100% CPU represents roughly one CPU core being fully utilized.

The process was stopped with:

```text
Ctrl+C
```

After stopping it, the CPU returned to normal.

### What this demonstrated

A high CPU percentage does not automatically mean the whole server is overloaded. It is important to consider:

- Number of CPU cores
- Which process is consuming CPU
- Load average
- Duration of the load
- Whether users are actually experiencing performance problems

---

## 3. Understanding Load Average

The load average was observed through `top`.

Linux normally displays:

```text
load average: 1 minute, 5 minutes, 15 minutes
```

Load average represents the number of tasks that are runnable or waiting for certain resources.

For a server with 6 CPUs:

- Load around `1` is roughly one CPU's worth of runnable work.
- Load around `6` means the CPUs are heavily occupied.
- Sustained load significantly above `6` indicates that runnable work is accumulating.

The controlled CPU test also demonstrated that load average does not immediately return to zero because it is calculated over time.

---

## 4. Process Investigation with `ps`

To inspect running processes:

```bash
ps aux
```

To identify the processes consuming the most CPU:

```bash
ps aux --sort=-%cpu | head
```

To identify the processes consuming the most memory:

```bash
ps aux --sort=-%mem | head
```

Processes observed during the lab included:

- Nginx master and worker processes
- Docker
- containerd
- fail2ban
- sshd
- system services
- The controlled Python memory-test process

### Important observation

A process appearing at `100%` CPU in a short `ps` snapshot does not automatically mean it has been consuming 100% CPU for a long time.

For meaningful troubleshooting, consider:

- `%CPU`
- CPU time
- Process lifetime
- Whether the process is short-lived
- The overall system load

---

# 5. Memory Monitoring

The main command used was:

```bash
free -h
```

The server had approximately 11 GiB of RAM and very high available memory.

### Important memory concepts

| Term | Meaning |
|---|---|
| `total` | Total physical memory available |
| `used` | Memory currently in use |
| `free` | Completely unused memory |
| `buff/cache` | Memory used for buffers and filesystem cache |
| `available` | Approximate memory available to applications without significant pressure |
| `swap` | Disk-backed memory used when required |

### Key lesson

Linux using memory for cache is normal. A low `free` value by itself does not necessarily mean the server is running out of RAM.

The more useful indicator is often:

```text
available
```

---

## 6. Controlled Memory Test

A controlled test allocated approximately 500 MB of memory:

```bash
python3 -c 'x = bytearray(500 * 1024 * 1024); input("Memory allocated. Press Enter to release it...")'
```

While the process was waiting, memory usage increased.

The process was identified with:

```bash
ps aux --sort=-%mem | head
```

The Python process appeared near the top of the memory-consuming processes.

After pressing Enter, the memory was released and the system returned close to its previous state.

### Lesson

Monitoring becomes much easier when you can correlate:

```text
system-level symptom → process → controlled test → recovery
```

---

# 7. `vmstat` — System Performance Overview

The following command was used:

```bash
vmstat 1 5
```

This provides repeated snapshots of system performance.

Important fields include:

| Field | Meaning |
|---|---|
| `r` | Runnable processes |
| `b` | Processes blocked |
| `swpd` | Used swap |
| `si` | Swap in |
| `so` | Swap out |
| `free` | Free memory |
| `buff` | Buffer memory |
| `cache` | Cache memory |
| `bi` | Blocks read |
| `bo` | Blocks written |
| `in` | Interrupts |
| `cs` | Context switches |
| `us` | User CPU |
| `sy` | System CPU |
| `id` | Idle CPU |
| `wa` | I/O wait |
| `st` | Steal time |

The observed system showed:

- No swap activity
- Very low runnable/blocked activity
- Very low I/O activity
- High CPU idle time
- Approximately 0% I/O wait

---

# 8. Disk Capacity with `df`

To check filesystem capacity:

```bash
df -h
```

The root filesystem was approximately:

```text
193G total
6.5G used
187G available
4% used
```

### Important distinction

`df` answers:

> How much filesystem capacity is being used?

It does **not** directly tell us how actively the disk is reading or writing.

---

# 9. Finding Large Directories with `du`

To identify where disk space is being used:

```bash
sudo du -sh /* 2>/dev/null
```

The largest areas included:

- `/var`
- `/usr`
- `/home`
- `/boot`

A more detailed check of `/var` was performed:

```bash
sudo du -sh /var/* 2>/dev/null
```

The main disk consumers included:

```text
/var/lib
/var/log
/var/cache
```

The log directory was then investigated:

```bash
sudo du -sh /var/log/* 2>/dev/null
```

The system journal was the largest area observed.

---

# 10. Investigating System Logs

The journal disk usage was checked with:

```bash
sudo journalctl --disk-usage
```

The journal was using approximately 1.5 GB.

The configuration was inspected with:

```bash
grep -R "SystemMaxUse\|SystemKeepFree\|RuntimeMaxUse\|MaxRetentionSec" \
/etc/systemd/journald.conf \
/etc/systemd/journald.conf.d/ 2>/dev/null
```

Only commented default configuration entries were found in the checked configuration.

A dry-run option was attempted with:

```bash
sudo journalctl --vacuum-size=500M --dry-run
```

This demonstrated that the installed `journalctl` version did not support the attempted `--dry-run` option.

### Operational lesson

Do not blindly delete files from `/var/log` or `/var/lib`.

First:

1. Identify what is consuming space.
2. Understand what the files are used for.
3. Check the service or application responsible.
4. Use the application's supported cleanup/retention mechanism.
5. Verify the result.

---

# 11. Network Interfaces

Network interfaces were inspected with:

```bash
ip addr
```

The main network interface was:

```text
eth0
```

Other interfaces included:

- `lo` — loopback
- `docker0` — Docker bridge
- `br-206165c0cae0` — container networking bridge

The main interface was operational and had IPv4 and IPv6 connectivity.

---

# 12. Routing

The routing table was inspected using:

```bash
ip route
```

The default route was:

```text
default via 173.249.47.1 dev eth0
```

This shows that traffic destined outside the local networks is sent through the configured gateway.

---

# 13. Connectivity Testing

External network connectivity was tested with:

```bash
ping -c 4 1.1.1.1
```

The test returned:

- 4 packets transmitted
- 4 packets received
- 0% packet loss
- Approximately 4.5 ms average latency

This confirmed basic IP connectivity.

---

# 14. DNS Testing

DNS resolution was tested with:

```bash
getent hosts google.com
```

The hostname successfully resolved.

Another test was:

```bash
ping cloudflare.com
```

The hostname resolved and the server received replies.

### Troubleshooting principle

When a hostname does not work, separate:

```text
DNS problem
```

from:

```text
network connectivity problem
```

For example:

```bash
ping 1.1.1.1
```

tests IP connectivity, while:

```bash
getent hosts google.com
```

tests hostname resolution.

---

# 15. Listening Network Services

Listening ports were checked with:

```bash
sudo ss -tulpn
```

Important services observed included:

- SSH on port 22
- Nginx on ports 80 and 443
- Local DNS resolver on port 53
- Container-related networking

### Address meanings

```text
0.0.0.0:PORT
```

Means the service is listening on all IPv4 interfaces.

```text
[::]:PORT
```

Means the service is listening on IPv6 interfaces.

```text
127.0.0.1:PORT
```

Means the service is listening only on localhost.

---

# 16. Nginx Validation

Nginx was tested locally with:

```bash
curl -I http://localhost
```

The server returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
```

Nginx service status was checked with:

```bash
sudo systemctl status nginx
```

The service was:

```text
active (running)
```

The server had:

- Nginx master process
- Six Nginx worker processes
- Low memory consumption
- Low accumulated CPU time

This confirmed that Nginx was running and responding locally.

---

# 17. Performance Troubleshooting Scenario

A simulated incident was used to practice the systems-engineering troubleshooting mindset.

### Incident

Users report:

> The server is slow.

Nginx is still responding, but requests are slower than normal.

### Step 1 — Check current resource pressure

Start with:

```bash
top
```

Hypothetical output showed:

- CPU around 95% busy
- Load average around `12.4, 10.8, 8.2`
- Server has 6 CPUs

This indicated significant CPU pressure.

### Step 2 — Identify the process

Use:

```bash
ps aux --sort=-%cpu | head
```

A hypothetical application worker appeared:

```text
/opt/app/worker.py
```

with approximately:

```text
385% CPU
```

Because the system has multiple CPUs, a process can exceed 100% when using multiple CPU cores.

Approximately 385% means the process was consuming about 3.85 CPU cores worth of CPU time.

### Step 3 — Investigate the process

The worker was processing uploaded files.

Repeated log entries referenced:

```text
customer_data.csv
```

Repeated processing was treated as a clue that the worker could be stuck in a retry/failure loop.

### Step 4 — Find the error

The simulated log showed:

```text
Permission denied: /uploads/customer_data.csv
```

The worker was running as:

```text
appuser
```

### Step 5 — Correct approach

The next investigation would be to check:

- File ownership
- File permissions
- Directory permissions
- The account running the application
- Whether the application actually needs access to the file

If the access is required and within scope, grant the **minimum required permission**.

Avoid:

```bash
chmod 777
```

because broad permissions are not an appropriate default fix.

### Troubleshooting cycle

The lab reinforced this workflow:

```text
Detect
  ↓
Isolate
  ↓
Investigate
  ↓
Identify root cause
  ↓
Apply controlled fix
  ↓
Verify recovery
```

---

# Lab 23 — Key Commands

| Purpose | Command |
|---|---|
| Live system monitoring | `top` |
| Process list | `ps aux` |
| Sort by CPU | `ps aux --sort=-%cpu \| head` |
| Sort by memory | `ps aux --sort=-%mem \| head` |
| Memory | `free -h` |
| Performance statistics | `vmstat 1 5` |
| Filesystem capacity | `df -h` |
| Directory usage | `du -sh` |
| Journal usage | `journalctl --disk-usage` |
| Interfaces | `ip addr` |
| Routing | `ip route` |
| IP connectivity | `ping -c 4 1.1.1.1` |
| DNS resolution | `getent hosts google.com` |
| Listening services | `ss -tulpn` |
| Nginx test | `curl -I http://localhost` |
| Service status | `systemctl status nginx` |

---

# Lab 24 — Backup & Restore

## Topics Covered

- `tar`
- `rsync`
- Scheduled backups
- Restore testing

---

# 1. Creating Test Data

A dedicated backup lab was created under:

```text
/opt/backup-lab/data
```

The test data included:

```text
customers.txt
config.txt
reports/
├── january.txt
└── february.txt
```

The files were verified with:

```bash
sudo find /opt/backup-lab/data -type f -ls
```

This created a controlled dataset for backup and restoration testing.

---

# 2. TAR Backup

A TAR archive was created to package the data.

The archive was then inspected using:

```bash
tar -tf backup.tar
```

This verified which files were contained in the archive.

---

# 3. TAR.GZ Compression

A compressed archive was also created:

```text
backup.tar.gz
```

Its contents were verified using:

```bash
tar -tzf backup.tar.gz
```

### TAR vs TAR.GZ

| Format | Purpose |
|---|---|
| `.tar` | Packages multiple files/directories together |
| `.tar.gz` | Packages and compresses the archive |

Compression can reduce storage requirements, while TAR provides the archive structure.

---

# 4. Separate Backup Location

A separate backup directory was used:

```text
/opt/backups
```

The purpose was to separate backup artifacts from the working dataset.

However, this is still on the **same VPS**.

### Production consideration

A backup stored on the same server does not protect against every disaster.

For production systems, backups should normally include an independent location such as:

- Remote storage
- Object storage
- Another server
- Off-site backup infrastructure

The important principle is:

> A backup is only useful if it survives the failure you are trying to protect against.

---

# 5. Restore Testing

Restore testing was performed by simulating data loss.

A file was deleted from the working dataset:

```bash
sudo rm /opt/backup-lab/data/customers.txt
```

The remaining files were checked:

```bash
sudo find /opt/backup-lab/data -type f -ls
```

The deleted file was then restored from the backup.

This demonstrated that creating a backup is only half of the process.

The other half is proving that the backup can be restored.

---

# 6. Rsync Backup

An Rsync backup location was created:

```text
/opt/rsync-backup
```

The initial synchronization was performed with:

```bash
rsync -av /opt/backup-lab/data/ /opt/rsync-backup/
```

The source and backup were compared:

```bash
sudo diff -r /opt/backup-lab/data /opt/rsync-backup
```

No differences were returned.

This confirmed that the backup matched the source at that point.

---

# 7. Incremental Synchronization

The configuration file was changed:

```text
config.txt
```

The updated content was:

```text
Updated application configuration
```

Before performing the actual synchronization, a dry run was used:

```bash
rsync -av --dry-run /opt/backup-lab/data/ /opt/rsync-backup/
```

The dry run showed what Rsync would change without modifying the destination.

The actual synchronization was then performed.

The result was verified with:

```bash
cat
```

and:

```bash
diff -r
```

### Why `--dry-run` matters

In operations work, a dry run provides a safer way to preview changes before applying them.

---

# 8. Single-File Restore with Rsync

The `customers.txt` file was deleted from the source:

```bash
sudo rm /opt/backup-lab/data/customers.txt
```

The file was restored from the Rsync backup:

```bash
sudo rsync -av /opt/rsync-backup/customers.txt /opt/backup-lab/data/
```

The restored file was verified:

```bash
sudo ls -lh /opt/backup-lab/data/customers.txt
```

Content was checked with:

```bash
sudo cat /opt/backup-lab/data/customers.txt
```

The content was:

```text
Customer database backup test
```

Finally:

```bash
sudo diff -r /opt/backup-lab/data /opt/rsync-backup
```

returned no differences.

This confirmed that the source and backup were synchronized again.

---

# 9. Backup Script

A backup script was created at:

```text
/usr/local/bin/backup-lab.sh
```

The script was made executable and tested manually.

The backup process demonstrated the basic automation pattern:

```text
Source
  ↓
Create backup
  ↓
Verify backup
  ↓
Retain recent backups
  ↓
Remove old backups
```

---

# 10. Existing Scheduled Backup Automation

An existing script was also inspected:

```text
/home/sysadmin/backup2.sh
```

The script:

- Requires a source directory argument
- Checks whether the source exists
- Creates a backup directory
- Creates timestamped `.tar.gz` backups
- Verifies the archive with `tar -tzf`
- Removes backups older than 7 days

This was useful because it showed how manual backup concepts can be turned into an operational process.

---

# 11. Cron Scheduling

The existing crontab contained:

```cron
0 2 * * * /home/sysadmin/backup2.sh /home/sysadmin/project_test >> /home/sysadmin/backup_lab/backup.log 2>&1
```

This schedules the backup every day at:

```text
02:00
```

Output and errors are redirected to:

```text
/home/sysadmin/backup_lab/backup.log
```

An existing log cleanup job was also present:

```cron
30 2 * * * /home/sysadmin/log_cleanup_lab/log_cleanup.sh >> /home/sysadmin/log_cleanup_lab/cleanup.log 2>&1
```

This demonstrates an important operational principle:

> Before creating new automation, inspect existing automation to avoid duplicate jobs.

---

# 12. TAR vs Rsync

| Feature | TAR / TAR.GZ | Rsync |
|---|---|---|
| Main purpose | Archive/backup | Synchronization |
| Compression | TAR.GZ supports compression | Does not inherently compress stored files |
| Incremental-style transfer | Not its main strength | Yes |
| Single-file restore | Possible by extracting | Very convenient |
| Archive portability | High | Destination remains file-based |
| Dry-run | Not the main workflow | Supported with `--dry-run` |
| Useful for | Backup archives | Replication/synchronization |

Both tools are useful, but they solve slightly different operational problems.

---

# Lab 24 — Key Commands

| Purpose | Command |
|---|---|
| Find files | `find /opt/backup-lab/data -type f -ls` |
| List TAR archive | `tar -tf backup.tar` |
| List TAR.GZ archive | `tar -tzf backup.tar.gz` |
| Rsync backup | `rsync -av SOURCE/ DEST/` |
| Rsync preview | `rsync -av --dry-run SOURCE/ DEST/` |
| Compare directories | `diff -r SOURCE DEST` |
| Restore file | `rsync -av BACKUP/file SOURCE/` |
| Inspect cron | `crontab -l` |
| Inspect backup script | `cat /home/sysadmin/backup2.sh` |

---

# Operational Lessons

## 1. Monitoring is about relationships

A systems engineer should not look at one number in isolation.

For example:

```text
High CPU
```

should lead to:

```text
Which process?
Why is it consuming CPU?
How long has it been doing this?
What changed?
Is the service affected?
```

---

## 2. Symptoms are not root causes

A slow server is a symptom.

High CPU may be another symptom.

The actual root cause could be an application retry loop, permissions problem, inefficient process, or another issue.

The troubleshooting process should move from:

```text
Symptom → Evidence → Cause → Fix → Verification
```

---

## 3. Backups must be tested

A backup file existing on disk does not prove that the backup is usable.

The lab deliberately:

1. Created data.
2. Backed it up.
3. Deleted data.
4. Restored data.
5. Compared the restored state with the backup.

This is much stronger evidence than simply checking that a `.tar.gz` file exists.

---

## 4. Use least privilege

When troubleshooting permissions, avoid using overly broad permissions such as:

```bash
chmod 777
```

Instead, identify:

- Which user needs access
- Which group should have access
- Which directory/file requires access
- What minimum permission is necessary

---

## 5. Verify before and after changes

Useful operational habits include:

```bash
rsync --dry-run
```

before synchronization and:

```bash
diff -r
```

after synchronization.

Similarly, after changing a service:

```bash
systemctl status SERVICE
```

and an application-level test such as:

```bash
curl -I http://localhost
```

can verify that the service actually works.

---

# Final Results

## Lab 23 — System Monitoring

Completed:

- [x] CPU monitoring
- [x] Controlled CPU load test
- [x] Load-average analysis
- [x] Process investigation with `ps`
- [x] Memory monitoring
- [x] Controlled memory test
- [x] `vmstat` performance monitoring
- [x] Disk capacity investigation
- [x] Directory and log investigation
- [x] Journal usage investigation
- [x] Network interface inspection
- [x] Routing inspection
- [x] Connectivity testing
- [x] DNS testing
- [x] Listening-port inspection
- [x] Nginx service validation
- [x] Performance troubleshooting scenario

## Lab 24 — Backup & Restore

Completed:

- [x] Created backup test data
- [x] Created TAR backup
- [x] Created TAR.GZ backup
- [x] Verified archive contents
- [x] Created separate backup location
- [x] Tested restore after deletion
- [x] Created Rsync backup
- [x] Used Rsync dry run
- [x] Tested incremental synchronization
- [x] Tested single-file restore
- [x] Compared source and backup with `diff`
- [x] Created/tested backup automation
- [x] Inspected existing backup script
- [x] Inspected existing cron scheduling
- [x] Verified scheduled backup logging
- [x] Reviewed retention and cleanup behavior

---

# Systems Engineer Mindset

The most important lesson from Phase 9 is not memorizing commands.

It is learning to think operationally:

```text
Observe
   ↓
Measure
   ↓
Identify the abnormal behavior
   ↓
Find the responsible component
   ↓
Investigate the evidence
   ↓
Make the smallest safe change
   ↓
Verify the result
   ↓
Document what happened
```

For backups:

```text
Backup
   ↓
Verify
   ↓
Restore
   ↓
Verify restoration
   ↓
Automate
   ↓
Monitor
```

A production systems engineer should be able to answer not only:

> "Do we have a backup?"

but also:

> "Can we restore it, how long will it take, and how do we know the restored data is correct?"

---

# Phase 9 Status

**Phase 9 — Monitoring & Operations: Completed Labs 23–24**

The next planned lab is:

## Lab 25 — Troubleshooting Scenarios

Planned scenarios:

- SSH failures
- Full disk
- Nginx will not start
- DNS issues
- Permission problems
- Service failures

The goal will be to apply the same troubleshooting methodology from Lab 23 to realistic Linux administration incidents.
