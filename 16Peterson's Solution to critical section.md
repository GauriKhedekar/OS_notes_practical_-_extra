# Peterson's Solution – Interview Notes

## Why can't a Flag Variable Alone Solve the Critical Section Problem?

Suppose we use only a flag array:

```c
bool flag[2] = {false, false};

// ------------------- Process P0 -------------------

flag[0] = true;      // P0 says: "I want to enter Critical Section."

// Wait while P1 also wants to enter.
// If flag[1] == true, P0 keeps spinning here.
while(flag[1]);

// Critical Section
// P0 enters only if P1 is not interested.
printf("P0 in Critical Section");

// P0 has finished using the shared resource.
flag[0] = false;


// ------------------- Process P1 -------------------

flag[1] = true;      // P1 says: "I want to enter Critical Section."

// Wait while P0 also wants to enter.
// If flag[0] == true, P1 keeps spinning here.
while(flag[0]);

// Critical Section
printf("P1 in Critical Section");

// P1 has finished using the shared resource.
flag[1] = false;
```

### Problem: Deadlock

If both processes execute: No process will enter the Critical Section as bith processes will spin in while loop.

```text
P0: flag[0] = true
P1: flag[1] = true
```

Now:

```text
P0 sees flag[1] = true  -> waits
P1 sees flag[0] = true  -> waits
```

Both keep waiting forever.

### Why Peterson Adds `turn`

The `flag` variable only tells **"I want to enter the Critical Section."**

It does **not decide who should enter first** when both want to enter simultaneously.

The `turn` variable acts as a tie-breaker:

```c
flag[i] = true;
turn = j;
while(flag[j] && turn == j);
```

Thus only one process waits and the other enters the Critical Section.

---

# Peterson's Solution (Interview Notes)

## Definition

Peterson's Solution is a **classical software-based solution** for the Critical Section Problem.

- Works for **2 processes only**
- Uses **busy waiting**
- May not work correctly on modern multicore architectures
- Important for understanding process synchronization concepts

---

## Shared Variables

### 1. flag[]

```c
bool flag[2];
```

Purpose:

- `flag[i] = true` → Process Pi wants to enter Critical Section.
- `flag[i] = false` → Process Pi is not interested.

### 2. turn

```c
int turn;
```

Purpose:

- Indicates whose turn it is to enter the Critical Section.
- Used to break ties when both processes want entry simultaneously.

---

## Peterson's Algorithm

For Process Pi:

```c
do{
    flag[i] = true;
    turn = j;

    while(flag[j] && turn == j);

    // Critical Section

    flag[i] = false;

    // Remainder Section
}while(true);
```

For Process Pj:

```c
do{
    flag[j] = true;
    turn = i;

    while(flag[i] && turn == i);

    // Critical Section

    flag[j] = false;

    // Remainder Section

}while(true);
```

---

## Core Idea

When a process wants to enter:

1. Set its flag to true.
2. Give priority to the other process using `turn`.
3. Wait only if:
   - Other process also wants to enter, and
   - It is the other process's turn.

This is why Peterson's Solution is often described as a **"humble algorithm."**

---

## Simultaneous Request Scenario

Both P0 and P1 want to enter:

```text
flag[0] = true
flag[1] = true
```

Then:

```text
P0 -> turn = 1
P1 -> turn = 0
```

The last write to `turn` wins.

Result:

- One process enters Critical Section.
- Other process stays in the while loop.
- Mutual exclusion is maintained.

---

## Why Peterson's Solution Works

### Mutual Exclusion

✔ At most one process can be in Critical Section at a time.

### Progress

✔ If Critical Section is free, a decision is made without indefinite postponement.

### Bounded Waiting

✔ Every waiting process eventually gets a chance.

---

## Limitations

- Restricted to only **two processes**
- Uses **busy waiting (spinlock)**
- Not reliable on modern CPUs due to memory reordering/cache effects

---

## Interview One-Liners

- Peterson's Solution is a software solution for the Critical Section Problem.
- It uses two shared variables: `flag[]` and `turn`.
- `flag` indicates interest; `turn` resolves conflicts.
- Satisfies Mutual Exclusion, Progress, and Bounded Waiting.
- Works only for two processes.
- Uses busy waiting.
- Mainly studied for conceptual understanding of synchronization.
