# Dining Philosophers Problem :-

## Problem

-   5 philosophers, 5 forks.
-   Each philosopher alternates between **Thinking** and **Eating**.
-   To eat, a philosopher needs **both left and right forks**.
-   If everyone picks the **left fork first**, everyone waits forever
    for the right fork → **Deadlock (Circular Wait).**

------------------------------------------------------------------------

## Basic Semaphore Solution (Deadlock Possible)

``` c
semaphore fork[5] = {1,1,1,1,1};

philosopher(i){
    while(true){

        think();

        wait(fork[i]);             // Pick left fork
        wait(fork[(i+1)%5]);       // Pick right fork
        // ⚠ Deadlock if everyone already holds left fork.

        eat();

        signal(fork[i]);           // Release left fork
        signal(fork[(i+1)%5]);     // Release right fork
    }
}
```

### Remember

-   `wait()` → Acquire resource (decrement semaphore).
-   `signal()` → Release resource (increment semaphore).

------------------------------------------------------------------------

# Why Deadlock Happens?

All philosophers execute:

1.  Pick Left ✔
2.  Wait for Right ❌

Everyone owns one fork and waits forever.

Conditions: - Mutual Exclusion - Hold and Wait - No Preemption -
Circular Wait ← culprit here

------------------------------------------------------------------------

# Deadlock Prevention

## 1. Allow only 4 philosophers

``` c
semaphore room = 4;

philosopher(i){

    wait(room);               // Only 4 philosophers allowed

    wait(fork[i]);
    wait(fork[(i+1)%5]);

    eat();

    signal(fork[i]);
    signal(fork[(i+1)%5]);

    signal(room);
}
```

**Idea:** One philosopher stays out, so someone can always obtain both
forks.

------------------------------------------------------------------------

## 2. Pick both forks atomically

``` c
lock(mutex);

// Check BOTH forks together.
// If both available -> acquire both.
// Else wait and acquire none.

unlock(mutex);
```

**Idea:** Never allow a philosopher to hold only one fork.

------------------------------------------------------------------------

## 3. Asymmetric Solution ⭐ (Interview Favorite)

``` c
if(i % 2 == 0){

    wait(rightFork);     // Even picks RIGHT first
    wait(leftFork);

}
else{

    wait(leftFork);      // Odd picks LEFT first
    wait(rightFork);

}

eat();

signal(leftFork);
signal(rightFork);
```

**Idea:** Different acquisition order breaks the circular wait.

------------------------------------------------------------------------

# Interview Tips

-   Forks = Binary Semaphores.
-   Goal = Avoid Circular Wait.
-   Simple semaphore solution **does NOT** prevent deadlock.
-   Mention **at least one** prevention strategy (4 philosophers, atomic
    pickup, asymmetric ordering).

## One-line Answer

**Dining Philosophers demonstrates process synchronization using
semaphores and shows that synchronization alone cannot prevent deadlocks
unless circular wait is broken.**
