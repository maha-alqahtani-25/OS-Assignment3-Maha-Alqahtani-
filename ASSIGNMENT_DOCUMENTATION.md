# Assignment 3 - Complete Documentation

**Student Name**: [Maha Mobarak Alqahtani]  
**Student ID**: [445052025]  
**Date Submitted**: [6,May ,2026]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: [Paste your personal Gmail Google Drive link here]

**Video filename**: `[YourStudentID]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [May 3, 6:00 PM]
**What I implemented**: 
Initialized the project and created the basic structure including Process class and main scheduler loop.
**Challenges encountered**: 
Understanding how to simulate process execution using threads.
**How I solved it**: 
Reviewed Java threading basics and used Runnable to represent each process.
**Testing approach**: 
Ran simple threads with fixed execution time to verify correct behavior.
**Time spent**: 
1.5 hours
---

### Entry 2 - [May 3, 9:00 PM]
**What I implemented**: 
Implemented Round Robin scheduling logic using a queue.
**Challenges encountered**: 
Managing process re-queuing after each time quantum.
**How I solved it**: 
Used a Queue<Thread> and re-added processes if they were not finished.
**Testing approach**: 
Tested with a small number of processes and printed execution order.
**Time spent**: 
2 hours
---

### Entry 3 - [May 4,5:00 PM]
**What I implemented**: 
Added shared counters (contextSwitchCount, totalWaitingTime, etc.).
**Challenges encountered**: 
Incorrect values due to concurrent updates (race conditions).
**How I solved it**: 
Introduced ReentrantLock to protect shared variables.
**Testing approach**: 
Ran the program multiple times and compared results for consistency.
**Time spent**: 
2 hours
---

### Entry 4 - [May 4,8:30 PM]
**What I implemented**: 
Added Semaphore to control CPU access.
**Challenges encountered**: 
Ensuring only one thread executes at a time without blocking others permanently.
**How I solved it**: 
Used a binary semaphore and ensured proper release using try-finally.
**Testing approach**: 
Verified that only one process runs at a time by observing output.
**Time spent**: 
1.5 hours
---

### Entry 5 - [May 5,7:00 PM]
**What I implemented**: 
Final improvements: logging system, statistics output, and UI formatting.
**Challenges encountered**: 
Avoiding concurrent modification issues in the execution log.
**How I solved it**: 
Added a separate lock (logLock) for the log list.
**Testing approach**: 
Performed full testing with multiple runs and different scenarios.
**Time spent**: 
2 hours
---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[One race condition occurs on the shared variable contextSwitchCount. Multiple threads increment this counter using contextSwitchCount++, which is not an atomic operation. If two threads execute it simultaneously, one update may be lost, resulting in an incorrect count of context switches.

Another race condition occurs on totalWaitingTime, where multiple threads add waiting times using totalWaitingTime += time. Concurrent access can cause inconsistent accumulated values due to overlapping read/write operations.

The problem arises because threads access and modify shared resources without synchronization, leading to unpredictable results. For example, incorrect statistics such as lower context switch counts or wrong total waiting time could be produced.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[ReentrantLock is a mutual exclusion lock that allows only one thread to access a critical section at a time, with more control than synchronized (such as manual lock/unlock). A Semaphore, on the other hand, controls access to a resource by allowing a fixed number of threads (permits).

In my code, I used multiple ReentrantLocks (fine-grained locks) to protect shared counters like contextSwitchCount, completedProcessCount, and totalWaitingTime. This ensures thread-safe updates.

I used a Semaphore (cpuSemaphore) with one permit to simulate CPU access, ensuring that only one process (thread) executes on the CPU at a time. This models real CPU scheduling behavior.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[Deadlock occurs when two or more threads are blocked forever, each waiting for a resource held by another thread. This usually happens due to circular waiting and improper resource handling.

One prevention technique is using try-finally blocks, which ensures that locks are always released even if an exception occurs. In my code, every lock and semaphore acquisition is followed by a finally block that releases it.

Another technique is avoiding circular wait by not holding multiple locks simultaneously or by maintaining a consistent lock acquisition order. In my implementation, each lock is used independently, reducing the risk of deadlock.

Additionally, the semaphore is carefully released after use, ensuring no thread holds the CPU indefinitely.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[I used fine-grained locking by assigning a separate ReentrantLock for each shared counter (contextSwitchLock, completedProcessLock, and waitingTimeLock).

I chose this approach because the three counters are independent, meaning updating one does not affect the others. Using separate locks allows multiple threads to update different counters concurrently, improving performance.

The trade-off is that fine-grained locking increases code complexity and requires careful management, while coarse-grained locking (one lock for all counters) is simpler but reduces concurrency since only one thread can access any counter at a time.

Given that the counters are independent, fine-grained locking provides better concurrency because it minimizes contention and allows more parallel execution.

However, in simpler systems or when operations are highly interdependent, coarse-grained locking may be preferred for simplicity and safety.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: 
The shared counter variables are:
* contextSwitchCount
* completedProcessCount
* totalWaitingTime
**Why they need protection**: 
These variables are shared among multiple threads, and they are updated concurrently. Operations like increment (++) and addition (+=) are not atomic, meaning they involve multiple steps (read → modify → write). Without synchronization, multiple threads could overwrite each other’s updates, leading to incorrect values such as lost increments or inaccurate total waiting time.
**Synchronization mechanism used**: 
I used ReentrantLock (fine-grained locking) by assigning a separate lock for each variable:
* contextSwitchLock
* completedProcessLock
* waitingTimeLock
This ensures that each counter is updated safely while allowing other counters to be accessed concurrently.
**Code snippet**:
```java
// // Increment context switch counter
public static void incrementContextSwitch() {
    contextSwitchLock.lock();
    try {
        contextSwitchCount++;
    } finally {
        contextSwitchLock.unlock();
    }
}

// Increment completed process counter
public static void incrementCompletedProcess() {
    completedProcessLock.lock();
    try {
        completedProcessCount++;
    } finally {
        completedProcessLock.unlock();
    }
}

// Add waiting time
public static void addWaitingTime(long time) {
    waitingTimeLock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        waitingTimeLock.unlock();
    }
}
```

**Justification**: 
I used separate (fine-grained) locks for each counter because they are independent variables. This allows multiple threads to update different counters at the same time without blocking each other, which improves performance. Using one lock for all counters would reduce concurrency and make the program slower without any benefit.
---

### Critical Section #2: Execution Log

**What resource**: 
The shared resource is the executionLog, which is a List<String> used to store log messages from different threads during execution.
**Why it needs protection**: 
Multiple threads may attempt to write to the executionLog at the same time. Since ArrayList is not thread-safe, concurrent modifications can lead to data corruption, inconsistent log entries, or even runtime exceptions like ConcurrentModificationException. Therefore, access must be synchronized to ensure safe and correct logging.
**Synchronization mechanism used**: 
I used a ReentrantLock (logLock) to protect access to the executionLog. This ensures that only one thread can modify the log at a time.
**Code snippet**:
```java
// Method to log execution safely
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
Using a ReentrantLock provides explicit control over locking and ensures thread-safe access to the shared log. I used a separate lock (fine-grained locking) instead of a global lock to reduce contention and allow other independent operations (like updating counters) to proceed concurrently. This improves overall performance while maintaining correctness.
---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 
The semaphore is used to control access to the CPU, ensuring that only one process (thread) executes at a time.
**Number of permits and why**: 
A single permit (1) is used to simulate a single-core CPU, where only one process can run at any moment.
**Where implemented**: 
It is implemented in the SharedResources class and used inside the run() and runToCompletion() methods of the Process class.
**Code snippet**:
```java
// Semaphore declaration
public static final Semaphore cpuSemaphore = new Semaphore(1);

// Usage inside run()
SharedResources.cpuSemaphore.acquire();
try {
    // process execution
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: 
The semaphore ensures that processes execute one at a time, preventing overlapping execution. This simulates real CPU scheduling and avoids conflicts between threads accessing the CPU.
---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: Run the program multiple times (at least 5 times) using:
```bash
javac SchedulerSimulationSync.java
java SchedulerSimulationSync
```

**Results**: 
(Each run produced consistent and correct statistics such as total completed processes, context switches, and waiting time. No incorrect or negative values were observed.)

**Why synchronization is necessary**: 
(Without synchronization, race conditions could occur when multiple threads update shared resources like contextSwitchCount, totalWaitingTime, and executionLog. This may lead to lost updates, incorrect totals, or corrupted logs. These shared resources require protection to ensure data consistency.)

**Conclusion**: 
Synchronization ensures reliable and consistent results across multiple executions.
---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 
Run the program repeatedly while monitoring for runtime exceptions.
**Results**: 
No ConcurrentModificationException or other concurrency-related exceptions occurred.
**What this proves**: 
It proves that shared resources (especially executionLog) are properly synchronized and safely accessed by multiple threads.
---

### Test 3: Correctness Verification
**What I tested**: 
Verification of final statistics (completed processes, waiting time, etc.)
**Expected values**: 
* Completed processes = total number of processes
* Waiting time ≥ 0
* Context switches ≥ number of processes
**Actual values**: 
All values matched expectations and were logically correct.
**Analysis**: 
The synchronization mechanisms ensured accurate calculations and prevented data inconsistencies.
---

### Test 4: Different Scenarios
**Scenario tested**: 
Different time quantum and varying number of processes (random values).
**Purpose**: 
To test program behavior under different workloads.
**Results**: 
The program handled all scenarios correctly and maintained stable performance.
**What I learned**: 
The scheduling logic and synchronization mechanisms work reliably under varying conditions.
---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[6-8 sentences about key concepts, challenges, insights]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: 

**Example 2**: 

---

### How I would explain synchronization to others:

[Explain to someone who just finished Assignment 1 - use simple terms and analogies]

---

## Part 6: GitHub Repository Information

**Repository URL**: 

**Number of commits**: 

**Commit messages**: 
1. 
2. 
3. 
4. 

---

## Summary

**Total time spent on assignment**: 

**Key takeaways**: 
1. 
2. 
3. 

**Most challenging aspect**: 

**What I'm most proud of**: 

---

**End of Documentation**
