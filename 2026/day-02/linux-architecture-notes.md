# Day 02 Notes: Linux Architecture, Processes, and systemd

## 1. Linux at a Glance

- Linux is the operating system core used in servers, cloud VMs, containers, and many embedded devices.
- It has three important parts:
  - Kernel: the core manager of CPU, memory, files, and devices.
  - User space: where our programs run, like `bash`, `nginx`, and `python`.
  - Init/systemd: the first user-space process that starts and supervises services.

### Why this matters

- The kernel decides how resources are shared.
- User space is where your applications live.
- systemd makes sure services start, stop, and restart properly.

### Simple analogy

- Kernel = the traffic controller.
- User space = the cars on the road.
- systemd = the dispatcher that sends cars out on schedule and checks if they are still running.

### Quick summary

- Linux is not just one thing; it is a layered system.
- The kernel controls hardware access.
- systemd helps Linux boot and manage services.

### Key takeaways

- Most troubleshooting starts by asking: is the problem in the kernel, process, or service layer?
- systemd is a major part of modern Linux administration.

---

## 2. How Processes Are Created and Managed

- A process is a running program.
- The Linux kernel gives each process a Process ID, or PID.
- Most processes are created using two steps:
  - `fork()`: creates a new child process by copying the parent.
  - `exec()`: replaces the child process with a new program.

### What happens internally

- The shell receives your command.
- The shell asks the kernel to create a child process.
- The child process loads the command you typed.
- The kernel tracks memory, open files, permissions, and CPU time for each process.

### Why this matters

- This explains why running a command creates a new process.
- It also helps you understand orphan processes, zombie processes, and service restarts.

### Quick summary

- `fork()` makes a copy.
- `exec()` loads the actual program.
- The kernel tracks every process in its process table.

### Key takeaways

- A shell is not the command itself; it launches the command.
- Process management is one of the most important Linux skills for DevOps.

---

## 3. Common Process States

- Running (`R`): the process is using CPU time right now.
- Sleeping (`S`): the process is waiting for input, disk, network, or another event.
- Uninterruptible sleep (`D`): the process is waiting in kernel space, often for I/O.
- Stopped (`T`): the process was paused, often by a signal or debugger.
- Zombie (`Z`): the process finished, but the parent has not collected its exit status yet.

### Beginner mistake to avoid

- Do not assume every sleeping process is broken.
- Do not try to "kill" a zombie as if it were still running. The real fix is usually the parent process.

### Quick summary

- Process state tells you what the process is doing.
- Sleeping is normal.
- Zombie means the process has exited but still has a record in the process table.

### Key takeaways

- Read process state before taking action.
- `ps` and `top` help you spot unusual states quickly.

---

## 4. What systemd Does and Why It Matters

- systemd is the first user-space process started by the Linux kernel.
- It is usually PID 1.
- It starts services, monitors them, and restarts them if needed.

### Real-world purpose

- Start services at boot.
- Restart failed services.
- Manage logs and service dependencies.
- Handle sockets, timers, mounts, and targets.

### How it works internally

- systemd reads unit files.
- A unit file describes what should run and when.
- systemd places services into control groups, also called cgroups.
- cgroups help systemd track resource usage and child processes.

### Example service unit

```ini
[Unit]
Description=Demo application service

[Service]
ExecStart=/usr/local/bin/demo-app   # Command systemd runs to start the app
Restart=on-failure                 # Restart only when the app crashes
User=appuser                       # Run the app as a non-root user

[Install]
WantedBy=multi-user.target         # Start this service during normal boot
```

### Why each part matters

- `[Unit]`: service metadata and dependencies.
- `[Service]`: how to start and manage the program.
- `[Install]`: when the service should start automatically.

### Quick summary

- systemd is the service manager used by most modern Linux distributions.
- It is responsible for boot-time service orchestration and recovery.

### Key takeaways

- When a service fails, systemd is usually the first place to check.
- `systemctl` and `journalctl` are the most useful systemd tools.

---

## 5. Five Daily Commands

```bash
ps aux            # Show all running processes, who owns them, and basic resource usage
top               # View live CPU and memory usage; press q to quit
systemctl status sshd   # Check whether the SSH service is active and healthy
journalctl -u sshd -n 50 --no-pager   # Read the last 50 logs from the SSH service
kill <PID>        # Politely ask a process to stop using SIGTERM
```

### What to expect

- `ps aux` gives a snapshot.
- `top` updates continuously.
- `systemctl status` shows whether a service is running, failed, or disabled.
- `journalctl` shows logs from systemd and services.
- `kill` sends a signal; by default it asks the process to exit cleanly.

### Common beginner mistakes

- Using `kill -9` too early. This force-stops a process and skips cleanup.
- Forgetting that `systemctl` controls services, not ordinary shell commands.
- Reading only the last error line instead of checking the full log context.

### Quick summary

- These five commands cover most beginner-level Linux troubleshooting.
- They help with process visibility, service health, and logs.

### Key takeaways

- Learn these commands first before moving to advanced Linux tools.
- Logs and process state are your fastest debugging path.

---

## 6. Troubleshooting Mindset

- If an app is slow, check CPU and memory with `top`.
- If a service does not start, check `systemctl status` first.
- If the error is unclear, read the logs with `journalctl`.
- If a process is stuck in `D` state, the problem is often I/O or storage.

### Quick summary

- Start with process state, then service status, then logs.

### Key takeaways

- Good Linux troubleshooting follows a simple order.
- Do not jump straight to force-killing processes.

---

## 7. Interview Revision

### Questions to practice

- What is the difference between the kernel and user space?
- How are processes created in Linux?
- What does PID 1 do?
- What is a zombie process?
- Why is systemd important in modern Linux systems?

### Concepts to revise

- Kernel vs user space
- `fork()` and `exec()`
- Process states
- systemd units
- `systemctl` and `journalctl`

### Where this is used in real companies

- Production servers that run APIs, databases, and background jobs.
- Cloud virtual machines on AWS, Azure, or GCP.
- Kubernetes nodes, which are Linux machines managed at scale.
- Incident response, where fast service and log checks save time.

### Final quick summary

- Linux is built around the kernel, user space, and a service manager.
- Processes are created by `fork()` and `exec()`.
- systemd keeps services healthy and restartable.

### Final key takeaways

- If you understand processes and systemd, Linux troubleshooting becomes much easier.
- These basics are used every day in DevOps and Cloud jobs.