# Automation

## Server Automation

### Overview

This was my first proper automation-focused Linux lab.

The main idea I wanted to understand was how to take tasks that I would normally do manually as a system administrator and turn them into repeatable scripts and scheduled jobs.

I worked through user creation, backup automation, cron scheduling, log cleanup and a small disk monitoring script.

I also tried to keep the same approach I have used in the earlier labs: understand what the command is doing, test it, check the result, and only then automate it.

---

## What I Worked On

- User creation and verification
- Bash scripts for creating multiple users
- Password configuration and password expiry
- Backup creation with `tar` and `gzip`
- Backup verification and restore testing
- Backup retention using `find`
- Cron scheduling
- Log cleanup automation
- Basic disk usage monitoring
- Error handling and exit codes
- Testing both successful and failed conditions

---

# 1. User Creation

I started by creating users manually so I could understand what happens before trying to automate it.

I used:

```bash
id username
getent passwd username
ls -ld /home/username
```

`id` helped me check whether a user existed and showed the UID, GID and group membership.

`getent passwd` was useful for checking the account information stored through the system's user database.

I created several test users during the lab, including:

```text
testuser
alice
bob
charlie
david
emma
frank
george
helen
iris
james
```

One useful thing I noticed was that creating an account and completely configuring an account are not necessarily the same thing. For example, some of the early users were created even when the password setup did not complete correctly.

That made me think more carefully about verification instead of assuming that a command succeeding halfway through meant the whole task was finished.

---

# 2. Automating User Creation

I created several versions of the user creation script while improving it.

The first version could accept usernames as arguments:

```bash
./create_users.sh alice bob charlie
```

I then added checks so the script would:

- Require at least one username
- Check whether the user already exists
- Create users only when necessary
- Report failures
- Return an appropriate exit code
- Verify that the user was actually created

The later version, `create_users4.sh`, also generated a temporary password for each new user and forced a password change at first login.

I tested this with `iris` and `james`.

I also tested the script again with users that already existed:

```text
iris already exists. Skipping.
james already exists. Skipping.
```

This helped me understand the idea of **idempotency**.

The script should be safe to run again without trying to recreate users that are already there.

I also tested the account by switching to `iris` with:

```bash
su - iris
```

and verified:

```bash
whoami
pwd
id
```

This confirmed that the account and home directory were working as expected.

---

# 3. Backup Automation

Next I worked on backup automation.

I first created some controlled test data:

```text
~/backup_lab/data/
├── lab_notes.txt
└── test.txt
```

I used `tar` to understand how archiving works before turning it into a script.

For example:

```bash
tar -cvf ~/backup_lab/backup.tar ~/backup_lab/data
```

I then created a compressed archive:

```bash
tar -czvf ~/backup_lab/backup.tar.gz ~/backup_lab/data
```

The important distinction I learned was that `tar` packages files into an archive while `gzip` provides compression.

I also restored the archive into a separate test directory and checked that the files were recovered correctly.

---

# 4. Backup Script

I created `backup.sh` first and then improved it into `backup2.sh`.

The second version accepts the source directory as an argument:

```bash
./backup2.sh ~/project_test
```

This made the script more reusable than having the source directory permanently hard-coded.

The script performs several checks:

```text
Check source directory
        ↓
Create backup directory
        ↓
Create timestamped .tar.gz
        ↓
Verify the archive
        ↓
Remove backups older than the retention period
        ↓
Return success or failure
```

I created a test directory containing:

```text
config.txt
readme.txt
```

The backup completed successfully and I verified the archive contents with:

```bash
tar -tzf ~/backup_lab/backups/backup_*.tar.gz
```

The archive contained the expected project files.

I also tested an invalid source directory:

```bash
./backup2.sh ~/does_not_exist
```

The script correctly returned an error and exit code `1`.

That was important because I don't want an automation script to report success when the requested operation actually failed.

---

# 5. Backup Retention

I also added retention handling to the backup process.

I used:

```bash
find ~/backup_lab/backups -type f -mtime +7
```

to identify backup files older than the retention period.

Before allowing the command to delete anything, I tested it with `-print`.

I created a fake old backup:

```bash
touch ~/backup_lab/backups/old_test_backup.tar.gz
touch -d "10 days ago" ~/backup_lab/backups/old_test_backup.tar.gz
```

The file was correctly identified as older than seven days.

I then tested:

```bash
find ~/backup_lab/backups -type f -mtime +7 -print -delete
```

and confirmed that the old test backup was removed while the current backup remained.

This was a good reminder that destructive commands should be tested with a non-destructive version first.

---

# 6. Cron Scheduling

After the scripts were working manually, I moved on to scheduling them.

I checked the local documentation with:

```bash
man cron
man 5 crontab
```

At first my user did not have a crontab:

```text
no crontab for sysadmin
```

I created one with:

```bash
crontab -e
```

I also checked that the cron service itself was running:

```bash
systemctl status cron --no-pager
```

The service was active and running.

Before scheduling the backup, I tested cron with a simple job that wrote the current date to a log file every minute.

That confirmed that cron was actually executing jobs rather than just having a configuration entry that looked correct.

---

# 7. Scheduled Backup

Once I was confident that cron was working, I scheduled the backup.

The final backup entry was:

```cron
0 2 * * * /home/sysadmin/backup2.sh /home/sysadmin/project_test >> /home/sysadmin/backup_lab/backup.log 2>&1
```

This means the backup runs every day at **02:00**.

I used absolute paths because scheduled jobs do not necessarily run with the same environment as my interactive shell.

The backup output is also redirected to:

```text
~/backup_lab/backup.log
```

so I have something to check when troubleshooting an automated run.

---

# 8. Log Cleanup

After backup automation, I created a small controlled log environment:

```text
~/log_cleanup_lab/
├── app.log
├── app.log.1
└── app.log.2
```

I changed the timestamps of the rotated logs so that they appeared to be 10 and 15 days old.

I then used:

```bash
find ~/log_cleanup_lab -type f -mtime +7 -print
```

to identify files older than seven days.

The two old log files were found while the current `app.log` was left alone.

I tested the deletion separately before putting it into the script.

---

# 9. Log Cleanup Script

I created:

```text
~/log_cleanup_lab/log_cleanup.sh
```

The script uses a seven-day retention period.

I also added validation so the script reports an error if the log directory does not exist.

I tested this by creating a separate test copy and changing the directory to a path that did not exist.

The script returned:

```text
ERROR: Log directory does not exist: /home/sysadmin/non_existent_log_directory
```

and the exit code was:

```text
1
```

After restoring the correct directory, the script completed successfully.

When there were no old files left to remove, it reported:

```text
No old log files found.
```

and returned exit code `0`.

---

# 10. Scheduling Log Cleanup

I then added the cleanup script to cron.

The final entry is:

```cron
30 2 * * * /home/sysadmin/log_cleanup_lab/log_cleanup.sh >> /home/sysadmin/log_cleanup_lab/cleanup.log 2>&1
```

This means the cleanup runs every day at **02:30**.

The final schedule is therefore:

```text
02:00 → Backup project_test
02:30 → Log cleanup
```

I also temporarily tested the cleanup job every minute so I did not have to wait until 02:30 to prove that cron was executing it.

After the test, I restored the intended 02:30 schedule.

---

# 11. Basic Disk Monitoring

For the final part of this lab, I wanted to connect automation with monitoring without turning this into the full System Monitoring lab.

I used:

```bash
df -h /
```

to check the root filesystem.

The server was using about:

```text
4%
```

of the root filesystem.

I then extracted just the percentage with:

```bash
df -h / | awk 'NR==2 {print $5}'
```

which returned:

```text
4%
```

I removed the `%` sign so the value could be compared numerically:

```bash
df -h / | awk 'NR==2 {gsub("%","",$5); print $5}'
```

which returned:

```text
4
```

This was a useful little exercise in taking command output and turning it into data that a script can actually make a decision about.

---

# 12. Disk Monitoring Script

I created:

```text
~/log_cleanup_lab/disk_monitor.sh
```

The script uses a threshold of `80%`.

The basic logic is:

```text
Get disk usage
      ↓
Compare usage with threshold
      ↓
If usage >= threshold
      ↓
WARNING
Otherwise
      ↓
OK
```

With the server at 4%, the normal test returned:

```text
Disk usage is 4%. OK.
```

I also tested the warning branch by temporarily lowering the threshold below the actual usage.

The script then returned:

```text
WARNING: Disk usage is 4%.
```

After testing, I restored the threshold to `80`.

I deliberately kept this monitoring script small because the next lab is dedicated to system monitoring in more depth.

---

# 13. What I Learned

The biggest thing I took away from this lab is that automation is not just about writing a script and making it executable.

The script also needs to know:

- What input it expects
- What should happen when something is missing
- How to detect failure
- What exit code to return
- How to verify the result
- What should happen if it is run again
- How it will behave when executed automatically

The user creation exercise helped me understand idempotency.

The backup exercise helped me understand validation and retention.

Cron helped me understand the difference between a script that works manually and a script that can be executed automatically.

The log cleanup exercise showed me why destructive commands should be tested carefully.

The monitoring exercise showed me the basic pattern of:

```text
metric → threshold → decision
```

I also learned that I shouldn't automatically use cron for everything. It is useful for scheduled administrative tasks such as backups and cleanup, but proper enterprise monitoring is a separate topic and normally involves dedicated monitoring and alerting systems.

---

# 14. Mistakes and Troubleshooting

I made a few small mistakes during the lab, which were actually useful.

At one point I typed:

```bash
dh -h /
```

instead of:

```bash
df -h /
```

The system showed that `dh` was not a valid command, and I corrected it.

I also initially left the log cleanup cron job running every minute while testing it. I checked the crontab again and restored it to the intended 02:30 schedule.

There were also some early password configuration problems while creating test users. This reinforced the importance of checking the final state instead of assuming that the whole operation succeeded.

These are small things, but they are exactly the kind of issues I expect to encounter when working with real systems.

---

# 15. Key Commands

Some of the main commands I used in this lab were:

```bash
id
getent passwd
adduser
useradd
chpasswd
passwd -S
passwd -e

tar
tar -tzf
find
touch -d

crontab -l
crontab -e
systemctl status cron

df -h
awk
chmod
echo $?
```

I also used the manual pages instead of relying only on copied commands:

```bash
man cron
man 5 crontab
man find
adduser --help
```

That is becoming an important part of how I want to learn Linux: understand the tool first, then use the syntax.

---

# 16. Final Automation Setup

At the end of the lab, the main automated tasks were:

```text
Daily at 02:00
    ↓
backup2.sh
    ↓
Backup project_test
    ↓
Verify archive
    ↓
Clean old backups


Daily at 02:30
    ↓
log_cleanup.sh
    ↓
Check for old logs
    ↓
Remove files outside retention period
    ↓
Write output to cleanup.log
```

The disk monitoring script was tested manually as a small introduction to monitoring automation.

---

# Lab Status

**Lab 16 — Server Automation: COMPLETE**

I now have practical experience with:

- Bash automation
- User provisioning
- Backup scripting
- Backup verification
- Retention policies
- Cron scheduling
- Log cleanup
- Basic monitoring logic
- Error handling
- Exit codes
- Idempotent scripts
- Testing automated tasks

---

# Final Thoughts

This lab felt different from the earlier Linux labs because I wasn't only troubleshooting or configuring the server anymore. I was starting to make the server do some of the work for me.

The part I found most useful was seeing how the pieces connect:

```text
Manual task
    ↓
Understand the command
    ↓
Write a script
    ↓
Test the script
    ↓
Handle failure
    ↓
Verify the result
    ↓
Schedule it
    ↓
Let the system do the work
```

That is probably the main reason automation is such an important part of system administration.

I'm also starting to see the difference between knowing Linux commands and actually thinking like a systems administrator. The command is only one part of the job. I also need to understand what I am trying to accomplish, what could go wrong, and how I will know that the task actually worked.

---

# Next

The next lab in this phase is **Lab 17 — System Monitoring**.

That will be a deeper monitoring exercise covering:

- Disk usage
- Memory
- CPU
- Network
- Performance

I kept the monitoring part of this lab intentionally small so that Lab 23 can focus on understanding those areas properly rather than repeating the automation exercise.
