# Thread Synchronization using Conditional Variables & Semaphores

## Why Do We Need These?

When multiple threads run concurrently, they often need access to shared resources.

Problems that arise:

* Race Conditions
* Busy Waiting
* Resource Contention
* Inefficient CPU Usage

Operating Systems provide synchronization primitives such as:

1. Mutex/Locks
2. Condition Variables
3. Semaphores

---

# 1. Busy Waiting

## Problem

Suppose Thread T1 holds a lock.

Thread T2 wants the same lock.

A naive implementation might continuously check:

```python
while lock_is_busy:
    pass
```

This is called **Busy Waiting (Spinning)**.

### Drawbacks

* Wastes CPU cycles
* Poor scalability
* Reduces overall system performance

Instead of continuously checking, we want the thread to:

1. Sleep
2. Release CPU
3. Wake up only when needed

This is where **Condition Variables** come in.

---

# 2. Condition Variables

## Definition

A Condition Variable allows a thread to:

* Sleep while waiting for a condition
* Avoid busy waiting
* Wake up when another thread signals that the condition may be true

---

## Real-Life Example

Imagine:

* Consumer wants data
* Producer generates data

Consumer should sleep until data becomes available.

---

# Python Example: Condition Variable

```python
import threading
import time

# Shared state
data_ready = False

# Condition variable
condition = threading.Condition()


def consumer():
    global data_ready

    # Step 1:
    # Acquire lock associated with condition variable
    with condition:

        # Step 2:
        # If data is not ready,
        # consumer should sleep
        while not data_ready:

            print("Consumer: Waiting for data...")

            # Step 3:
            # wait() does TWO things atomically:
            #
            # 1. Releases the lock
            # 2. Blocks the thread
            #
            # Thread moves to Condition Wait Queue
            condition.wait()

        # Step 7:
        # Thread wakes up and automatically
        # re-acquires lock before proceeding
        print("Consumer: Data received, processing...")


def producer():
    global data_ready

    # Simulate work
    time.sleep(2)

    # Step 4:
    # Acquire lock
    with condition:

        print("Producer: Producing data")

        # Step 5:
        # Update shared state
        data_ready = True

        # Step 6:
        # Wake one waiting thread
        condition.notify()

        print("Producer: Sent signal")


t1 = threading.Thread(target=consumer)
t2 = threading.Thread(target=producer)

t1.start()
t2.start()

t1.join()
t2.join()
```

---

# Step-by-Step Execution

## Initial State

```text
data_ready = False
```

---

## Consumer Starts

```python
with condition:
```

Lock acquired.

---

Consumer checks:

```python
while not data_ready:
```

Condition is false.

---

Consumer executes:

```python
condition.wait()
```

OS performs:

```text
1. Release lock
2. Move consumer to wait queue
3. Block consumer
4. Give CPU to another thread
```

State:

```text
Lock = Free
Consumer = Sleeping
```

---

## Producer Starts

Producer acquires lock.

```python
data_ready = True
```

Condition becomes true.

---

Producer executes:

```python
condition.notify()
```

OS performs:

```text
Move consumer from waiting queue
to ready queue
```

Important:

```text
Consumer does NOT run immediately.
```

It only becomes ready.

---

Producer releases lock.

---

## Consumer Wakes Up

Before returning from wait():

```text
Consumer automatically re-acquires lock.
```

Execution continues after:

```python
condition.wait()
```

Consumer checks condition again:

```python
while not data_ready:
```

Now:

```text
data_ready = True
```

Loop ends.

Consumer processes data.

---

# Why Use While Instead of If?

Wrong:

```python
if not data_ready:
    condition.wait()
```

Correct:

```python
while not data_ready:
    condition.wait()
```

Reason:

## Spurious Wakeups

A thread may wake up unexpectedly.

Or:

```text
Thread wakes up
Another thread changes state
Before it gets CPU
```

Therefore:

```python
while(condition_false):
    wait()
```

is always the safe pattern.

---

# Interview Definition

> A Condition Variable is a synchronization primitive that allows threads to sleep until a particular condition becomes true, avoiding busy waiting.

---

# Producer-Consumer Pattern

Condition Variables are commonly used in:

* Producer Consumer Problem
* Task Queues
* Job Scheduling Systems
* Thread Pools

---

# 3. Semaphores

## Definition

A Semaphore is a synchronization primitive that maintains an integer counter.

It controls access to a limited number of resources.

---

# Operations

## wait()

Also called:

```text
down()
P()
acquire()
```

Operation:

```text
Semaphore--
```

If resource unavailable:

```text
Thread blocks
```

---

## signal()

Also called:

```text
up()
V()
release()
```

Operation:

```text
Semaphore++
```

If threads are waiting:

```text
Wake one blocked thread
```

---

# Types of Semaphores

## Binary Semaphore

Values:

```text
0 or 1
```

Acts similarly to a Mutex.

---

## Counting Semaphore

Values:

```text
0,1,2,3,4...
```

Used when multiple resource instances exist.

Examples:

* 3 Printers
* 5 Database Connections
* 10 Network Ports

---

# Python Example: Counting Semaphore

Suppose:

```text
3 Printers
10 Users
```

Only 3 users can print simultaneously.

```python
import threading
import time

# Three printers available
printer_sem = threading.Semaphore(3)


def user(user_id):

    print(f"User {user_id} wants printer")

    # Step 1:
    # acquire() = wait()/down()/P()
    printer_sem.acquire()

    print(f"User {user_id} got printer")

    # Critical Section
    time.sleep(3)

    print(f"User {user_id} finished printing")

    # Step 2:
    # release() = signal()/up()/V()
    printer_sem.release()


threads = []

for i in range(10):
    t = threading.Thread(target=user, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

---

# Internal Working

Initial value:

```text
Semaphore = 3
```

---

## User 1

```python
acquire()
```

Result:

```text
Semaphore = 2
```

Allowed.

---

## User 2

```text
Semaphore = 1
```

Allowed.

---

## User 3

```text
Semaphore = 0
```

Allowed.

---

## User 4

```python
acquire()
```

No printer available.

OS performs:

```text
Move User4 to waiting queue
Block User4
```

---

## User 5

Blocked.

---

## User 6

Blocked.

---

# Resource Release

User1 finishes.

```python
release()
```

OS performs:

```text
Semaphore++
Wake one waiting thread
```

Example:

```text
User4 wakes up
Gets printer
```

---

# Visualization

```text
Printers = 3

Printer 1 -> User1
Printer 2 -> User2
Printer 3 -> User3

Waiting Queue:
User4
User5
User6
```

When User1 leaves:

```text
Printer 1 -> User4
```

---

# Binary Semaphore Example

```python
import threading

sem = threading.Semaphore(1)

def task():

    sem.acquire()

    # Critical Section
    print("Inside critical section")

    sem.release()
```

Only one thread enters at a time.

---

# Mutex vs Binary Semaphore

| Feature                  | Mutex             | Binary Semaphore |
| ------------------------ | ----------------- | ---------------- |
| Values                   | Locked / Unlocked | 0 / 1            |
| Ownership                | Yes               | No               |
| Same Thread Must Unlock? | Yes               | No               |
| Primary Purpose          | Mutual Exclusion  | Synchronization  |

---

# Condition Variable vs Semaphore

| Feature            | Condition Variable    | Semaphore              |
| ------------------ | --------------------- | ---------------------- |
| Purpose            | Wait for state change | Control resource count |
| Counter            | No                    | Yes                    |
| Sleep/Wakeup       | Yes                   | Yes                    |
| Tracks Resources   | No                    | Yes                    |
| Producer-Consumer  | Common                | Common                 |
| Multiple Resources | Not Suitable          | Ideal                  |

---

# FAANG Interview Questions

## Q1. Why not use busy waiting?

Answer:

```text
Busy waiting continuously consumes CPU cycles.
Condition variables block the thread and release CPU,
making synchronization more efficient.
```

---

## Q2. Why does wait() release the lock?

Answer:

```text
Other threads must acquire the lock to update
the shared state and eventually signal the waiting thread.
```

---

## Q3. Why always use while around wait()?

Answer:

```text
To protect against:

1. Spurious wakeups
2. State changes by other threads

Always re-check the condition after waking up.
```

---

## Q4. Difference Between Mutex and Semaphore?

Answer:

```text
Mutex protects one critical section and has ownership.

Semaphore manages resource counts and has no ownership.
```

---

## Q5. When Would You Use a Semaphore?

Examples:

```text
3 printers
5 database connections
10 worker slots
Limited API requests
```

---

## Q6. When Would You Use a Condition Variable?

Examples:

```text
Queue becomes non-empty
Task completed
File downloaded
Data available
```

---

# Quick Revision

## Condition Variable

```text
Condition Variable

Thread sleeps
↓
Condition becomes true
↓
signal()/notify()
↓
Thread wakes up
↓
Re-check condition
↓
Continue execution
```

---

## Semaphore

```text
Semaphore = Counter

acquire()
↓
Counter--

If resource unavailable:
Thread blocks

release()
↓
Counter++

Wake waiting thread
```

---

- How They Execute (The Flow)Process synchronization uses a tool called a Semaphore. A semaphore is basically an integer variable that acts like a key to your - CS.
- Check & Lock (wait): A process asks to execute code. It runs the wait() operation.The semaphore's value is lowered by 1.If the semaphore value drops below 0, the resource is busy. The process is blocked and placed into a waiting queue.
- Execution (CS): If the wait operation passes, the process enters the Critical Section and safely executes its tasks.
- Unlock (signal): When the process finishes its work in the CS, it runs the signal() operation.The semaphore's value is increased by 1.If there are other waiting processes in the queue, one of them is woken up and gets access to the CS.

- Basic Code ExampleHere is the general structure of how these operations guard the CS:
```
// Shared Semaphore (initialized to 1)
Semaphore mutex = 1; 

// Process P
do {
    // 1. Wait (P-Operation): Request entry
    wait(mutex); 
    
    // --- CRITICAL SECTION --- 
    // Only one process can be in here at a time
    
    // 2. Signal (V-Operation): Leave and unlock
    signal(mutex); 
    
    // --- REMAINDER SECTION ---
} while (true);
```

# One-Line Interview Definitions

Condition Variable:

> A sleep-and-wakeup synchronization mechanism used when a thread must wait for a condition to become true.

Semaphore:

> A counter-based synchronization primitive used to manage access to a finite number of resources.

Memory Trick:

Mutex → One key, one room

Binary Semaphore → One token

Counting Semaphore → Multiple tokens

Condition Variable → Sleep until notified
