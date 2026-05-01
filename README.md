# 🖥️ Parallel Computer Architecture (PCA)
## UNIT 5 — Operating System Issues for Multiprocessing
### *OS Ka Scene Samajh Lo — Last Unit, Full Marks! 🎯*

---

> **📌 Unit 5 ke baad tu:**
> - OS multiprocessing mein kya karta hai — samjhega
> - Scheduling techniques — sab types clear honge
> - Gang Scheduling — kya hota hai exactly
> - IPC — message boxes, shared memory — dono explain kar payega
> - Distributed semaphores, monitors, spin-locks — yaad honge
> - OpenMP + MPI — implementation level pe samjhega
> - **Exam mein full confidence! 💪**

---

## 📚 UNIT 5 Complete Syllabus Checklist

- [x] Need for Pre-emptive OS
- [x] Scheduling Techniques
- [x] Threads (OS Perspective)
- [x] Distributed Scheduler
- [x] Multiprocessor Scheduling — Gang Scheduling
- [x] Communication Between Processes
  - [x] Message Boxes (Message Passing)
  - [x] Shared Memory
- [x] Sharing Issues and Synchronization
  - [x] Distributed Semaphores
  - [x] Monitors
  - [x] Spin-locks
- [x] Implementation on Multi-cores
  - [x] OpenMP
  - [x] MPI

---

# 🔵 TOPIC 1: Need for Pre-emptive OS

## OS Ka Role Multiprocessing Mein

> **Normal OS** = Ek ghar ka caretaker — ek hi kamra, ek hi guest
> **Multiprocessing OS** = Hotel manager — kai kamare, kai guests, sab ko manage karo!

## Pre-emptive vs Non-Pre-emptive OS

### Non-Pre-emptive (Cooperative) OS:
> Process tab tak CPU use karta hai jab tak **khud** nahi chod deta

```
Process 1: Running.....Running.....Running..... (never gives up!)
Process 2: Waiting.....Waiting.....Waiting..... 😤
OS: Kuch nahi kar sakta — process ne CPU nahi choda!
```

**Problems:**
- Ek process crash ho jaaye → Poora system hang!
- Greedy process sab CPU kha jaaye
- Real-time tasks miss ho jaayein
- **Example:** Old Windows 3.1

---

### Pre-emptive OS:
> OS **forcefully** CPU kisi process se cheen sakta hai aur doosre ko de sakta hai!

```
Process 1: Running [TIMER INTERRUPT!] → Stopped by OS!
Process 2: Gets CPU now!
Process 1: Waiting (will get CPU later)
OS: Main control mein hun! 😎
```

**How Pre-emption Works:**
```
1. Timer chip generates interrupt every few milliseconds (time quantum)
2. OS interrupt handler runs
3. OS decides: Should I switch processes?
4. If yes → Save current process state (context switch)
5. Load next process state → Run it
```

## Why Multiprocessing NEEDS Pre-emptive OS?

### Reason 1: Fairness
> Multiple processors pe multiple processes hain — sab ko fair share chahiye
> Bina pre-emption ke: ek process sab resources hog kar le!

### Reason 2: Real-time Guarantees
> Parallel systems mein time-critical tasks hote hain (e.g., sensor processing)
> Pre-emptive OS high-priority tasks ko immediately run kar sakta hai

### Reason 3: Deadlock Prevention
```
Without pre-emption:
Process A holds Resource 1, waiting for Resource 2
Process B holds Resource 2, waiting for Resource 1
→ DEADLOCK! OS can do nothing without pre-emption!

With pre-emption:
OS can forcefully take Resource 1 from Process A → Give to B → B completes → A gets resource
→ Deadlock RESOLVED!
```

### Reason 4: Load Balancing
> Multiprocessor system mein kuch processors khali, kuch busy
> Pre-emptive OS process ko ek processor se doosre pe MIGRATE kar sakta hai

### Reason 5: Process Coordination
> Parallel processes ek doosre pe wait karte hain
> Pre-emptive OS waiting processes ko block kar sakta hai → CPU free karo!

## Context Switch — OS Ka Trick

> **Context Switch** = Ek process ka "game save" karke doosre ka "game load" karna

```
Context Save (of Process A):
→ Save: PC (Program Counter)
→ Save: All Registers (R0-R15)
→ Save: Stack Pointer
→ Save: Memory Management Registers
→ Save all in Process Control Block (PCB)

Context Load (of Process B):
→ Load B's PCB
→ Restore all registers
→ Jump to B's saved PC
→ B continues from where it left off!
```

**Context Switch Cost:**
- Time: ~1-100 microseconds (overhead!)
- Too frequent → Thrashing (more switching than working!)
- Too infrequent → Poor responsiveness

## 🧠 Pre-emptive OS Trick

> **"Pre-emptive = Police wali OS"** 👮
> Police force se rok sakti hai (pre-empt), non-pre-emptive = begging karna padta hai process ko rokne ke liye!
>
> **Reasons = "FRLPCD"** = Fairness, Real-time, Load balancing, Process coordination, Deadlock handling

---

## ✍️ EXAM TEMPLATE — Need for Pre-emptive OS

```
📝 Answer Template:

[Definition]
A pre-emptive operating system can forcibly interrupt a running process
and allocate the CPU to another process, without waiting for the current
process to voluntarily release the CPU.

[Non-Pre-emptive Problem]
In a non-pre-emptive OS, a process retains the CPU until it completes
or voluntarily yields. This causes problems in multiprocessor systems:
• A single process can monopolize all resources
• High-priority tasks cannot preempt low-priority ones
• Deadlocks cannot be resolved

[Why Multiprocessing Needs Pre-emptive OS]
1. Fairness: Ensures all processes get fair CPU time across
   multiple processors.

2. Real-time Support: High-priority tasks can immediately preempt
   low-priority ones to meet timing constraints.

3. Deadlock Prevention: OS can forcibly reclaim resources from
   deadlocked processes.

4. Load Balancing: OS can migrate processes between processors
   to maintain even workload distribution.

5. Process Coordination: Blocked processes release CPU, allowing
   other useful work to proceed.

[Context Switch]
Pre-emption involves a context switch — saving the current process
state (PC, registers, stack) into its PCB and loading another
process's state. This has overhead but is essential for multiprocessing.

[Conclusion]
Pre-emptive OS is essential for multiprocessor systems to ensure
fairness, real-time responsiveness, deadlock handling, and efficient
resource utilization across multiple processors.
```

---

# 🔵 TOPIC 2: Scheduling Techniques

## Scheduling Kya Hai?

> **Scheduling** = OS ka kaam — decide karna ki **KAUNSA process KAUNSE processor pe KAB chalega**

> 🚦 **Analogy:** Traffic police — kai roads (processors) pe kai gaadiyaan (processes) — police decide karti hai kaunsi gaadi kab kahan jaayegi!

## Scheduling Goals:

```
1. Maximize CPU Utilization    → CPU hamesha busy rahe
2. Maximize Throughput         → Zyada se zyada processes complete ho
3. Minimize Turnaround Time    → Process start se end tak time kam ho
4. Minimize Waiting Time       → Process queue mein kam wait kare
5. Minimize Response Time      → User ko fast response mile
6. Fairness                    → Sab processes ko equal chance
```

## Important Terms:

| Term | Meaning |
|------|---------|
| **Arrival Time** | Process kab aaya |
| **Burst Time** | CPU time kitna chahiye |
| **Completion Time** | Process kab khatam hua |
| **Turnaround Time** | Completion - Arrival |
| **Waiting Time** | Turnaround - Burst Time |
| **Response Time** | First CPU se pehle kitna wait |

## Key Scheduling Algorithms:

---

### 1. FCFS — First Come First Served

> Jis process ne pehle request ki, use pehle CPU mila!

```
Process  Arrival  Burst
P1        0        5
P2        1        3
P3        2        2

Gantt Chart:
[P1: 0-5] [P2: 5-8] [P3: 8-10]

Waiting Time:
P1: 0
P2: 5-1 = 4
P3: 8-2 = 6
Average WT = (0+4+6)/3 = 3.33
```

**Problem:** Convoy effect — ek bada process sab ko rokta hai!

---

### 2. SJF — Shortest Job First

> Sabse chota burst time wala process pehle!

```
Process  Arrival  Burst
P1        0        5
P2        1        3
P3        2        2

At time 0: Only P1 available → P1 runs
At time 1: P2 arrives (burst=3)
At time 2: P3 arrives (burst=2) ← Shortest!
After P1: Run P3 (2), then P2 (3)

Gantt Chart:
[P1:0-5][P3:5-7][P2:7-10]
```

**Problem:** Starvation — bade processes hamesha wait karte rahte hain!

---

### 3. Round Robin (RR)

> Har process ko ek fixed time quantum milta hai, phir queue mein jaata hai!

```
Time Quantum = 2

Process  Burst
P1        5
P2        3
P3        2

Gantt Chart:
[P1:0-2][P2:2-4][P3:4-6][P1:6-8][P2:8-9][P1:9-10]

Most fair! Used in most OS for interactive processes.
```

**Best for:** Time-sharing systems, interactive processes

---

### 4. Priority Scheduling

> Har process ko priority assign hoti hai — high priority process pehle!

```
Process  Burst  Priority
P1        5        3
P2        3        1  ← Highest priority (lowest number = highest priority)
P3        2        2

Execution order: P2 → P3 → P1

Problem: Low priority processes may STARVE!
Solution: Aging — priority badhao agar zyada wait ho raha hai
```

---

### 5. Multilevel Queue Scheduling

> Processes different queues mein hote hain — har queue ki alag priority aur algorithm!

```
Queue 1 (Highest Priority): System processes    [RR, quantum=1]
Queue 2: Interactive processes                   [RR, quantum=4]
Queue 3: Batch processes (Lowest Priority)       [FCFS]

OS pehle Queue 1 serve karta hai, phir 2, phir 3
Lower queues tabhi chalte hain jab higher queues EMPTY hon!
```

---

### 6. Multilevel Feedback Queue (MLFQ)

> Processes queues ke beech MOVE kar sakte hain based on behavior!

```
Queue 1 (quantum=2):   New processes → if not done in 2 → move down
Queue 2 (quantum=4):   If not done in 4 → move down
Queue 3 (FCFS):        Long-running batch processes

CPU-bound processes gradually sink to lower queues
I/O-bound processes stay in higher queues (good response!)
```

## 📊 Scheduling Algorithms Comparison

| Algorithm | Type | Preemptive? | Starvation? | Best For |
|-----------|------|-------------|-------------|----------|
| FCFS | Non-preemptive | No | No | Simple batch |
| SJF | Both | Optional | Yes | Minimize avg wait |
| RR | Preemptive | Yes | No | Time-sharing |
| Priority | Both | Optional | Yes | Real-time |
| MLFQ | Preemptive | Yes | Possible | General purpose |

## 🧠 Scheduling Trick

> **"FC-SJ-RR-PR-MLFQ"** = FCFS, SJF, Round Robin, Priority, MLFQ
> **"First Serve Small Round Priority Multilevel"**

---

# 🔵 TOPIC 3: Threads — OS Perspective

## Thread Kya Hai? (OS Ke Nazar Se)

> OS thread ko ek **independent execution unit** ke roop mein manage karta hai

**OS Thread Types:**

### 1. User-Level Threads (ULT)
> Thread library OS ke upar — OS ko pata nahi threads hain!

```
User Space:
┌────────────────────────────────┐
│  Process                       │
│  Thread Library (manages ULTs) │
│  T1  T2  T3  T4                │
└────────────────────────────────┘
         │
Kernel Space:
┌────────────────────────────────┐
│  OS sees only ONE process!     │
│  (unaware of T1, T2, T3, T4)  │
└────────────────────────────────┘
```

**Advantages:** Fast thread switching (no kernel involvement), portable
**Disadvantages:** If one thread blocks → ALL threads block (OS blocks whole process)

---

### 2. Kernel-Level Threads (KLT)
> OS directly manages each thread!

```
User Space:
┌────────────────────────────────┐
│  T1  T2  T3  T4                │
└────────────────────────────────┘
         │  (system calls)
Kernel Space:
┌────────────────────────────────┐
│  OS knows about T1, T2, T3, T4 │
│  Schedules each independently  │
└────────────────────────────────┘
```

**Advantages:** True parallelism on multi-core, one thread blocks → others continue
**Disadvantages:** Slower (kernel involvement for every switch)

---

### 3. Hybrid Threads (M:N Model)
> M user threads mapped to N kernel threads (M > N)

```
User threads: T1 T2 T3 T4 T5 T6  (6 user threads)
                 ↕↕↕
Kernel threads: K1  K2  K3         (3 kernel threads)
Best of both worlds!
```

## Thread States in OS:

```
     ┌─────────────────────────────────────────┐
     │                                          │
  [NEW] ──create──► [READY] ──schedule──► [RUNNING]
                      ▲                      │    │
                      │               preempt│    │block
                      └──────────────────────┘    │
                                                   ▼
                                              [BLOCKED/WAITING]
                                                   │
                                              event occurs
                                                   │
                                              [READY again]
                                                   
  [TERMINATED] ◄──exit──[RUNNING]
```

## 🧠 Thread Trick

> **ULT = User ka control** (fast but no true parallelism per OS view)
> **KLT = Kernel ka control** (slower but true multicore parallelism)
> **Hybrid = Best of both!**

---

# 🔵 TOPIC 4: Distributed Scheduler

## Distributed Scheduler Kya Hai?

> **Single processor** mein: Ek scheduler, ek queue, simple!
> **Multiprocessor** mein: Kai processors, kai queues — scheduling complex ho jaati hai!

**Distributed Scheduler** = Scheduling decisions **distributed** hain — har processor apna scheduling decision le sakta hai

## Types of Multiprocessor Scheduling Approaches:

---

### Approach 1: Centralized Scheduler (Single Queue)

```
           ┌─────────────┐
All        │   SINGLE    │
Processes ►│   READY     │──► CPU1
           │   QUEUE     │──► CPU2
           └─────────────┘──► CPU3
           
One scheduler makes all decisions
```

**Advantages:** Simple, globally optimal, automatic load balancing
**Disadvantages:** Scheduler becomes bottleneck! (Contention for single queue lock)

---

### Approach 2: Distributed Scheduler (Per-CPU Queues)

```
Process → CPU1's Queue → CPU1
Process → CPU2's Queue → CPU2
Process → CPU3's Queue → CPU3
          ↕ Work Stealing ↕
```

**Each CPU has its OWN ready queue!**
**Work Stealing:** Ek CPU idle ho → doosre CPU ki queue se kaam chura lo!

**Advantages:** No central bottleneck, scalable, cache-friendly (process stays on same CPU)
**Disadvantages:** Load imbalance possible, complex implementation

---

### Approach 3: Hierarchical Scheduler

```
Global Scheduler (coarse-grained decisions)
        │
Local Schedulers (fine-grained, per-cluster)
        │
CPU schedulers (per-processor)
```

## Work Stealing — Important Concept!

> **Idle CPU steals work from overloaded CPU**

```
CPU1: [P1][P2][P3][P4]   ← Overloaded!
CPU2: [P5][P6]
CPU3: []                  ← IDLE → Steals from CPU1!

After stealing:
CPU1: [P1][P2]
CPU2: [P5][P6]
CPU3: [P3][P4]            ← Now balanced!
```

**Used in:** Java ForkJoinPool, OpenMP runtime, Go scheduler

## NUMA-Aware Scheduling:

> In NUMA systems, memory access latency depends on which processor owns the memory
> OS scheduler should try to run processes on the SAME processor that owns their memory

```
NUMA Node 1: CPU1, CPU2, Memory A
NUMA Node 2: CPU3, CPU4, Memory B

Process using Memory A → Schedule on CPU1 or CPU2!
(Accessing Memory A from CPU3 = Remote access = SLOW!)
```

## 🧠 Distributed Scheduler Trick

> **"CDA"** = Centralized (one queue), Distributed (per-CPU queue), Affinity (NUMA-aware)
> **Work Stealing = "Chori karo agar khali ho"** 😄

---

## ✍️ EXAM TEMPLATE — Distributed Scheduler

```
📝 Answer Template:

[Introduction]
A distributed scheduler manages process/thread scheduling across
multiple processors in a multiprocessor system. Unlike single-processor
scheduling, it must handle load distribution and processor affinity.

[Approaches]
1. Centralized Scheduling:
   A single global ready queue serves all processors.
   Simple and ensures good load balancing automatically.
   Drawback: Single queue becomes a bottleneck with many processors.

2. Distributed Scheduling (Per-CPU Queues):
   Each processor has its own ready queue.
   Processes are initially assigned to queues by a dispatcher.
   Idle processors use work-stealing to take tasks from busy ones.
   Advantage: Scalable, no central bottleneck, cache-friendly.
   Disadvantage: Load imbalance possible without work-stealing.

3. Hierarchical Scheduling:
   Combines global scheduler (coarse decisions) with local schedulers
   (fine-grained decisions per CPU cluster).

[Work Stealing]
When a processor's queue is empty, it randomly selects another
processor and steals half of its tasks. This ensures load balance
without a centralized bottleneck.

[NUMA-Aware Scheduling]
In NUMA systems, the scheduler prefers to run a process on the
processor that owns the memory used by that process, reducing
remote memory access latency.

[Conclusion]
Distributed scheduling with work-stealing and NUMA awareness is
the preferred approach for scalable multiprocessor systems.
```

---

# 🔵 TOPIC 5: Gang Scheduling 🔥

## Gang Scheduling Kya Hai?

> **Gang Scheduling** = Related threads ko ek saath ek hi time pe MULTIPLE processors pe schedule karo!

> 👫 **Analogy:** Ek "gang" (group) ke sab log ek saath party mein jaate hain — koi alag time pe nahi jaata!
> Waise hi — ek parallel program ke sab threads ek saath CPU pe aate hain!

## Problem Bina Gang Scheduling Ke:

```
Parallel program with 4 threads: T1, T2, T3, T4
(They communicate with each other frequently!)

Without Gang Scheduling:
Time 1: [T1 on CPU1] [T3 on CPU2]   ← T2, T4 not running!
Time 2: [T2 on CPU1] [T4 on CPU2]   ← T1, T3 not running!

Problem:
T1 wants to communicate with T2 — but T2 is NOT running!
T1 must WAIT (blocking on communication)
While T1 waits → CPU1 is WASTED!
Result: Lots of SPINNING/WAITING → Performance loss!
```

## Solution — Gang Scheduling:

```
With Gang Scheduling:
Time 1: [T1 on CPU1] [T2 on CPU2] [T3 on CPU3] [T4 on CPU4]
         All 4 threads run SIMULTANEOUSLY!
         
T1 can communicate with T2 directly — T2 is running!
No waiting!
Result: Efficient communication, no spinning!
```

## How Gang Scheduling Works:

```
Step 1: Identify a "gang" = all threads of one parallel process
Step 2: When scheduling, find enough FREE CPUs for entire gang
Step 3: Schedule ALL threads of gang simultaneously
Step 4: When gang's time quantum ends, ALL threads are swapped out together
Step 5: Next gang gets scheduled (all its threads together)
```

```
Time Slot 1: [Gang A: T1|T2|T3|T4] on [CPU1|CPU2|CPU3|CPU4]
Time Slot 2: [Gang B: T1|T2|T3|T4] on [CPU1|CPU2|CPU3|CPU4]
Time Slot 3: [Gang A again]
Time Slot 4: [Gang C: T1|T2] on [CPU1|CPU2] (CPU3,CPU4 idle if no other gang)
```

## Advantages of Gang Scheduling:

1. **Eliminates Spin Waiting:** Communicating threads run simultaneously → No waiting
2. **Better Communication Performance:** All threads ready when others need to communicate
3. **Reduced Context Switch Overhead:** Whole gang switches together
4. **Predictable Performance:** Consistent timing for parallel programs

## Disadvantages:

1. **Fragmentation:** Gang size may not match number of available CPUs → Some CPUs idle
2. **Inflexible:** Small gangs waste CPU resources
3. **Starvation:** Large gangs may wait long for enough free CPUs
4. **Complexity:** Harder to implement than simple per-CPU scheduling

## Gang Scheduling vs Space Sharing:

| Feature | Gang Scheduling | Space Sharing |
|---------|----------------|---------------|
| CPU allocation | All threads at same time | Each thread gets its own CPU permanently |
| Communication | Excellent (all running) | Excellent (dedicated CPU) |
| CPU utilization | May have idle CPUs | Better utilization |
| Context switching | Yes (time-shared gangs) | No switching |
| Use case | Time-shared systems | Large parallel jobs |

## 🧠 Gang Scheduling Trick

> **"Gang = Group jaisa"** — whole group together or not at all!
>
> **Without Gang:** Thread A calls Thread B — B not running → A spins → CPU waste!
> **With Gang:** A calls B — B is running → Instant response → Efficient!
>
> **Key exam point:** Gang scheduling eliminates **spin-waiting in parallel programs!**

---

## ✍️ EXAM TEMPLATE — Gang Scheduling

```
📝 Answer Template:

[Definition — 2 lines]
Gang scheduling is a multiprocessor scheduling technique where all
threads of a parallel program (a "gang") are scheduled simultaneously
on multiple processors at the same time slot.

[Problem it Solves]
In a parallel program, threads frequently communicate with each other.
Without gang scheduling, some threads may be scheduled while their
communicating partners are not running, causing:
• Spin-waiting (busy waiting for partner to respond)
• CPU cycles wasted while waiting
• Poor parallel program performance

[How it Works]
1. All threads of a parallel program form a "gang".
2. The scheduler waits until enough processors are free for the entire gang.
3. All threads of the gang are dispatched simultaneously.
4. When the time quantum ends, all threads are swapped out together.
5. The next gang gets scheduled.

[Diagram — Time Slots]
Time Slot 1: Gang A [T1|T2|T3|T4] → [CPU1|CPU2|CPU3|CPU4]
Time Slot 2: Gang B [T1|T2|T3|T4] → [CPU1|CPU2|CPU3|CPU4]

[Advantages]
1. Eliminates spin-waiting — all communicating threads run together.
2. Reduces synchronization overhead.
3. Predictable and consistent parallel execution.

[Disadvantages]
1. CPU fragmentation — gang size may not match available CPUs.
2. Larger gangs may suffer starvation.
3. More complex implementation.

[Conclusion]
Gang scheduling significantly improves the performance of parallel
programs with frequent inter-thread communication by ensuring all
threads run simultaneously.
```

---

# 🔵 TOPIC 6: Communication Between Processes — IPC

## IPC Kya Hai?

> **IPC (Inter-Process Communication)** = Alag-alag processes aapas mein data kaise share karte hain?

> 🏘️ **Analogy:** Alag-alag ghar mein rehne wale logon ka communication
> - Message Box = Letter/WhatsApp message
> - Shared Memory = Ek common notice board

## Two Main IPC Mechanisms:

---

## IPC Method 1: Message Passing (Message Boxes) 📬

### Message Passing Kya Hai?

> Processes data directly **bhejte aur receive karte hain** — shared memory nahi hoti!

> 📱 **Analogy:** WhatsApp — tu message bhejta hai, doosra receive karta hai. Ek hi phone nahi chalate dono!

### Message Box / Mailbox:

```
Process A                Message Box              Process B
─────────                (OS manages)             ─────────
send(msg)  ──────────► [msg1][msg2][msg3] ──────► receive(msg)
                        ↑ Queue/Buffer ↑
```

**Mailbox = OS-managed buffer** jahan messages store hote hain sender aur receiver ke beech!

### Direct vs Indirect Communication:

```
DIRECT:
Process A sends directly to Process B:
send(B, message)
receive(A, message)
→ Link directly between A and B

INDIRECT (via Mailbox):
Process A sends to Mailbox M:
send(M, message)
Process B receives from Mailbox M:
receive(M, message)
→ Decoupled! A doesn't need to know B directly
```

### Synchronous vs Asynchronous:

| Type | Sender | Receiver |
|------|--------|---------|
| **Blocking send** | Waits until receiver gets message | — |
| **Non-blocking send** | Sends and continues | — |
| **Blocking receive** | — | Waits until message arrives |
| **Non-blocking receive** | — | Returns immediately (message or null) |

### Message Passing — POSIX API:

```c
// Create a message queue
mqd_t mq = mq_open("/myqueue", O_CREAT | O_RDWR, 0644, &attr);

// Send message (Process A):
mq_send(mq, "Hello Process B!", 16, 0);

// Receive message (Process B):
char buffer[100];
mq_receive(mq, buffer, 100, NULL);
printf("Received: %s\n", buffer);  // Prints: Hello Process B!

// Cleanup
mq_close(mq);
mq_unlink("/myqueue");
```

### Advantages of Message Passing:
1. **No shared memory needed** — works across machines (distributed systems)
2. **No synchronization issues** — OS handles message queuing
3. **Natural for distributed systems** (MPI uses message passing)
4. **Process isolation** — processes don't share address space

### Disadvantages:
1. **Slower** than shared memory (OS overhead for each message)
2. **Copying overhead** — data copied multiple times
3. **Limited message size** — queue/buffer has size limits

---

## IPC Method 2: Shared Memory 🗒️

### Shared Memory Kya Hai?

> **Multiple processes ek hi physical memory region share karte hain!**

> 📋 **Analogy:** Office ka common whiteboard — koi bhi likh sakta hai, koi bhi padh sakta hai. Fast! But coordination chahiye.

```
Process A's Virtual Memory:    Process B's Virtual Memory:
┌─────────────────────┐        ┌─────────────────────┐
│ Private memory A    │        │ Private memory B    │
├─────────────────────┤        ├─────────────────────┤
│  SHARED REGION      │◄──────►│  SHARED REGION      │
│  (same physical     │        │  (same physical     │
│   memory!)          │        │   memory!)          │
└─────────────────────┘        └─────────────────────┘
         ↓                              ↓
         └──────────► Physical Memory ◄─┘
                      (One physical copy!)
```

### POSIX Shared Memory API:

```c
// Process A (Creator):
// Step 1: Create shared memory
int shm_fd = shm_open("/mysharedmem", O_CREAT | O_RDWR, 0644);

// Step 2: Set size
ftruncate(shm_fd, 1024);  // 1KB shared memory

// Step 3: Map into address space
char *ptr = mmap(NULL, 1024, PROT_READ|PROT_WRITE, MAP_SHARED, shm_fd, 0);

// Write to shared memory:
sprintf(ptr, "Hello from Process A!");

// ─────────────────────────────────────────

// Process B (User):
// Step 1: Open existing shared memory
int shm_fd = shm_open("/mysharedmem", O_RDWR, 0644);

// Step 2: Map into address space
char *ptr = mmap(NULL, 1024, PROT_READ|PROT_WRITE, MAP_SHARED, shm_fd, 0);

// Read from shared memory:
printf("Got: %s\n", ptr);  // Prints: Hello from Process A!
```

### Advantages of Shared Memory:
1. **Fastest IPC** — No copying, direct memory access!
2. **Large data** — Can share gigabytes easily
3. **Low overhead** — After setup, just read/write like normal memory

### Disadvantages:
1. **Synchronization required** — Race conditions! Must use semaphores/mutexes
2. **Works only on same machine** — Cannot use across network
3. **Security risk** — All processes in shared memory can see each other's data

---

## 📊 Message Passing vs Shared Memory

| Feature | Message Passing | Shared Memory |
|---------|----------------|---------------|
| Speed | Slower (OS overhead) | Faster (direct access) |
| Data Copy | Yes (copied to buffer) | No (direct memory) |
| Synchronization | Handled by OS | Programmer must handle |
| Distance | Works across machines | Same machine only |
| Ease | Easier to use | More complex |
| Data Size | Limited (queue size) | Large (GBs possible) |
| Used by | MPI | OpenMP, pthreads |

## 🧠 IPC Trick

> **"Message = Post Office"** 📬 (Slow, OS handles, works everywhere)
> **"Shared Memory = Whiteboard"** 🗒️ (Fast, programmer handles sync, same room only)

---

## ✍️ EXAM TEMPLATE — IPC (Message Passing + Shared Memory)

```
📝 Answer Template:

[Introduction]
Inter-Process Communication (IPC) refers to mechanisms that allow
processes to communicate and share data. The two main IPC methods
are message passing and shared memory.

[Method 1: Message Passing]
In message passing, processes communicate by sending and receiving
messages through an OS-managed channel or mailbox.

Types:
• Direct: Process names recipient explicitly (send/receive to/from process)
• Indirect: Messages sent to/received from a mailbox (decoupled)

Variants:
• Blocking (synchronous): Sender waits until message received
• Non-blocking (asynchronous): Sender continues immediately

Advantages: Works across machines, OS handles sync, process isolation
Disadvantages: Slow (copying overhead), limited message size

[Method 2: Shared Memory]
A region of physical memory is mapped into the address space of
multiple processes. They communicate by reading/writing this shared region.

Steps:
1. Create shared memory region (shm_open)
2. Map it into each process's address space (mmap)
3. Synchronize access using semaphores/mutex

Advantages: Fastest IPC, no copying, handles large data
Disadvantages: Synchronization required, same machine only

[Comparison Table]
[Add comparison table]

[Conclusion]
Message passing is simple and works across networks (MPI uses it),
while shared memory is faster and preferred for same-machine
high-performance communication (OpenMP uses it).
```

---

# 🔵 TOPIC 7: Sharing Issues and Synchronization

## Synchronization Issues in Multiprocessing OS

In a multiprocessing system, processes/threads share resources — OS must provide mechanisms to manage this safely.

---

## Mechanism 1: Distributed Semaphores 🔢

### Normal Semaphore (Single System):
```
Semaphore S = 1

wait(S):   S--; if S < 0 → block
signal(S): S++; if S ≤ 0 → wake one blocked process
```

### Distributed Semaphore — Across Multiple Machines!

> **Problem:** Normal semaphores work only on ONE machine (shared memory)
> **Distributed systems** pe kai alag machines hain — shared memory nahi!

> 🌐 **Analogy:** Normal semaphore = School ki ek bell (sab building mein sunti hai)
> Distributed semaphore = Network announcement (different buildings mein message jaata hai)

### How Distributed Semaphores Work:

**Method 1: Centralized Server Approach**
```
All processes contact a CENTRAL SEMAPHORE SERVER:

Process A: "Wait(S)" → Send request to Server
Server: S-- → If S ≥ 0: Reply "OK, proceed" to A
              If S < 0: A added to wait queue
Process B: "Signal(S)" → Server S++ → Wake up next from queue
```

**Problem:** Server = Single point of failure, bottleneck

---

**Method 2: Token-Based Distributed Semaphore**
```
A TOKEN represents the semaphore:
Who has the token → Has the semaphore!

Process A wants semaphore → Requests token → Gets it → Enters CS
Process B wants semaphore → Requests token → Waits
Process A done → Passes token to B
```

**Used in:** Token Ring networks, distributed file systems

---

**Method 3: Ricart-Agrawala Algorithm**
```
Process wanting critical section:
1. Broadcast REQUEST to all processes (with timestamp)
2. Wait for OK from ALL processes
3. Enter Critical Section
4. Send OK to anyone waiting

Process receiving REQUEST:
- If not in CS and not waiting: Send OK immediately
- If in CS: Queue the request (send OK when done)
- If waiting with earlier timestamp: Send OK; with later timestamp: Queue
```

---

## Mechanism 2: Monitors 🏛️

### Monitor Kya Hai?

> **Monitor** = A HIGH-LEVEL synchronization construct that combines:
> - Shared data (variables)
> - Procedures to access that data
> - Automatic mutual exclusion

> 🏛️ **Analogy:** Bank ka VIP room — sirf ek customer andar allowed hai, aur room mein sab tools available hain. Bank manager (monitor) ensure karta hai ek time pe ek!

### Monitor Structure:

```
┌─────────────────────────────────────────┐
│                MONITOR                  │
│                                         │
│  Shared Variables:                      │
│  int balance = 0;                       │
│  int count = 0;                         │
│                                         │
│  Procedures (auto-mutual exclusion):    │
│  ┌─────────────────────────────────┐    │
│  │ deposit(amount) { ... }         │    │
│  │ withdraw(amount) { ... }        │    │
│  │ getBalance() { ... }            │    │
│  └─────────────────────────────────┘    │
│                                         │
│  Condition Variables:                   │
│  condition not_empty;                   │
│  condition not_full;                    │
│                                         │
└─────────────────────────────────────────┘
   │ Only ONE process inside at a time!
```

### Monitor in Java (synchronized):

```java
// Java Monitor using synchronized:
class BankAccount {
    private int balance = 0;
    
    // Synchronized = Monitor procedure!
    // Only ONE thread can execute at a time!
    public synchronized void deposit(int amount) {
        balance += amount;
        notifyAll();    // Wake up waiting threads
    }
    
    public synchronized void withdraw(int amount) {
        while (balance < amount) {
            try {
                wait();    // Wait (condition variable)
            } catch (InterruptedException e) {}
        }
        balance -= amount;
    }
    
    public synchronized int getBalance() {
        return balance;
    }
}
```

### Monitor vs Semaphore:

| Feature | Semaphore | Monitor |
|---------|-----------|---------|
| Abstraction level | Low (P/V operations) | High (procedures) |
| Mutual exclusion | Programmer must add | Automatic! |
| Ease of use | Error-prone | Easier and safer |
| Condition variables | Limited | Built-in |
| Language support | C (manual) | Java, C# (built-in) |

### Monitor Operations:

```
wait(condition):   
  - Release monitor lock
  - Block process on this condition
  - When signaled, re-acquire lock before continuing

signal(condition):
  - Wake up one process waiting on condition
  - That process will re-acquire monitor lock

broadcast(condition):
  - Wake up ALL processes waiting on condition
```

---

## Mechanism 3: Spin Locks 🔄

### Spin Lock Kya Hai?

> **Spin Lock** = Lock jo milne tak CPU sirf CHECK karta rehta hai — koi sleep nahi!

> 🔄 **Analogy:** Busy waiting = Store ke bahar khada rehna aur har second door check karna ki "khula?" — chalta rehta hai jab tak khul na jaaye!

```c
// Spin Lock implementation:
volatile int lock = 0;   // 0 = free, 1 = locked

// Acquire (spin):
void spin_lock(volatile int *lock) {
    while (__sync_lock_test_and_set(lock, 1)) {
        // Keep spinning! CPU busy waiting
        while (*lock) { }    // Read without bus transaction (TTS-style)
    }
}

// Release:
void spin_unlock(volatile int *lock) {
    __sync_lock_release(lock);    // Atomic write 0
}
```

### When to Use Spin Locks:

```
Spin Lock GOOD when:
✅ Lock held for VERY SHORT time (few microseconds)
✅ Few processors competing
✅ Context switch cost > Expected wait time

Mutex (Sleep Lock) GOOD when:
✅ Lock held for LONG time
✅ Many processors competing
✅ Don't want to waste CPU cycles
```

### Spin Lock vs Mutex:

| Feature | Spin Lock | Mutex (Sleep Lock) |
|---------|-----------|-------------------|
| Waiting | Busy-wait (wastes CPU) | Sleep (yields CPU) |
| Best for | Short critical sections | Long critical sections |
| Context switch | None | Yes (expensive) |
| Overhead | Low if short wait | High (sleep/wake overhead) |
| CPU usage | 100% while waiting | 0% while sleeping |
| Used in | OS kernel, real-time | User-space programs |

### Types of Spin Locks (already in Unit 3):
- **TAS (Test-and-Set)** — Simple but high bus traffic
- **TTS (Test-and-Test-and-Set)** — Better, less bus traffic
- **Ticket Lock** — Fair (FIFO)
- **Array Lock** — Scalable, per-thread slot

---

## 📊 Synchronization Mechanisms Summary

| Mechanism | Level | Mutual Exclusion | Condition | Best For |
|-----------|-------|-----------------|-----------|----------|
| Spin Lock | Low | Manual | No | Short waits, OS kernel |
| Mutex | Low-Mid | Manual | No | General purpose |
| Semaphore | Mid | Optional | Limited | Resource counting |
| Monitor | High | Automatic | Yes (built-in) | OOP languages |
| Dist. Semaphore | Distributed | Distributed | Via message | Across machines |

## 🧠 Synchronization Tricks

> **"Spin = Chakkar maaro"** 🔄 (busy wait)
> **"Monitor = VIP room"** 🏛️ (auto mutual exclusion)
> **"Distributed Semaphore = Token pass karo"** 🎟️

---

## ✍️ EXAM TEMPLATE — Synchronization (Distributed Semaphore + Monitor + Spin Lock)

```
📝 Answer Template:

[Introduction]
In multiprocessor systems, sharing resources between processes requires
synchronization mechanisms to ensure correct, race-condition-free execution.

[1. Distributed Semaphores]
A distributed semaphore extends the concept of semaphores to distributed
systems where processes run on different machines without shared memory.

Implementation approaches:
• Centralized Server: All wait/signal requests go through one server.
• Token-Based: Process holding token has the semaphore.
• Ricart-Agrawala: Broadcast-based mutual exclusion using timestamps.

Challenge: Maintaining consistency across machines without shared memory.

[2. Monitors]
A monitor is a high-level synchronization construct that encapsulates
shared variables and procedures with automatic mutual exclusion.
Only one process can execute inside a monitor at any time.

Key features:
• Automatic mutual exclusion (no explicit lock/unlock needed)
• Condition variables for waiting/signaling (wait, signal, broadcast)
• Prevents errors caused by incorrect use of semaphores

Example: Java's 'synchronized' keyword implements monitor semantics.

[3. Spin Locks]
A spin lock is a low-level lock where a process repeatedly checks
the lock status (busy-waits) instead of sleeping.

Types: TAS → TTS → Ticket Lock → Array Lock (improving efficiency)

When to use:
• Short critical sections (microseconds)
• Context switch cost > Expected wait time
• OS kernel code

[Comparison Table]
[Add mechanism comparison table]

[Conclusion]
Different synchronization mechanisms suit different scenarios:
spin locks for short kernel-level CS, monitors for high-level OOP,
and distributed semaphores for cross-machine coordination.
```

---

# 🔵 TOPIC 8: Implementation on Multi-cores — OpenMP & MPI

## OpenMP on Multi-core — OS Perspective

### How OS Supports OpenMP:

```
Programmer writes: #pragma omp parallel for

Compiler generates: Thread creation code
Runtime library:   Creates OS threads (pthreads)
OS:               Schedules threads on multiple cores
Hardware:         Cores execute threads in parallel
```

### OpenMP Thread Lifecycle (OS View):

```
Main thread (T0) running:

#pragma omp parallel → OS creates T1, T2, T3 (fork)
                        All 4 threads run on 4 cores!
                        
End of parallel region → T1, T2, T3 destroyed (join)
                          Only T0 continues
```

### OpenMP — Complete Exam-Ready Example:

```c
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>

// Matrix addition in parallel:
void matrix_add(int N, double A[][N], double B[][N], double C[][N]) {
    
    #pragma omp parallel for collapse(2) schedule(static)
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            C[i][j] = A[i][j] + B[i][j];
        }
    }
    // collapse(2) treats both loops as one for better distribution
}

// Parallel reduction:
double parallel_sum(double *arr, int N) {
    double sum = 0.0;
    
    #pragma omp parallel for reduction(+:sum) schedule(dynamic, 100)
    for (int i = 0; i < N; i++) {
        sum += arr[i];
    }
    return sum;
}

int main() {
    omp_set_num_threads(4);    // Use 4 threads
    printf("Max threads: %d\n", omp_get_max_threads());
    
    // Thread info inside parallel region:
    #pragma omp parallel
    {
        int tid = omp_get_thread_num();
        int ntid = omp_get_num_threads();
        printf("Thread %d of %d\n", tid, ntid);
    }
    return 0;
}
```

### OpenMP — Important Directives for Exam:

| Directive | Purpose | Example |
|-----------|---------|---------|
| `parallel` | Create parallel region | `#pragma omp parallel` |
| `for` | Parallelize loop | `#pragma omp parallel for` |
| `sections` | Divide tasks | `#pragma omp sections` |
| `single` | One thread only | `#pragma omp single` |
| `critical` | Critical section | `#pragma omp critical` |
| `atomic` | Atomic operation | `#pragma omp atomic` |
| `barrier` | Sync all threads | `#pragma omp barrier` |
| `reduction` | Combine results | `reduction(+:sum)` |
| `collapse(n)` | Collapse n loops | `collapse(2)` |
| `nowait` | Skip barrier | `#pragma omp for nowait` |
| `schedule` | Work distribution | `schedule(dynamic)` |

### OpenMP Memory Model:

```
SHARED by default in parallel region:
- Variables declared outside
- Heap memory (malloc)
- Static variables

PRIVATE by default:
- Loop iteration variables
- Stack variables inside parallel region

Override with clauses:
private(x)      → Each thread gets own copy
shared(x)       → All threads share
firstprivate(x) → Private + initialized from original
lastprivate(x)  → Private + last value saved
```

---

## MPI on Multi-core (and Distributed) — OS Perspective

### How MPI Works at OS Level:

```
MPI Program Launch:
mpirun -np 4 ./myprogram

OS creates 4 SEPARATE PROCESSES (not threads!):
Process 0 (rank 0) → CPU 0
Process 1 (rank 1) → CPU 1
Process 2 (rank 2) → CPU 2
Process 3 (rank 3) → CPU 3

Each process has its OWN address space!
Communication via network or shared memory
```

### MPI Communication Types:

**Point-to-Point:**
```c
MPI_Send(data, count, MPI_INT, dest, tag, MPI_COMM_WORLD);
MPI_Recv(data, count, MPI_INT, src, tag, MPI_COMM_WORLD, &status);
```

**Collective Operations:**
```c
MPI_Bcast(&data, 1, MPI_INT, 0, MPI_COMM_WORLD);    // One→All broadcast
MPI_Scatter(sendbuf, n, MPI_INT, recvbuf, n, MPI_INT, 0, MPI_COMM_WORLD); // Distribute
MPI_Gather(sendbuf, n, MPI_INT, recvbuf, n, MPI_INT, 0, MPI_COMM_WORLD);  // Collect
MPI_Reduce(sendbuf, recvbuf, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);     // Reduce
MPI_Allreduce(sendbuf, recvbuf, 1, MPI_INT, MPI_SUM, MPI_COMM_WORLD);     // All reduce
MPI_Barrier(MPI_COMM_WORLD);                          // Synchronize all
```

### Complete MPI Example — Parallel Sum:

```c
#include <mpi.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
    int rank, size;
    int N = 100;
    int local_sum = 0, global_sum = 0;
    
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);    // My process ID
    MPI_Comm_size(MPI_COMM_WORLD, &size);    // Total processes
    
    // Each process computes partial sum
    int chunk = N / size;
    int start = rank * chunk;
    int end = start + chunk;
    
    for (int i = start; i < end; i++) {
        local_sum += i;    // Each process adds its portion
    }
    
    // Reduce: Sum all local_sums → global_sum at rank 0
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_INT,
               MPI_SUM, 0, MPI_COMM_WORLD);
    
    if (rank == 0) {
        printf("Total sum = %d\n", global_sum);
    }
    
    MPI_Finalize();
    return 0;
}
```

### MPI Process Topology:

```
Linear (Ring):     P0 ↔ P1 ↔ P2 ↔ P3 ↔ P0
2D Grid:           P0─P1
                   │  │
                   P2─P3
3D Cube:           For 3D problems (weather simulation, etc.)
```

---

## OpenMP vs MPI — Complete Comparison

| Feature | OpenMP | MPI |
|---------|--------|-----|
| **Model** | Shared Memory | Message Passing |
| **Processes/Threads** | Threads | Processes |
| **Memory** | Shared address space | Private per process |
| **Communication** | Shared variables | Send/Receive messages |
| **Scope** | Single machine (multi-core) | Multiple machines (cluster) |
| **Sync Mechanism** | Directives, barriers | MPI_Barrier, blocking calls |
| **Ease of use** | Easier (add directives) | More complex |
| **Scalability** | Limited (one machine) | Very high (thousands) |
| **Race conditions** | Possible | Not possible |
| **Data sharing** | Automatic | Explicit (programmer) |
| **Compilation** | `gcc -fopenmp` | `mpicc` |
| **Execution** | `./program` | `mpirun -np 4 ./program` |
| **Best for** | Multi-core desktop/server | Supercomputers, clusters |
| **Used for** | Parallel loops, tasks | Distributed computing, HPC |

## Hybrid OpenMP + MPI:

> Modern HPC uses BOTH together!

```
MPI handles inter-node communication:
Node 1 (8 cores): MPI Process 0 + OpenMP (8 threads)
Node 2 (8 cores): MPI Process 1 + OpenMP (8 threads)
Node 3 (8 cores): MPI Process 2 + OpenMP (8 threads)

→ MPI across nodes, OpenMP within each node!
→ Best of both worlds!
```

```c
// Hybrid MPI+OpenMP:
int main(int argc, char* argv[]) {
    int provided;
    MPI_Init_thread(&argc, &argv, MPI_THREAD_FUNNELED, &provided);
    
    int rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    
    // OpenMP within each MPI process:
    #pragma omp parallel
    {
        printf("MPI rank %d, OMP thread %d\n",
               rank, omp_get_thread_num());
    }
    
    MPI_Finalize();
}
```

## 🧠 OpenMP vs MPI Trick

> **"OpenMP = Ghar ke andar sab share karo"** 🏠
> (Same house/machine, shared memory, threads)
>
> **"MPI = Alag-alag ghar pe phone se baat karo"** 📞
> (Different machines, message passing, processes)
>
> **"Hybrid = Ek neighbourhood — ghar ke andar share, bahar call"** 🏘️

---

## ✍️ EXAM TEMPLATE — OpenMP + MPI Implementation

```
📝 Answer Template:

[Introduction]
Multi-core systems are programmed using two main parallel programming
frameworks: OpenMP for shared-memory systems and MPI for distributed
memory systems.

[OpenMP Implementation]
OpenMP (Open Multi-Processing) uses compiler directives to parallelize
code on shared-memory multi-core processors.

Key directives:
• #pragma omp parallel: Creates thread team
• #pragma omp parallel for: Distributes loop iterations
• #pragma omp critical: Protects critical section
• #pragma omp reduction: Safely accumulates values
• #pragma omp barrier: Synchronizes all threads

Compilation: gcc -fopenmp program.c -o program
Execution: ./program (OMP_NUM_THREADS=4)

OS role: Creates kernel threads, schedules on multiple cores.

[MPI Implementation]
MPI (Message Passing Interface) creates separate processes that
communicate via explicit send/receive messages.

Key functions:
• MPI_Init/Finalize: Initialize/cleanup MPI
• MPI_Comm_rank/size: Get process ID and count
• MPI_Send/Recv: Point-to-point communication
• MPI_Bcast: Broadcast one to all
• MPI_Reduce/Allreduce: Combine values
• MPI_Barrier: Synchronize all processes

Compilation: mpicc program.c -o program
Execution: mpirun -np 4 ./program

OS role: Creates separate processes, handles message routing.

[Comparison Table]
[Add OpenMP vs MPI comparison table]

[Hybrid Approach]
Modern HPC systems combine both: MPI between nodes (machines)
and OpenMP within each node (shared-memory multi-core).

[Conclusion]
OpenMP is simpler and preferred for single-machine parallelism,
MPI is essential for distributed cluster computing, and hybrid
MPI+OpenMP provides the best performance on modern supercomputers.
```

---

# 📊 MASTER SUMMARY TABLE — Unit 5

| Topic | Key Point | Trick/Analogy |
|-------|-----------|---------------|
| Pre-emptive OS | OS can forcibly take CPU away | Police wali OS 👮 |
| Context Switch | Save/restore process state | Game save/load |
| Why Pre-emptive | Fairness, Real-time, Deadlock, Load balance | "FRLPCD" |
| FCFS | First come first served | Queue at bank |
| SJF | Shortest job first | Shortest line at checkout |
| Round Robin | Equal time quantum, rotate | Chakkar system 🎡 |
| Priority | High priority first | VIP entry 🎭 |
| MLFQ | Dynamic priority change | Escalator — slow → lower floor |
| Threads (ULT) | User manages, OS sees one process | Hidden workers |
| Threads (KLT) | OS manages each thread | OS knows all workers |
| Distributed Scheduler | Per-CPU queues + work stealing | Steal when idle 😄 |
| Work Stealing | Idle CPU takes from busy CPU | "Chori karo agar khali ho" |
| Gang Scheduling | All threads of parallel prog run together | Gang jaata hai saath |
| Why Gang Scheduling | Eliminate spin-waiting | No more spinning! |
| Message Passing | Send/receive messages via OS | WhatsApp 📱 |
| Shared Memory | Common memory region | Whiteboard 🗒️ |
| Distributed Semaphore | Semaphore across machines | Token pass 🎟️ |
| Monitor | High-level auto mutual exclusion | VIP room 🏛️ |
| Spin Lock | Busy wait for lock | Chakkar maaro 🔄 |
| OpenMP | Shared memory, directives, threads | Ghar ke andar share 🏠 |
| MPI | Distributed, message passing, processes | Alag ghar, phone se 📞 |
| Hybrid MPI+OpenMP | Both combined | Neighbourhood 🏘️ |

---

# 🎯 EXAM-DAY TIPS FOR UNIT 5

## Top Questions Jo Aayenge:

1. Why is a pre-emptive OS needed for multiprocessing? Explain.
2. Explain gang scheduling with advantages and disadvantages.
3. Compare message passing and shared memory for IPC.
4. Explain monitors with example. Compare with semaphores.
5. What are spin locks? Compare with mutex.
6. Explain distributed semaphores — how do they work?
7. Compare OpenMP and MPI for multi-core implementation.
8. Explain scheduling algorithms — FCFS, RR, Priority.
9. What is a distributed scheduler? Explain work stealing.
10. Explain context switching — when and why it occurs.

## Marks Strategy for Unit 5:

```
Gang Scheduling → 5 mark question almost certain
  → Definition + Why needed + How works + Diagram + Advantages/Disadvantages

IPC Comparison → Could be 5 or 10 marks
  → Both methods + code snippets + comparison table

OpenMP vs MPI → 5 marks very likely (teacher specifically mentioned)
  → Table + Key directives/functions + Example code

Monitor → 5 marks possible
  → Definition + Java synchronized example + vs Semaphore table
```

---

## 💡 FINAL TIPS FROM YOUR TEACHER

> **Unit 5 ke liye bhai/behen:**
>
> 1. **Gang Scheduling** = Ye topic Unit 5 ka HERO hai — poori explanation ke saath yaad karo
> 2. **Pre-emptive OS reasons** = 5 reasons yaad karo "FRLPCD"
> 3. **IPC** = Message Passing vs Shared Memory table banana — instant marks
> 4. **Monitor** = Java synchronized example = easy marks
> 5. **Spin Lock vs Mutex** = Table banana — kab use karo
> 6. **OpenMP vs MPI** = Ye comparison table ZAROOR banana — almost certain question
> 7. **Distributed Semaphore** = Concept clear karo — token-based yaad karo
>
> **Golden Tip:** Unit 5 mein har topic ke 2 cheezein yaad rakho:
> 1. Kya hai (definition)
> 2. Kab use karo / kyon better hai (use case)
> Yahi 2 points = Full marks! 🎯

---

*Prepared with ❤️ by Your Teacher | PCA Unit 5 | All 8 Topics Covered*
*"Paanchon Units Complete! Ab exam mein parallel speed se likhna!" ⚡*
*"OS manages the machine — YOU manage the exam!" 😄 Good Luck! 🎉*
