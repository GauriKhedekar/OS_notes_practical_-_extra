# Operating Systems Notes: Orphan Process & Zombie Process (Interview + Practical)



---

# Use WSL for better execution as these are Linux based commands:-

# 1. Orphan Process

## Definition

An orphan process is a process whose parent process terminates before the child process.

When this happens, the OS assigns the orphan process to the `init/systemd` process (`PID = 1`).

---

## Normal Process Flow

```text
Terminal
   |
   V
orphan.sh
   |
   V
sleep 200
```

### Process Table

| PID  | PPID | Process   |
| ---- | ---- | --------- |
| 9161 | 1    | Terminal  |
| 9185 | 9161 | orphan.sh |
| 9186 | 9185 | sleep 200 |

Here:

* Terminal is parent of `orphan.sh`
* `orphan.sh` is parent of `sleep 200`

No orphan process exists.

---

## Creating an Orphan Process

### Step 1: Create Script

```bash
nano orphan.sh
```

Write:

```bash
#!/bin/bash

sleep 200 &
```

Save and exit.(ctrl + o and ctrl + x)

---

### Step 2: Give Execute Permission

```bash
chmod +x orphan.sh
```

---

### Step 3: Run Script

```bash
./orphan.sh
```

The `&` runs `sleep` in the background.

---

## What Happens Internally?

### Before orphan.sh Exits

```text
Terminal
   |
   V
orphan.sh
   |
   V
sleep 200
```

### After orphan.sh Exits

```text
init/systemd (PID 1)
        |
    sleep 200
```

The OS changes the PPID.

---

## Verify

Run:

```bash
ps -ef | grep sleep 
```

or

```bash
ps -o pid,ppid,cmd | grep sleep

# use "ps -f" in git bash
```

Example Output:

```text
9228  1  sleep 200
```

Observe:

```text
PPID = 1
```

This proves it became an orphan process.

---

## Diagram

### Before

```text
Terminal
   |
orphan.sh
   |
sleep 200
```

### After

```text
init/systemd (PID 1)
        |
    sleep 200
```

---

## Interview Questions

### Q1: What is an Orphan Process?

A child process whose parent terminates before the child.

### Q2: What happens to an Orphan Process?

It is adopted by `init/systemd` (`PID 1`).

### Q3: Does an orphan process cause problems?

No.

The operating system handles it automatically.

---

# 2. Zombie Process

## Definition

A Zombie Process is a process that has completed execution but still has an entry in the process table.

Also called:

* Defunct Process
* Zombie Process

---

## Why Does a Zombie Occur?

When a child exits:

```c
exit();
```

its memory and resources are released.

However, its process table entry remains until the parent collects its exit status using:

```c
wait();
```

or

```c
waitpid();
```

---

## Normal Execution

```text
Parent
   |
 Child
```

Child exits.

Parent calls:

```c
wait();
```

Parent reads exit status.

OS removes child entry.

No zombie is created.

---

## Zombie Creation Scenario

```text
Parent
   |
 Child
```

Child exits immediately.

Parent does NOT call `wait()`.

Result:

```text
Parent (running)

Zombie Child
```

Zombie remains in the process table.

---

## Diagram

### Step 1

```text
P1
 |
P2
```

### Step 2

```text
P2 exits

P1

P2 (Zombie)
```

### Step 3

Parent calls:

```c
wait();
```

### Step 4

Zombie removed.

```text
P1
```

---

## Why Is a Zombie Dangerous?

A zombie occupies:

* PID
* Process Table Entry

Although memory is released.

If many zombies accumulate:

```text
Process Table Full
```

Then:

* New processes cannot be created
* System may become unstable

---

## Practical Creation of a Zombie Process

> **Note:** The shell script method below may not reliably create zombies on all Linux distributions because Bash often reaps child processes automatically. For guaranteed zombie creation, use the C program shown later.

### Script Method

Create:

```bash
nano zombie.sh
```

Write:

```bash
#!/bin/bash

for i in {1..20}
do
    sleep 1 &
done

exec sleep 100
```

### Explanation

```bash
sleep 1 &
```

Creates many child processes:

```text
sleep 1 &
sleep 1 &
sleep 1 &
...
```

All run independently.

```bash
exec sleep 100
```

Replaces the bash process.

Depending on the shell and system behavior, some child processes may briefly appear as zombies.

---

### Give Permission

```bash
chmod +x zombie.sh
```

### Run

```bash
./zombie.sh
```

### Check Zombies

```bash
ps -el | grep Z
```

or

```bash
ps aux | grep defunct
```

Example Output:

```text
Z  sleep
Z  sleep
Z  sleep
```

The `Z` state means:

```text
Zombie
```

---

## Guaranteed Zombie Demo (C Program)

Create:

```bash
nano zombie.c
```

Paste:

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main() {
    pid_t pid = fork();

    if (pid == 0) {
        printf("Child exiting...\n");
        return 0;
    }

    printf("Parent PID = %d\n", getpid());

    sleep(100);

    return 0;
}
```

Compile:

```bash
gcc zombie.c -o zombie
```

Run:

```bash
./zombie
```

Open another terminal:

```bash
ps -el | grep Z
```

or

```bash
ps aux | grep defunct
```

Expected:

```text
Z    zombie <defunct>
```

### Why This Works

1. Child exits immediately.
2. Parent stays alive for 100 seconds.
3. Parent never calls `wait()`.
4. Kernel keeps the child's process table entry.
5. Process state becomes `Z` (Zombie).

---

## Understanding the Flow

### Normal Process

```text
Parent
  |
  +---- Child Running
```

### Child Finishes

```text
Parent
  |
  +---- Child Exits
```

### Parent Doesn't Call wait()

```text
Parent (alive)
  |
  +---- Zombie
```

### Parent Exits

```text
init/systemd
     |
     +---- adopts zombie
     |
     +---- wait()
```

Zombie disappears.

---

# Important Commands

## Show All Processes

```bash
ps -ef
```

## Show PID and PPID

```bash
ps -o pid,ppid,cmd
```

## Show Zombie Processes

```bash
ps -el | grep Z
```

## Show Defunct Processes

```bash
ps aux | grep defunct
```

## Kill Process

```bash
kill PID
```

Example:

```bash
kill 9228
```

## Force Kill

```bash
kill -9 PID
```

---

# Difference Between Orphan and Zombie

| Feature              | Orphan     | Zombie      |
| -------------------- | ---------- | ----------- |
| Parent Alive?        | No         | Yes         |
| Child Alive?         | Yes        | No          |
| Execution Running?   | Yes        | No          |
| Resources Occupied?  | Yes        | Almost none |
| Process Table Entry? | Yes        | Yes         |
| Adopted by PID 1?    | Yes        | No          |
| Dangerous?           | Usually No | Yes         |

---

# Quick Revision (1 Minute)

## Orphan Process

* Parent dies first
* Child survives
* PPID becomes `1`

Example:

```bash
sleep 200 &
```

inside a script.

---

## Zombie Process

* Child dies first
* Parent doesn't call `wait()`

Result:

```text
Zombie entry remains
```

Check:

```bash
ps -el | grep Z
```

Remove:

```c
wait();
```

---

# Interview One-Liners

### Orphan Process

> A process whose parent terminates before it. It is adopted by `init/systemd (PID 1)`.

### Zombie Process

> A terminated child process whose exit status has not yet been collected by the parent using `wait()`.

### How to Identify a Zombie?

```bash
ps -el | grep Z
```

### How to Prevent Zombies?

```c
wait();
waitpid();
```

### PPID of an Orphan Process?

```text
1
```

---

## Most Important Interview Summary

### Orphan

```text
Parent dies first
Child survives
PPID becomes 1
```

### Zombie

```text
Child dies first
Parent doesn't call wait()
Zombie entry remains
State = Z
```

These two topics (Orphan vs Zombie) are among the most frequently asked Operating System interview questions, especially when discussing `fork()`, `wait()`, process creation, and process management.
