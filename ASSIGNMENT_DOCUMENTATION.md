# Assignment 3 - Complete Documentation

**Student Name**: [Suliman ahmed ALdabaan]  
**Student ID**: [444050061]  
**Date Submitted**: []

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

**Which variables**: 

**Why they need protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #2: Execution Log

**What resource**: 

**Why it needs protection**: 

**Synchronization mechanism used**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Justification**: 

---

### Critical Section #3: CPU Semaphore

**Purpose of semaphore**: 

**Number of permits and why**: 

**Where implemented**: 

**Code snippet**:
```java
// Paste your implementation here
```

**Effect on program behavior**: 

---

## Part 4: Testing and Verification (2 marks)

### Test 1: Consistency Check
**What I tested**: Running program multiple times to verify consistent results

**Testing procedure**: 
```bash
# Commands used (run the program at least 5 times)
```

**Results**: 
(Show that running multiple times produces consistent, correct results)

**Why synchronization is necessary**: 
(Explain what race conditions COULD occur without synchronization, even if you didn't observe them. Explain which shared resources need protection and why.)

**Conclusion**: 

---

### Test 2: Exception Testing
**What I tested**: Checking for ConcurrentModificationException

**Testing procedure**: 

**Results**: 

**What this proves**: 

---

### Test 3: Correctness Verification
**What I tested**: Verifying correct final values (total burst time, context switches, etc.)

**Expected values**: 

**Actual values**: 

**Analysis**: 

---

### Test 4: Different Scenarios
**Scenario tested**: [e.g., different time quantum, more processes, etc.]

**Purpose**: 

**Results**: 

**What I learned**: 

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
