# Multi-Container Runtime 

---

# 1. Team Information

Chetana V Iyer - SRN: PES1UG24C130

Gagan B Sasalatti - SRN: PES1UG24CS164

---

# 2. Build, Load, and Run Instructions

## 🔧 Build

```
make
```

## 🔌 Load Kernel Module

```
sudo insmod monitor.ko
```

## ✅ Verify Device

```
ls -l /dev/container_monitor
```

## 🚀 Start Supervisor

```
sudo ./engine supervisor ./rootfs-base
```

## 📁 Create Writable Root Filesystems

```
cp -a ./rootfs-base ./rootfs-alpha
cp -a ./rootfs-base ./rootfs-beta
```

## ▶️ Start Containers

```
sudo ./engine start alpha ./rootfs-alpha /bin/sh --soft-mib 48 --hard-mib 80
sudo ./engine start beta ./rootfs-beta /bin/sh --soft-mib 64 --hard-mib 96
```

## 📊 List Containers

```
sudo ./engine ps
```

## 📜 View Logs

```
sudo ./engine logs alpha
```

## 🧪 Run Workloads

```
cp cpu_hog ./rootfs-alpha/
cp io_pulse ./rootfs-beta/
cp memory_hog ./rootfs-alpha/

sudo ./engine start cpu ./rootfs-alpha ./cpu_hog
sudo ./engine start io ./rootfs-beta ./io_pulse
```

## 🛑 Stop Containers

```
sudo ./engine stop alpha
sudo ./engine stop beta
```

## 📟 Inspect Kernel Logs

```
dmesg | tail
```

## ❌ Unload Module

```
sudo rmmod monitor
```

---

# 3. Demo with Screenshots

---

## 1. Multi-container supervision

👉 Multiple containers running under a single supervisor

📸 **Paste Screenshot Here**

<img width="734" height="101" alt="osjone" src="https://github.com/user-attachments/assets/db546ac2-7ec8-47ee-9481-5741ff7386a2" />

---

## 2. Metadata tracking

👉 Output of `engine ps`

📸 **Paste Screenshot Here**

 <img width="617" height="97" alt="osj2" src="https://github.com/user-attachments/assets/d179ffa0-30a0-41f3-96e3-11b572c2f29c" />
---

## 3. Logging system

👉 Output from `echo one`, `echo two`

📸 **Paste Screenshot Here**

<img width="938" height="163" alt="osj3" src="https://github.com/user-attachments/assets/b13dcbf9-b560-412c-b6da-1b231689bdc7" />

---

## 4. CLI and IPC

👉 Command execution and response

<img width="977" height="66" alt="osj4" src="https://github.com/user-attachments/assets/b27a60dc-d327-4af3-b6c7-325678aa5eac" />

---

## 5. Soft-limit + Hard-limit logs

👉 Kernel showing warning + kill

<img width="719" height="143" alt="osj5" src="https://github.com/user-attachments/assets/0426e2d4-d3bd-49af-baff-14f65c8e7efb" />


## 6. Hard-limit enforcement (ps output)

👉 `hard_limit_killed` state

<img width="674" height="343" alt="osj6" src="https://github.com/user-attachments/assets/68dfbe4e-d7e6-409e-9960-ffc4f69f3b4a" />

---

## 7. Scheduling experiment

👉 cpu_hog vs io_pulse

<img width="1096" height="514" alt="osj7" src="https://github.com/user-attachments/assets/2138668e-1d41-434f-9e35-51a9431fcde7" />

---

## 8. Clean teardown

👉 No zombie processes

<img width="762" height="75" alt="osj8" src="https://github.com/user-attachments/assets/5fc97625-3fd3-428d-8716-5aeb8d54c633" />

---

# 4. Engineering Analysis

---

## 1. Isolation Mechanisms

We use Linux namespaces:

* CLONE_NEWPID → separate process tree
* CLONE_NEWUTS → separate hostname
* CLONE_NEWNS → separate filesystem

`chroot()` ensures filesystem isolation.

Containers share the same kernel but operate in isolated environments.

---

## 2. Supervisor and Process Lifecycle

A long-running supervisor:

* creates containers using `clone()`
* tracks PID, state, limits
* prevents zombies using `waitpid(..., WNOHANG)`

It also distinguishes:

* normal exit
* manual stop
* hard-limit kill

---

## 3. IPC, Threads, and Logging

Two IPC systems:

* Pipes → logging
* UNIX sockets → control

Logging uses:

* Producer (container output)
* Consumer (writes to file)

Protected using mutex + condition variables.

---

## 4. Memory Management and Enforcement

Memory is monitored using **RSS (Resident Set Size)**.

* Soft limit → warning log
* Hard limit → SIGKILL

Kernel-space monitoring ensures:

* accuracy
* fast enforcement

---

## 5. Scheduling Behavior

We tested:

* cpu_hog → CPU-bound
* io_pulse → I/O-bound

Observed:

* cpu_hog → ~100% CPU
* io_pulse → ~0–1% CPU

This demonstrates Linux CFS scheduler behavior.

---

## 6. Design Decisions and Tradeoffs

### Namespace Isolation

* Used: clone + chroot
* Tradeoff: weaker than pivot_root
* Reason: simpler

### Supervisor

* Centralized control
* Tradeoff: single failure point
* Reason: easier management

### IPC

* Pipes + sockets
* Tradeoff: complexity
* Reason: modular design

### Kernel Monitor

* Kernel-space enforcement
* Tradeoff: complexity
* Reason: accurate memory tracking

---

## 7. Scheduler Experiment Results

### Setup

* cpu_hog (CPU-bound)
* io_pulse (I/O-bound)

### Results

* cpu_hog → ~100% CPU
* io_pulse → ~0–1% CPU

### Conclusion

* Fairness → CPU distributed
* Responsiveness → I/O prioritized
* Efficiency → CPU fully utilized

---

# FINAL SUMMARY

We implemented a lightweight container runtime using Linux namespaces and chroot for isolation, a supervisor for lifecycle management, pipes and sockets for IPC, and a kernel module for enforcing memory limits using RSS monitoring.

---
