# Day 04 — Linux Practice: Processes and Services (Solved)

This is a beginner-friendly, step-by-step practice note for Day 04. It walks you through at least six commands (process, service, and log checks), explains what and why, shows example outputs, gives macOS alternatives, points out common mistakes, and provides interview-focused review questions.

> Note: The original assignment expects a Linux environment (systemd, journalctl). Your machine may be macOS — in that case use the "macOS alternatives" shown below. Wherever I show example output, treat it as a sample: run the commands on your machine to capture your real output and replace the samples.

---

## Table of contents

- Process checks (2+ commands)
- Service checks (2+ commands)
- Log checks (2+ commands)
- Mini troubleshooting flow (one service)
- Common mistakes and debugging tips
- Tool purpose and industry usage
- Summary, takeaways, interview questions, and revision topics

---

## Environment note

- These commands are written for Linux (systemd). If you're on macOS, I provide alternatives in each section.
- Commands that require root/sudo will be marked with `sudo` — run them only when you trust what they do.

---

## 1) Process checks

Goal: Learn how to list processes, find specific processes, and observe resource usage.

Commands used: `ps`, `top`, `pgrep` (examples below show `ps` and `top`).

Why:
- Processes are programs in execution. Knowing how to inspect them is a primary troubleshooting skill.
- Real-life analogy: processes are like workers in a factory — `ps` gives you a snapshot attendance list, `top` is a live dashboard.

1.1 List running processes (snapshot) using `ps`

```bash
# Show all processes with full details. Explanations:
# - `a` : show processes for all users
# - `u` : display user-oriented format (owner, cpu%, mem%, start time)
# - `x` : include processes without a controlling terminal
ps aux | head -n 12

# Example output (sample). Replace with your machine's output when you run it.
# USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# root         1  0.0  0.1 169084  6528 ?        Ss   Feb12   0:04 /sbin/init
# root       245  0.0  0.2  89236  8848 ?        Ss   Feb12   0:01 /usr/lib/systemd/systemd-journald
# sshd      1024  0.1  0.3 124392 12084 ?        Ss   Feb12   0:02 /usr/sbin/sshd -D

# Notes on interpreting columns:
# - USER: process owner
# - PID: process id (unique number)
# - %CPU / %MEM: resource usage
# - VSZ / RSS: virtual / resident memory sizes (kilobytes)
# - STAT: process state (S=sleeping, R=running, D=uninterruptible sleep, Z=zombie)

```

How it works internally:
- `ps` reads the kernel's process table and prints a snapshot at the time you run it. It does not refresh.

macOS alternative:
- `ps aux` works on macOS as well; columns and behavior are similar.

Common beginner mistakes:
- Expecting `ps` to update live — it doesn't. Use `top` for a live view.
- Interpreting VSZ/RSS incorrectly — RSS is the memory actually resident in RAM.

Quick summary (Process checks): Use `ps aux` for snapshots and inspect USER, PID, %CPU, %MEM, STAT.

Key takeaway: `ps` gives a reliable static picture; use it to capture evidence for debugging or ticketing.

---

1.2 Live process view using `top` (or `htop` if available)

```bash
# Start top in an interactive terminal. Use q to quit.
top

# Non-interactive snapshot (useful for capturing output into a file):
top -b -n 1 | head -n 20

# Example (sample):
# top - 10:00:00 up  5:22,  2 users,  load average: 0.12, 0.09, 0.05
# Tasks: 123 total,   1 running, 122 sleeping,   0 stopped,   0 zombie
# %Cpu(s):  1.2 us,  0.5 sy,  0.0 ni, 98.0 id,  0.2 wa,  0.0 hi,  0.1 si,  0.0 st
# Mem:  16384256k total, 1048576k used, 15335680k free,  123456k buffers

# Explanation:
# - top is like a live operations dashboard showing CPU, memory, and top resource-consuming processes.

```

How it works internally:
- `top` polls the kernel for process and CPU stats and refreshes them on-screen every few seconds.

macOS alternative:
- `top` exists on macOS but the flags differ. For a macOS-friendly snapshot:
  `top -l 1 | head -n 20`

Common mistakes:
- Running `top` in a narrow terminal and missing columns — widen your terminal or pipe to `head` for snapshots.
- Not using `htop` if you want an easier UI (if available, install via your package manager).

Quick summary: `top` provides the live view; great for spotting spikes and noisy processes.

---

## 2) Service checks

Goal: Inspect system services (systemd) — status, enabled/disabled, and unit listing.

Note: systemd is the init system on many Linux distributions. It manages services (units). On macOS, `launchd` is analogous.

Commands used: `systemctl status`, `systemctl list-units`.

Why: Services provide functionality to the system (e.g., SSH, web server). When users report issues, check the service status first.

2.1 Check a service status (example: `sshd`)

```bash
# Show detailed status for the SSH daemon (sshd). Use sudo if needed.
sudo systemctl status sshd

# Example output (sample):
# ● sshd.service - OpenSSH Daemon
#    Loaded: loaded (/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
#    Active: active (running) since Fri 2026-05-15 10:00:00 UTC; 2 days ago
#   Process: 1023 ExecStart=/usr/sbin/sshd -D $SSHD_OPTS (code=exited, status=0/SUCCESS)
#  Main PID: 1024 (sshd)
#     Tasks: 1 (limit: 4915)
#    CGroup: /system.slice/sshd.service
#            └─1024 /usr/sbin/sshd -D

# Explanation of important lines:
# - Loaded: whether the unit file is present and from where
# - Active: running / exited / failed — the most important line for health
# - Main PID: process id of the main daemon process

```

macOS alternative (OpenSSH server on macOS):

```bash
# Check sshd status on macOS (if running via launchd or brew):
# If using launchd (system sshd):
sudo launchctl list | grep sshd

# If you installed sshd via Homebrew `brew services`:
brew services list | grep sshd

```

Common mistakes:
- Looking only at `Loaded:` and missing `Active:` which indicates runtime health.
- Not using `sudo` when required — `systemctl status` may show limited info without sudo for system units.

Quick summary: `systemctl status <unit>` is your first stop to know if a service is running and why it might have failed.

---

2.2 List active units (services)

```bash
# List loaded units with their state, filtered to services only
systemctl list-units --type=service --state=running

# Example output (sample):
# UNIT                         LOAD   ACTIVE SUB     DESCRIPTION
# sshd.service                 loaded active running OpenSSH Daemon
# cron.service                 loaded active running Regular background program processing daemon
# docker.service               loaded active running Docker Application Container Engine

# To see all units including inactive: systemctl list-units --type=service

```

How it works internally:
- `systemctl` talks to the systemd process (PID 1) via D-Bus to query unit states and properties.

macOS alternative:
- `brew services list` (for brew-managed services) or `launchctl list` for launchd-managed services.

Common mistakes:
- Expecting `systemctl` on non-systemd OSes (macOS, some containers). Know your platform.

Quick summary: `systemctl list-units` gives you an inventory of what systemd runs now.

---

## 3) Log checks

Goal: Inspect logs for a given service to find recent errors and clues.

Commands used: `journalctl -u <service>`, `tail -n 50 <logfile>` (or `journalctl -n 200`), plus examples.

Why: Logs are the ground truth. They tell you what happened and when.

3.1 Service logs with `journalctl` (systemd)

```bash
# Show recent logs for sshd (unit name may be ssh or sshd depending on distro). -r shows newest first.
sudo journalctl -u sshd -n 100 --no-pager

# Explanation:
# - `-u sshd` : filter for the sshd unit
# - `-n 100`  : last 100 lines
# - `--no-pager`: avoid using less, prints directly

# Example output (sample):
# May 17 10:01:23 myhost sshd[2048]: Accepted publickey for user from 10.0.0.1 port 52622 ssh2: RSA SHA256:...
# May 17 11:22:05 myhost sshd[3101]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0

```

macOS alternative:
- macOS does not use journalctl. Use `log show` or legacy `/var/log` files. Example:

```bash
# Show system log entries for sshd on macOS (sample):
sudo log show --predicate 'process == "sshd"' --last 1h

# Or check legacy logs (if available):
sudo tail -n 200 /var/log/system.log | grep sshd

```

Common mistakes:
- Forgetting `--no-pager` in automation (scripted checks should avoid interactive pagers).
- Assuming logs are only in one place — web apps often log elsewhere (app-specific logs).

Quick summary: `journalctl -u <unit>` is the fastest way to see what a systemd-managed service has recently reported.

---

3.2 File tailing: `tail -n 50` or `tail -f` for live logs

```bash
# Tail a file (last 50 lines) for quick check
sudo tail -n 50 /var/log/auth.log

# For live monitoring, follow the file (press Ctrl+C to stop):
sudo tail -n 200 -f /var/log/auth.log

# Explanation:
# - `-n 50`: last 50 lines
# - `-f`: follow new lines as file grows (good for watching logs while reproducing an issue)

# Sample output (sample lines):
# May 17 11:22:05 myhost sshd[3101]: Failed password for invalid user root from 10.0.0.2 port 51234 ssh2

```

How it works internally:
- `tail` reads the end of a file and, with `-f`, keeps the file descriptor open and prints appended content.

macOS alternative:
- Same `tail` commands work on macOS for plaintext logs.

Common mistakes:
- Not running with `sudo` when the log file is only readable by root.
- Confusing `tail -f` with `less +F` — both can follow files but `less` is interactive.

Quick summary: `tail -n` for snapshots, `tail -f` for live troubleshooting when reproducing issues.

---

## 4) Mini troubleshooting flow (pick one service: `sshd`)

Goal: Demonstrate a short troubleshooting session for `sshd` (SSH daemon). This shows a typical beginner-friendly incident workflow.

Scenario: Users report they cannot SSH into the server.

Step 0 — Gather quick facts

```bash
# 1) Can I reach the host? (network check)
ping -c 4 your-server.example.com

# 2) Is sshd process running?
ps aux | grep [s]shd

# 3) Listen sockets (is ssh listening on port 22?)
sudo ss -tlnp | grep :22

# macOS alternative for listening sockets:
# sudo lsof -iTCP -sTCP:LISTEN -P | grep 22

```

What we are doing & why:
- `ping`: tests basic network reachability (ICMP). If ping fails, the problem may be networking rather than SSH.
- `ps` and `ss`/`lsof`: confirm the sshd process and whether it's listening on the expected port.

Interpretation of sample outputs:
- `ps` shows a running `sshd` with PID 1024: process is present.
- `ss` shows `LISTEN` on 0.0.0.0:22: sshd is listening.

If sshd is not running

```bash
# Try to start the service and then re-check status/logs
sudo systemctl start sshd
sudo systemctl status sshd
sudo journalctl -u sshd -n 50 --no-pager

# macOS (if using brew services):
brew services start sshd
```

If sshd fails to start, look at the logs printed by `journalctl` and the unit `status` details for hints (bad config, port in use, permission issue).

Common troubleshooting findings and fixes:
- Error: `Address already in use` → another service is bound to port 22. Use `ss -tlnp` to find it and fix the conflicting service.
- Error: `Permission denied` on host keys → check ownership/permissions in `/etc/ssh/` (keys should be root-owned with strict permissions).
- Error: PAM/auth failures → check `/var/log/auth.log` or `journalctl` for authentication messages.

Quick summary: The sequence is reachability → process existence → listen socket → start/restart → read logs.

Key takeaway: Collect facts (commands above) before making changes; logs and `systemctl status` often tell the root cause.

---

## 5) Tools explained (brief)

- `ps`: process snapshot utility. Industry: used in incident tickets and debugging scripts.
- `top` / `htop`: live process viewer for performance triage.
- `systemctl`: systemd service manager. Industry: standard in modern Linux servers (Debian, Ubuntu, CentOS/RHEL 7+). Used in operations to manage daemons and timers.
- `journalctl`: systemd's centralized logging interface. Industry: primary log source on systemd systems.
- `ss` / `netstat` / `lsof`: tools to inspect sockets and ports. Industry: used to detect binding conflicts.
- `tail`: basic log inspection and live following. Industry: simple, universal for log watching.

Real-world usage:
- Cloud servers, on-call rotations, and CI runners rely on these tools to diagnose failing services.

---

## 6) Common mistakes and how to avoid them

- Not checking logs: always inspect logs (`journalctl`, `/var/log/*`) after service failures.
- Restarting randomly: collect evidence (ps, top, logs) before restarting; restarting may hide intermittent problems.
- Wrong platform assumptions: `systemctl` won't work on macOS — learn platform-specific tools.
- Forgetting `sudo`: many service and log files are root-only — run commands with `sudo` responsibly.

---

## 7) Short checklist to include in your submission

- At least 6 commands captured (I used many more in examples).
- 2 process commands: `ps aux`, `top` (or `pgrep`)
- 2 service commands: `systemctl status sshd`, `systemctl list-units --type=service`
- 2 log commands: `journalctl -u sshd -n 100`, `tail -n 50 /var/log/auth.log`
- Picked service inspected: `sshd` (OpenSSH daemon). Replace with `docker` or `cron` if preferred.

---

## 8) Interview-focused questions and revision topics

- What is the difference between a process and a service?
- How do you find which process is listening on port 80?
- How does `systemd` differ from older init systems like SysVinit?
- What does `%MEM` vs `RSS` mean in `ps` output?
- How do you follow logs in real time while reproducing an issue?

Revision topics:
- Linux process states (R, S, D, Z)
- Basic networking checks (`ping`, `ss`, `netstat`, `curl`)
- systemd basics: units, targets, timers, `systemctl` usage
- Basics of logging with `journalctl` and `/var/log`

Where this is used in real companies/projects:
- On-call responders in SRE teams use these commands to triage production incidents.
- DevOps engineers use `systemctl`, `journalctl`, and `ss` when deploying and validating services on VMs.
- Container hosts may use similar tools inside base images or rely on container-specific logging (docker logs, kubectl logs).

---

## 9) Final notes — how to submit

1. Replace the sample outputs above by running the commands on your Linux VM or environment and copying the real output.
2. Save this file as `linux-practice.md` in `2026/day-04/` (already created by this tutorial).
3. Commit and push to your fork as the original assignment requests.

Example git commands to commit and push (run in the repository root):

```bash
# Stage the file
git add 2026/day-04/linux-practice.md

# Commit with a clear message
git commit -m "Day 04: Linux practice - process and service checks (linux-practice.md)"

# Push to your fork (replace origin/branch as appropriate)
git push origin main
```

---

If you want, I can:

- Run these commands on your behalf (I cannot run them on your local machine, but I can run them in a container if you want an isolated Linux environment here).
- Or, update this file with your actual command outputs if you paste them here and I will format them and annotate them inline.

Good work — want me to format the same notes for another service (e.g., `docker`), or convert this into a short slide-friendly summary for LinkedIn?
