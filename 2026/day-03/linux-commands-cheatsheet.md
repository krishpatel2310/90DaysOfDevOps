# Day 03 — Linux Commands Cheat Sheet (Practical, Beginner-friendly)

Purpose: a compact, easy-to-scan cheat sheet focused on process management, filesystem tasks, and networking troubleshooting. Each command includes a one-line usage note, a short explanation, and example with comments for beginners.

---

## Process Management (10 commands)

1. ps aux  # Snapshot of all processes with owner and resource usage
```bash
# Show all processes (user, PID, CPU%, MEM%, command)
ps aux
```

2. top  # Live, updating view of CPU and memory usage (press `q` to quit)
```bash
# Interactive live view — press `q` to exit
top
```

3. htop  # Improved `top` with keyboard navigation (install separately)
```bash
# If installed, provides colored, interactive process view
htop
```

4. pstree  # Show process tree (parent-child relationships)
```bash
# Visual parent/child process layout
pstree -p    # '-p' shows PIDs
```

5. pgrep / pkill  # Find or kill processes by name
```bash
# Find PIDs matching name
pgrep nginx
# Kill processes matching name (sends SIGTERM)
pkill -f myscript.py
```

6. kill (and kill -9)  # Send signals to specific PID
```bash
# Graceful termination (SIGTERM)
kill 1234
# Force kill (SIGKILL) — avoids cleanup; use only when necessary
kill -9 1234
```

7. nice / renice  # Adjust scheduling priority for CPU sharing
```bash
# Start process with lower priority (niceness +10)
nice -n 10 ./long-task.sh
# Change priority of running process
renice -n 5 -p 4321
```

8. strace  # Trace system calls of a process (advanced debugging)
```bash
# Trace syscalls for PID 2345 — helpful to see what a process is waiting on
strace -p 2345
```

9. lsof  # List open files and network sockets by process
```bash
# Show files opened by PID 1234
lsof -p 1234
# Show which process is listening on port 80
lsof -i :80
```

10. ss  # Show sockets and listening ports (modern netstat replacement)
```bash
# List TCP/UDP listening sockets with process info
ss -tulwn
```

### Summary (Process)

- Use `ps`, `top`, and `htop` to see who is using CPU/memory.
- Use `pstree`, `pgrep`, and `pkill` for parent/child and name-based ops.
- Use `lsof`, `ss`, and `strace` for deeper debugging.

Key takeaways:
- Start with `ps`/`top`. Only use `kill -9` when gentle shutdown fails.

---

## Filesystem & Logs (8 commands)

11. ls  # List directory contents with details
```bash
# Show all files with human-readable sizes
ls -lah
```

12. du  # Disk usage of directories/files
```bash
# Summarize disk usage of current directory in human-readable form
du -sh .
```

13. df  # Disk free space for mounted filesystems
```bash
# Show filesystem usage in human-readable form
df -h
```

14. find  # Search files by name, size, time, and run actions
```bash
# Find files modified in last 1 day
find /var/log -mtime -1 -type f
```

15. tail -f  # Follow growing log file in real time
```bash
# Follow last lines of a log; useful during troubleshooting
tail -n 200 -f /var/log/syslog
```

16. less / cat / head  # Read files (less is interactive pager)
```bash
# Page through a file, navigate with arrows, 'q' to quit
less /var/log/syslog
# Show first 20 lines
head -n 20 /etc/passwd
# Show whole file (not recommended for large files)
cat /etc/hosts
```

17. chmod / chown  # Change permissions and ownership
```bash
# Add execute permission for owner
chmod u+x ./deploy.sh
# Change owner to user 'appuser'
chown appuser:appuser /var/www/html -R
```

18. tar / gzip  # Package and compress files
```bash
# Create compressed archive
tar czf site-backup.tgz /var/www/html
# Extract archive
tar xzf site-backup.tgz
```

### Summary (Filesystem)

- Use `ls`, `du`, and `df` to inspect files and disk space.
- Use `find` and `tail -f` for log discovery and live troubleshooting.

Key takeaways:
- Always check disk space first when services act strange — full disks often cause failures.

---

## Networking Troubleshooting (at least 3 commands required; included more)

19. ping  # Test basic reachability and latency to an IP/host
```bash
# Send ICMP pings; Ctrl+C to stop. Shows packet loss and round-trip times
ping -c 5 8.8.8.8
```

20. ip addr / ip link  # Show network interfaces and IP addresses
```bash
# Show all interfaces and addresses
ip addr show
# Show link-level info (up/down)
ip link show
```

21. dig  # DNS lookup and debugging (from BIND dig tool)
```bash
# Query A records for example.com
dig example.com +short
```

22. curl  # Fetch HTTP resources, useful for HTTP health checks
```bash
# Simple HTTP GET and show headers and body
curl -v https://example.com/
# Use --fail to return non-zero on HTTP errors (useful in scripts)
curl --fail -sS https://example.com/ -o /dev/null
```

23. traceroute  # Show path packets take to a destination (where they hop)
```bash
# Trace route to host; useful to identify network hop problems
traceroute example.com
```

24. tcpdump  # Capture packets on an interface (advanced; needs privileges)
```bash
# Capture 200 packets on eth0 and write to file (requires sudo)
sudo tcpdump -i eth0 -c 200 -w capture.pcap
```

25. nslookup / netcat (nc)  # Quick DNS check and raw TCP/UDP testing
```bash
# DNS check
nslookup example.com
# Check if a TCP port is open (replace host/port)
nc -vz example.com 22
```

### Summary (Networking)

- Start with `ping` and `ip addr` to check connectivity and interface state.
- Use `dig`/`nslookup` to inspect DNS issues.
- Use `curl` to verify HTTP endpoints and `tcpdump`/`traceroute` for deeper network path analysis.

Key takeaways:
- Networking problems are often DNS or routing; validate DNS first, then reachability, then service.

---

## Common mistakes and how to avoid them

- Using `kill -9` as a first attempt: try `kill` (SIGTERM) first to allow cleanup.
- Ignoring disk usage: use `df -h` and `du -sh` before restarting services.
- Running heavy commands on production without testing: use `--dry-run` or test on staging.
- Forgetting to run `sudo` when needed — read permission errors carefully before changing owners/permissions.

---

## How these tools are used in industry (short)

- Purpose: Rapid investigation and remediation for incidents on servers and cloud VMs.
- Real-world usage: On-call engineers use these commands when debugging web servers, databases, and networking issues.

Example workflow during an incident:
1. `top` to see CPU spikes.
2. `ps aux` and `pstree` to find runaway processes.
3. `journalctl -u <service>` and `tail -f` logs to read errors.
4. `df -h` to rule out disk full.
5. `curl` to check HTTP health and `ip addr`/`ping` for network issues.

---

## Interview-focused questions & revision checklist

- What commands would you run first when a web server is slow? (Answer: `top`, `ps`, `df`, `journalctl`, `curl`)
- How do you find which process is listening on port 5432? (Answer: `ss -tulwn` or `lsof -i :5432`)
- How do you inspect recent logs for a systemd service? (Answer: `journalctl -u <service> -n 200 --no-pager`)

Topics to revise:
- Process lifecycle and signals
- Disk and inode usage (`df`, `du`, `find -xdev -type f -size +100M`)
- Basic networking checks (DNS, ping, routes, interfaces)

Where this is used:
- Cloud VM troubleshooting (AWS EC2, GCP Compute Engine, Azure VMs)
- On-call incident response and production debugging
- Prepping container images and debugging containers (e.g., `docker exec` + these commands)

---

If you want, I can:
- Commit this cheat sheet to `2026/day-03/linux-commands-cheatsheet.md` and push it.
- Expand this into printable PDF or a one-page quick-reference card.

Happy practicing — hands-on use of these commands during incidents is the fastest way to learn.
