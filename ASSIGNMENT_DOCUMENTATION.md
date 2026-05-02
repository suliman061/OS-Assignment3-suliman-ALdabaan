# Assignment 3 - Complete Documentation

**Student Name**: [Suliman ahmed ALdabaan]  
**Student ID**: [444050061]  
**Date Submitted**: [2 MAY]

---

## 🎥 VIDEO DEMONSTRATION LINK (REQUIRED)

> **⚠️ IMPORTANT: This section is REQUIRED for grading!**
> 
> Upload your 3-5 minute video to your **PERSONAL Gmail Google Drive** (NOT university email).
> Set sharing to "Anyone with the link can view".
> Test the link in incognito/private mode before submitting.

**Video Link**: https://drive.google.com/drive/folders/1njMUzRbsDUodf0rduaFvwHnaDqiEqjhN?dmr=1&ec=wgc-drive-%5Bmodule%5D-goto

**Video filename**: `[444050061]_Assignment3_Synchronization.mp4`

**Verification**:
- [ ] Link is accessible (tested in incognito mode)
- [ ] Video is 3-5 minutes long
- [ ] Video shows code walkthrough and commits
- [ ] Video has clear audio
- [ ] Uploaded to PERSONAL Gmail (not @std.psau.edu.sa)

---

## Part 1: Development Log (1 mark)

Document your development process with **minimum 3 entries** showing progression:

### Entry 1 - [1 may, 4:00]
**What I implemented**: 
carried out a thorough code examination of the original simulation. I identified shared resources in SharedResources.java and CPU access by threads in Process.java. I also verified my Student ID 444050061 was configured properly.
**Challenges encountered**: 
determining the precise "Race Conditions" in which many processes were simultaneously updating the totalWaitingTime.
**How I solved it**: 
To find areas where synchronization was lacking, I charted the data flow between the SharedResources class and the Process threads.
**Testing approach**: 
Ran the unsynchronized code multiple times and observed that the final statistics (Average Waiting Time) were inconsistent between runs.
**Time spent**: 
30 min
---

### Entry 2 - [1 may, 4:30]
**What I implemented**: 
 To safeguard the statistics counters (completedProcesses, totalWaitingTime, and totalTurnaroundTime), I added a ReentrantLock to the SharedResources class.
**Challenges encountered**: 
Choosing whether to use separate locks for each counter or one lock for all of them.
**How I solved it**: 
In order to guarantee that all relevant statistics are updated as a single atomic unit and avoid any data corruption during the creation of the final report, I opted for a coarse-grained lock method.
**Testing approach**: 
Verified that the completedProcesses count always matched the number of processes created, even under high thread contention.
**Time spent**: 
around 30 min
---

### Entry 3 - [1 may, 5:00]
**What I implemented**: 
I wrapped the burst execution logic in the Process class's run() function by initializing a Semaphore with one permission in SharedResources.
**Challenges encountered**: 
The execution log was hard to read and unrealistic for a single-core CPU simulation since processes were displaying their "Starting" statements at the same time.
**How I solved it**: 
To make sure that only one process is using the CPU at a time, I used cpuSemaphore.acquire() before the execution log begins and cpuSemaphore.release() in a finally block.
**Testing approach**: 
examined the console output to make sure that each process "Finished" before the subsequent one "Started."
**Time spent**: 
around in hour
---

### Entry 4 - [1 may, 5:55]
**What I implemented**: 
To handle the last process in the queue, I used the same semaphore logic in the runToCompletion() function.
**Challenges encountered**: 
This function required careful treatment to make sure it didn't circumvent the CPU restrictions because it follows a different execution route from the usual run().
**How I solved it**: 
In order to guarantee that the cpuSemaphore permission is released even in the event that the thread is interrupted, I wrapped the method's whole logic in a try-finally block.
**Testing approach**: 
When just one process is left, the simulation behavior was confirmed to adhere to the same mutual exclusion constraints as the other processes.
**Time spent**: 
hour and 10 min
---

### Entry 5 - [1 may, 7:10]
**What I implemented**: 
To print the Semaphore and Lock status at the conclusion of the simulation, I included a final audit section.
**Challenges encountered**: 
Sometimes an error might result in a permit being "lost," leaving the semaphore at 0.
**How I solved it**: 
In order to assure resource recovery, I double-checked each release() call to make sure it was in a finally block.
**Testing approach**: 
A clean execution was demonstrated by the last verification run, which verified that the output displayed Available Permits: 1 and Lock Status: Unlocked.
**Time spent**: 
around 2 hours
---

## Part 2: Technical Questions (1 mark)

### Question 1: Race Conditions
**Q**: Identify and explain TWO race conditions in the original code. For each:
- What shared resource is affected?
- Why is concurrent access a problem?
- What incorrect behavior could occur?

**Your Answer**:

[A race issue in the SharedResources class's totalWaitingTime variable plagued the original code. The action (which is not atomic) might result in "lost updates" when one thread overwrites the increment of another when several process threads execute totalWaitingTime += waitingTime; concurrently. Additionally, concurrent access during the ++ increment procedure had an impact on the completedProcesses counter. The final simulation report would frequently show inaccurate averages and a smaller number of finished processes than really performed because these threads were not synced. For instance, if two threads simultaneously received the number 5, they would both increase it to 6 and write it back rather than the right value 7.]

---

### Question 2: Locks vs Semaphores
**Q**: Explain the difference between ReentrantLock and Semaphore. Where did you use each in your code and why?

**Your Answer**:

[The main distinction is that while a Semaphore maintains a set of permits and can be used for both mutual exclusion and signaling between different threads, a ReentrantLock is a mutual exclusion mechanism that only allows access to one thread at a time and must be released by the same thread that acquired it. Because ReentrantLock is lightweight and guarantees atomic data changes, I utilized it in my implementation to safeguard the statistical counters (such totalWaitingTime) in the SharedResources class. However, I simulated the CPU in the Process class using a Binary Semaphore (1 permit). This was done in order to tightly regulate the "Execution" resource, guaranteeing that only one process thread can use the CPU even while several are prepared.]

---

### Question 3: Deadlock Prevention
**Q**: What is deadlock? Explain TWO prevention techniques and what you did to prevent deadlocks in your code.

**Your Answer**:

[In concurrent programming, a deadlock occurs when two or more threads are stuck waiting for one another to release a resource. I used two crucial strategies to stop this: try-finally blocks and resource ordering. I made sure that SharedResources by utilizing the try-finally pattern.Even in the event of an exception or stoppage during execution, lock.unlock() and cpuSemaphore.release() are always invoked. Additionally, by centralizing shared data access within the SharedResources class, I was able to maintain a rigorous lock hierarchy and avoid circular wait situations where multiple locks may be obtained in different sequences across distinct classes.]

---

### Question 4: Lock Granularity Design Decision 
**Q**: For Task 1 (protecting the three counters), explain your lock design choice:
- Did you use ONE lock for all three counters (coarse-grained) OR separate locks for each counter (fine-grained)?
- Explain WHY you made this choice
- What are the trade-offs between the two approaches?
- Given that the three counters are independent, which approach provides better concurrency and why?

**Your Answer**:

[Using ONE lock for each of the three counters (completedProcesses, totalWaitingTime, and totalTurnaroundTime), I opted for a coarse-grained locking strategy. Since all three counters in this particular simulation are updated at the same logical point—the conclusion of a process execution—I decided against separating them because doing so would not significantly improve performance. Clarity and speed are the trade-offs between the two methods: fine-grained locking (separate locks) improves concurrency by enabling many threads to update distinct counters at once, but it also raises the possibility of deadlocks and increases code complexity. In high-contention systems, coarse-grained locking can become a bottleneck while being considerably easier to create and troubleshoot. The fine-grained technique might theoretically improve concurrency in a large multi-core system even if the counters are independent because it lowers "Lock Contention," enabling threads to change one number without waiting for a lock held on another unrelated counter.]

---

## Part 3: Synchronization Analysis (1 mark)

### Critical Section #1: Counter Variables

**Which variables**: completedProcesses, totalWaitingTime, totalTurnaroundTime.

**Why they need protection**: These variables are shared across all process threads. Without protection, concurrent increments (like ++ or +=) lead to race conditions where updates are lost, resulting in incorrect final statistics.

**Synchronization mechanism used**: ReentrantLock (named lock in SharedResources).

**Code snippet**:
```java
// public static void addWaitingTime(long time) {
    lock.lock();
    try {
        totalWaitingTime += time;
    } finally {
        lock.unlock();
    }
    }
```

**Justification**: Only one thread may alter the statistical data at a time thanks to a lock. To ensure that the lock is released even in the event of a runtime exception, a finally block must be used.

---

### Critical Section #2: Execution Log

**What resource**: The Standard Output (Console) and the logical sequence of process execution.

**Why it needs protection**: to avoid "Interleaving" the output. Without synchronization, the log would be illegible and illogical for a single-core simulation as one process may print "Starting" while another is still reporting its progress.

**Synchronization mechanism used**: Semaphore (Binary).

**Code snippet**:
```java
// try {
    SharedResources.cpuSemaphore.acquire();
    System.out.println(Colors.CYAN + "  ▶ " + name + " is starting burst..." + Colors.RESET);
    Thread.sleep(burst);
    // ... logic ...
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Justification**: The gatekeeper is the semaphore. We make sure the execution log shows a clear, sequential process flow by including the print statements and the sleep (burst) inside the acquire/release block.

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: To simulate the physical limitation of a single-core CPU where only one process can be in the "Running" state at any given time.

**Number of permits and why**: 1 permit. A single permit ensures Mutual Exclusion (Mutex), meaning the capacity of our "resource" (the CPU) is exactly one.

**Where implemented**: Inside the run() method (for standard Round Robin cycles) and the runToCompletion() method (for the final execution phase) of the Process class.

**Code snippet**:
```java
// // Inside runToCompletion()
try {
    SharedResources.cpuSemaphore.acquire();
    // CPU-bound task execution
    Thread.sleep(remainingTime);
} finally {
    SharedResources.cpuSemaphore.release();
}
```

**Effect on program behavior**: During the burst phase, it serializes thread execution. The semaphore creates a realistic simulation of a non-parallel CPU scheduler by forcing Java's many threads to wait in line.

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running the program multiple times to verify consistent results in the final statistics.

**Testing procedure**: 
```bash
# # Executed the main class 5 consecutive times in the IDE
java CPUScheduler (Run 1)
java CPUScheduler (Run 2)
java CPUScheduler (Run 3)
java CPUScheduler (Run 4)
java CPUScheduler (Run 5)
```

**Results**: 
(In all 5 runs, the Average Waiting Time and Average Turnaround Time remained identical to the second decimal place. The Available Permits at the end of every run was consistently 1.)

**Why synchronization is necessary**: 
(Without synchronization, a race condition could occur on the totalWaitingTime variable. Even if not observed in every run, two threads could update the variable at the exact same time, causing one update to be lost. This would lead to a lower total waiting time and incorrect averages, which is unacceptable in a simulation.)

**Conclusion**: The implementation is stable and produces deterministic results.

---

### Test 2: Exception Testing
**What I tested**: Verifying that the CPU Semaphore is released even if a thread is interrupted.

**Testing procedure**: I simulated a thread interruption during the Thread.sleep(burst) phase using a try-catch block and observed the finally block execution.

**Results**: When an interruption was caught, the console printed the error message, and the finally block immediately executed cpuSemaphore.release().

**What this proves**: This proves the system is Deadlock-proof against unexpected thread failures. The CPU resource will never be "stuck" at 0 permits if a process crashes.

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values for the simulation based on Student ID 444050061.

**Expected values**: Completed Processes: 5

Final CPU Semaphore Status: 1 (All permits returned)

Lock Status: Unlocked

**Actual values**: Completed Processes: 5

Final CPU Semaphore Status: 1

Lock Status: Unlocked

**Analysis**: The results match the expected logic perfectly. The ReentrantLock ensured all 5 processes were counted, and the Semaphore logic ensured the CPU was fully vacated after the simulation finished.

---

### Test 4: Different Scenarios
**Scenario tested**:Running the simulation with a very small Time Quantum (e.g., 5ms) to increase context switching

**Purpose**: To stress-test the synchronization mechanisms by forcing more frequent acquire() and release() calls.

**Results**: The program handled the high frequency of context switches without any "Interleaving" in the logs and without any race conditions in the counters.

**What I learned**: I learned that robust synchronization (using try-finally) is even more critical when the system has high contention or frequent state changes. Proper locking ensures data integrity regardless of how many times the processes switch in and out of the CPU.

---

## Part 5: Reflection and Learning

### What I learned about synchronization:

[Through this assignment, I learned that synchronization is not just about adding locks, but about managing the orderly access of threads to shared resources. I grasped the critical concept of "Mutual Exclusion," ensuring that only one process occupies the CPU at a time to maintain a realistic simulation. One of the biggest challenges was identifying all potential race conditions, especially those affecting statistical counters like totalWaitingTime. I also gained a deep insight into the importance of the try-finally pattern; without it, any runtime exception could leave a lock permanently held, leading to a system-wide deadlock. Furthermore, I learned how to distinguish between signaling (using Semaphores) and data protection (using Locks). Ultimately, this project showed me that robust concurrent programming requires a proactive design to prevent data corruption before it happens.]

---

### Real-world applications:

Give TWO examples where synchronization is critical:

**Example 1**: When multiple transactions (like a withdrawal and a transfer) happen on the same account simultaneously, synchronization ensures the balance is updated correctly and prevents "Double Spending" or incorrect balance reporting.

**Example 2**: When thousands of users try to buy the last remaining item in stock, synchronization prevents the system from overselling the item by ensuring only one thread can decrement the stock count at a time.

---

### How I would explain synchronization to others:

[Imagine a busy coffee shop with only one espresso machine. If every barista (Thread) tried to use the machine at the exact same time, they would crash into each other and spill the coffee (Race Condition). Synchronization is like a "waiting line" or a "rule" that says: "Only the barista holding the special gold key can use the machine." Once they finish making the coffee, they hand the key to the next person in line. This ensures that every cup is made perfectly without any mess, even though many baristas are working in the shop at once.]

---

## Part 6: GitHub Repository Information

**Repository URL**: https://github.com/suliman061/OS-Assignment3-suliman-ALdabaan.git

**Number of commits**: 7

**Commit messages**: 
1. added locks and semaphore to shared resources and import package for both of them
2. protecting counters using mutex locks
3. implementing cpu semaphore to prevent process overlapping and changed finally postion
4. Finalizing synchronization for all process execution paths

---

## Summary

**Total time spent on assignment**: 7 hours

**Key takeaways**: 
1. Race conditions can lead to silent but catastrophic data corruption in multi-threaded environments.
2. The finally block is the most reliable way to prevent deadlocks by ensuring resource release.
3. Proper synchronization creates deterministic and consistent behavior in otherwise unpredictable thread executions.

**Most challenging aspect**: Ensuring that the runToCompletion() method—which is an edge-case execution path—was perfectly synchronized with the main run() method so they never overlapped.

**What I'm most proud of**: Achieving a "Clean Exit" where the final audit shows exactly 1 available permit, proving that my resource management logic is 100% leak-proof.

---

**End of Documentation**
