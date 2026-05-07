# Assignment 3 - Complete Documentation

**Student Name**: Reema Saeed Alqahtani 
**Student ID**: 445052064 
**Date Submitted**: May 7, 2026

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: https://drive.google.com/file/d/13ycqQQnds6nv_kGskf9vQKtQYFuuo9dG/view?usp=drive_link

**Video filename**: `445052064_Assignment3_Synchronization.mp4`

**Verification**:
- [x] Link is accessible (tested in incognito mode)
- [x] Video is 3-5 minutes long
- [x] Video shows code walkthrough and commits
- [x] Video has clear audio
- [x] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - May 6, 2026, 6:30 PM
**What I implemented**: 
I started the project in Visual Studio Code, adjusted the student ID in `SchedulerSimulationSync.java`, examined the assignment criteria, then forked and cloned the starting repository.

**Challenges encountered**: 
Initially, I had to figure out which resources are shared by threads and comprehend how the scheduling simulation functions.

**How I solved it**: 
After looking into the `SharedResources` class, I found that the shared resources that need to be synchronized are `contextSwitchCount`, `completedProcessCount`, `totalWaitingTime`, and `executionLog`.

**Testing approach**: 
Before implementing synchronization, I made sure the scheduler simulation was functioning by compiling and running the original application.
**Time spent**: 
About 40 minutes.
---

### Entry 2 - May 7, 2026, 03:00 PM
**What I implemented**: 
I included the synchronization imports for Semaphore and ReentrantLock. In the `SharedResources` class, I also built `counterLock`, `logLock`, and `cpuSemaphore`.

**Challenges encountered**: 
Choosing how to set up synchronization for counters and logs was the difficult part.

**How I solved it**: 
To keep synchronization duties clear, I utilized a single shared lock for the counter variables and a different lock for the execution log.

**Testing approach**: 
I verified that the imports were appropriately built and that the creation of `new Semaphore(1)` was successful.

**Time spent**: 
About 35 minutes.
---

### Entry 3 - May 7, 2026, 04:20 PM
**What I implemented**: 
I used `counterLock` to safeguard the shared counter methods. `incrementContextSwitch()`, `incrementCompletedProcess()`, and `addWaitingTime(long time)` were the protected methods.

**Challenges encountered**: 
To prevent deadlock, I had to make sure that every lock acquisition had a guaranteed release.

**How I solved it**: 
In each counter technique, I employed `try-finally` blocks. This ensures that even in the event of an exception, the lock will be released.

**Testing approach**: 
I thoroughly examined every synchronized function, making sure that each call to `lock()` had a corresponding call to `unlock()` inside of `finally`.

**Time spent**: 
About 30 minutes.

---

### Entry 4 - May 7, 2026, 05:30 PM
**What I implemented**: 
In the `logExecution(String message)` function, I used `logLock` to safeguard the `executionLog` list.

**Challenges encountered**: 
The `ArrayList` used in the execution log is not thread-safe when accessed concurrently by several threads.

**How I solved it**: 
To ensure exclusive access when adding log entries, I employed a different `ReentrantLock` named `logLock`.

**Testing approach**: 
I confirmed that the synchronized `logExecution()` function is the only way to update `executionLog`.

**Time spent**: 
About 20 minutes.

---

### Entry 5 - May 7, 2026, 06:40 PM
**What I implemented**: 
Semaphore synchronization was incorporated into the `run()` function. The CPU semaphore is now acquired by each process before to execution and released in a `finally` block.

**Challenges encountered**: 
Correctly handling `InterruptedException` and ensuring that the semaphore is released only after it has been successfully obtained was a problem.

**How I solved it**: 
Before executing `release()`, I introduced a boolean variable named `acquired` and utilized it within the `finally` section.

**Testing approach**: 
After implementing semaphore synchronization, I examined the scheduler's execution to make sure the software continued to run all processes appropriately.

**Time spent**: 
About 1 hour.

---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

Because many threads may execute `contextSwitchCount++` at the same time, there is only one race situation in `contextSwitchCount`. Updates might be missed and the final context switch count could be off since this procedure is not atomic. Because `ArrayList` is not thread-safe when used by several threads simultaneously, there is an additional race condition in `executionLog`. Concurrent changes have the potential to erase execution entries or contaminate the log. Using `ReentrantLock` to safeguard shared resources, I was able to resolve these issues.

```java
counterLock.lock();
try {
    contextSwitchCount++;
} finally {
    counterLock.unlock();
}
```

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

ReentrantLock ensures mutual exclusion between threads and protects important parts. I used `logLock` to prevent concurrent access to the `executionLog` ArrayList and `counterLock` to safeguard shared counters. Permits are used by a `Semaphore` to manage access to a restricted resource. To mimic CPU access and limit the number of processes that may run at once, I used `cpuSemaphore = new Semaphore(1)`. Thus, the semaphore was employed to govern CPU scheduling, while locks were utilized to safeguard shared data.

```java
public static final ReentrantLock counterLock = new ReentrantLock();
public static final Semaphore cpuSemaphore = new Semaphore(1);
```

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

When many threads wait endlessly for resources owned by one another, a deadlock develops. Releasing locks and semaphores inside of `finally` blocks is one preventive strategy that ensures resource release even in the case of an error. Another strategy is to prevent needless nested locking and to keep important portions brief. In my implementation, I released semaphores and locks within the `finally` portions of the `run()` and `runToCompletion()` functions. This lessens the chance of deadlock and stops threads from hoarding resources indefinitely.

```java
finally {
    if (acquired) {
        SharedResources.cpuSemaphore.release();
    }
}
```

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

I used one shared lock called `counterLock` for the three counter variables, so my design is coarse-grained locking. I chose this approach because the counter updates are short and simple, so one lock makes the code easier to read and less error-prone. The advantage of coarse-grained locking is simplicity and easier debugging. The disadvantage is that it reduces concurrency because only one thread can update any counter at a time. Fine-grained locking would use a separate lock for each counter. Since the three counters are independent, fine-grained locking can provide better concurrency because different threads could update different counters at the same time. However, fine-grained locking also adds more complexity. For this assignment, I chose one shared lock because correctness and clarity were more important than maximum concurrency.

```java
public static final ReentrantLock counterLock = new ReentrantLock();
```

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**:
`contextSwitchCount`, `completedProcessCount`, and `totalWaitingTime`.

**Why they need protection**:
These variables are shared resources accessed by multiple process threads. Without synchronization, two or more threads may update the same counter at the same time, which can cause lost updates and incorrect final statistics.

**Synchronization mechanism used**:
`ReentrantLock` using one shared lock called `counterLock`.

**Code snippet**:
```java
public static final ReentrantLock counterLock = new ReentrantLock();

public static void incrementContextSwitch() {
    counterLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        counterLock.unlock();
    }
}

public static void incrementCompletedProcess() {
    counterLock.lock();
    try {
        completedProcessCount++;
    } finally {
        counterLock.unlock();
    }
}

public static void addWaitingTime(long time) {
    counterLock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        counterLock.unlock();
    }
}
```

**Justification**: 
Only one thread may alter the shared counter variables at a time since the lock ensures mutual exclusion. This maintains the final scheduler statistics accurate and avoids race situations.
---

### Critical Section #2: Execution Log

**What resource**: 
`executionLog` implemented using `ArrayList<String>`.

**Why it needs protection**: 
`ArrayList` is not thread-safe. Multiple process threads may add log entries simultaneously, which can corrupt the list or lose execution messages.

**Synchronization mechanism used**: 
`ReentrantLock` using a separate lock called `logLock`.

**Code snippet**:
```java
public static final ReentrantLock logLock = new ReentrantLock();

public static void logExecution(String message) {
    logLock.lock();
    try {
        executionLog.add(message);
    } finally {
        logLock.unlock();
    }
}
```

**Justification**: 
Only one thread may alter the execution log at a time thanks to the lock. This ensures accurate log entries and avoids concurrent modification issues.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore controls access to the simulated CPU resource and ensures that only one process executes its CPU section at a time.

**Number of permits and why**: 
I used one permit with `new Semaphore(1)` because the scheduler simulates a single CPU. This creates a binary semaphore that allows only one process thread to execute at a time.

**Where implemented**: 
The semaphore was implemented inside `SharedResources` and used in both `run()` and `runToCompletion()` methods.

**Code snippet**:
```java
public static final Semaphore cpuSemaphore = new Semaphore(1);

boolean acquired = false;

try {
    SharedResources.cpuSemaphore.acquire();
    acquired = true;

    // process execution code

} finally {
    if (acquired) {
        SharedResources.cpuSemaphore.release();
    }
}
```

**Effect on program behavior**: 
By preventing numerous process threads from performing crucial CPU parts at the same time, the semaphore ensures restricted CPU access. By doing this, resource conflicts are avoided and synchronization accuracy is improved.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
java SchedulerSimulationSync
```

**Results**: 
```text
           ALL PROCESSES COMPLETED
═══ Synchronization Statistics ═══
Total Context Switches: 35
Total Completed Processes: 17
Total Waiting Time: 1257050ms
Average Waiting Time: 73944ms

═══ Process Summary Table ═══
Process    Priority     Burst Time   Waiting Time
────────────────────────────────────────────────
P1         5            6348         61235
P2         5            5077         63587
P3         2            8872         89758
P4         5            4158         68693
P5         3            9461         90637
P6         4            8645         92108
P7         1            6021         76883
P8         3            3500         28214
P9         1            3198         31740
P10        4            8613         92757
P11        2            8076         93378
P12        1            4115         86929
P13        5            6819         87049
P14        1            3890         51010
P15        1            4377         89877
P16        4            7487         90258
P17        2            2282         62937

═══ Execution Log Summary ═══
Total log entries: 70
```

**Why synchronization is necessary**: 
Synchronization is necessary because shared resources such as contextSwitchCount, completedProcessCount, totalWaitingTime, and executionLog may be accessed by multiple threads at the same time. Without locks, counter updates may be lost. Without protecting the ArrayList, log entries may be corrupted or lost.

**Conclusion**: 
The synchronized scheduler completed successfully and produced correct final statistics, which shows that the shared resources were protected properly.

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
I executed the scheduler multiple times while monitoring the execution log updates generated by concurrent process threads. I verified that all log updates were performed through the synchronized `logExecution()` method protected by `logLock`.

**Results**: 
The program completed successfully without throwing `ConcurrentModificationException` or producing corrupted log entries. The final output consistently displayed `Total log entries: 70`.

**What this proves**: 
This proves that `logLock` correctly synchronized access to the `executionLog` ArrayList and prevented concurrent modification problems caused by multiple threads.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 
The completed process count should match the total number of generated processes. The execution log should contain valid entries, and the scheduler should complete without synchronization errors.

**Actual values**: 
═══ Synchronization Statistics ═══
Total Context Switches: 35
Total Completed Processes: 17
Total Waiting Time: 1257050ms
Average Waiting Time: 73944ms


**Analysis**: 
The final output confirms that all generated processes completed execution successfully. The completed process count matched the number of generated processes, which indicates correct synchronization behavior. The execution log was also updated correctly without missing entries or concurrency errors.

---

### Test 4: Different Scenarios
**Scenario tested**: Running the scheduler multiple times using the same student ID seed and observing process execution behavior.

**Purpose**: 
The purpose was to verify that the scheduler behaves consistently and that synchronization protects shared resources correctly during concurrent execution.

**Results**: 
The scheduler successfully completed all 17 processes in every execution. The synchronization statistics and execution logs were printed correctly without crashes or concurrency issues.

**What I learned**: 
I learned that synchronization is essential in multithreaded systems because race conditions may produce unpredictable results. Using locks and semaphores guarantees safe access to shared resources and improves program reliability.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

I gained a better understanding of synchronization in multithreaded operating system applications thanks to this project. I discovered that when numerous threads access shared resources like counters and logs at the same time, they may become inconsistent. Additionally, I discovered that non-atomic operations like `count++` might result in race problems. I was able to establish mutual exclusion for crucial areas by using `ReentrantLock`. I was able to manage access to the simulated CPU resource by using `Semaphore`. In order to prevent deadlocks, I also discovered how crucial it is to release synchronization resources inside of "finally" blocks. Overall, this assignment made the connection between real-world Java implementation and operating system principles including critical sections, race situations, semaphores, and deadlock avoidance.

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 
Banking systems use synchronization to protect account balances during deposits and withdrawals performed by multiple users simultaneously.

**Example 2**: 
Operating systems use synchronization when multiple processes access shared hardware resources such as CPUs, memory, files, or printers.

---

### How I would explain synchronization to others:

Synchronization is similar to controlling access to a shared room. If many people try to modify the same document at the same time, the document may become corrupted or inconsistent. A lock acts like a key that allows only one person to enter and modify the document at a time. A semaphore works similarly but allows a limited number of people depending on the number of available permits. In this assignment, locks protected shared variables while the semaphore controlled access to the CPU.

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/Reema21415/OS-Assignment3-Reema-Alqahtani

**Number of commits**: 12

**Commit messages**: 
1. Set my student ID
2. adding ReentrantLock and import package
3. adding Semaphore and import its package for synchronization
4. Protect context switch counter with ReentrantLock
5. Protect completed process counter with ReentrantLock
6. Protect total waiting time with ReentrantLock
7. Protect execution log with ReentrantLock
8. Implement semaphore acquire and exception handling in run method
9. Release CPU semaphore in finally block
10. Release CPU semaphore after final process execution
11. last update
12. Complete ASSIGNMENT_DOCUMENTATION answers
13. Finalize assignment submission

---

## Summary

**Total time spent on assignment**: 
Approximately 5-6 hours across two days.

**Key takeaways**: 
1. Shared resources in multithreaded programs must be synchronized to prevent race conditions.
2. `ReentrantLock` provides mutual exclusion for critical sections and shared data protection.
3. `Semaphore` controls access to limited resources such as the simulated CPU.


**Most challenging aspect**: 
The most challenging part was implementing semaphore synchronization correctly in `run()` and `runToCompletion()` while ensuring that resources are always released safely using `finally` blocks.

**What I'm most proud of**: 
I am most proud of successfully implementing synchronization mechanisms and verifying the correctness of the scheduler using real multithreaded execution output.

---

**End of Documentation**
