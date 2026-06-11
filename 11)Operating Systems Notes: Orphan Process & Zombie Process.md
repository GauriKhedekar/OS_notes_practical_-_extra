# Operating Systems Notes: Orphan Process & Zombie Process

## Use WSL

These commands are Linux-based and were tested in WSL.

---

# 1. Orphan Process

## Definition

An **Orphan Process** is a child process whose parent terminates before the child process.

The OS assigns the child to **init/systemd (PID = 1)**.

### Interview One-Liner

> Parent dies first, child survives, PPID becomes 1.

---

## Practical Commands

Create file:

```bash
nano orphan.sh
```

Give permission:

```bash
chmod +x orphan.sh
```

Run:

```bash
./orphan.sh
```

Verify:

```bash
ps -f
```

```bash
ps -ef | grep sleep
```

```bash
ps -o pid,ppid,cmd | grep sleep
```

Expected:

```text
sleep 200
PPID = 1
```

Kill process:

```bash
kill PID
```

---

# 2. Zombie Process

## Definition

A **Zombie Process** is a terminated child process whose exit status has not been collected by the parent using `wait()`.

### Interview One-Liner

> Child dies first, parent doesn't call wait(), state becomes Z.

---

## Practical Commands

Create file:

```bash
nano zombie.sh
```

Give permission:

```bash
chmod +x zombie.sh
```

Run:

```bash
./zombie.sh
```

Check processes:

```bash
ps -al
```

Check zombie state:

```bash
ps -el | grep Z
```

Check defunct processes:

```bash
ps aux | grep defunct
```

---

# Important Commands

Show current processes:

```bash
ps -f
```

Show all processes:

```bash
ps -ef
```

Show process details and state:

```bash
ps -al
```

Show PID and PPID:

```bash
ps -o pid,ppid,cmd
```

Show zombies:

```bash
ps -el | grep Z
```

Show defunct processes:

```bash
ps aux | grep defunct
```

Kill process:

```bash
kill PID
```

Force kill:

```bash
kill -9 PID
```

---

# Orphan vs Zombie

| Orphan            | Zombie                               |
| ----------------- | ------------------------------------ |
| Parent dies first | Child dies first                     |
| Child running     | Child terminated                     |
| PPID becomes 1    | State becomes Z                      |
| Adopted by PID 1  | Waiting for parent to collect status |

---

# Quick Revision

## Orphan

```text
Parent dies first
Child survives
PPID = 1
```

Verify:

```bash
ps -o pid,ppid,cmd
```

---

## Zombie

```text
Child dies first
Parent doesn't call wait()
State = Z
```

Verify:

```bash
ps -el | grep Z
```

```bash
ps aux | grep defunct
```

---

# Most Asked Interview Questions

### What is an Orphan Process?

A child process whose parent terminates before it. It is adopted by PID 1.

### What is a Zombie Process?

A terminated child process whose exit status has not been collected by the parent.

### PPID of Orphan Process?

```text
1
```

### How to identify a Zombie?

```bash
ps -el | grep Z
```

### How to prevent Zombies?

Use:

```c
wait();
waitpid();
```
