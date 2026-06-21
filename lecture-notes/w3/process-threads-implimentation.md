# Processes and Threads Implimentation

## MIPS R3000
- Widely used through the 90s
- Load/Store ONLY instructions on memory
- All other operations: register to register
- 32-bit instructions; BUT immediate values can only take up 16-bits
 
---

## MIPS Registers
- R0 -> R31: "general purpose"
- HI/LO: used for multiplication and division operands EXCLUSIVELY
- PC: Program counter (just like Instruction Pointer)
  
#### Branching and Jumping
- When a jump instruction is called, it doesn't jump immediately
- instead, it executes the next instruction before jumping
- The compiler protects the jump destination by ensuring the following action does not interfere
- If it cannot find anything useful to place there, it inserts a NOP; a non-instruction.
 
#### Jump and Link
- Wrapper jump instruction
- Saves return address at IP + 8 -> stores in R31
  
#### Register Table
| Register Number | Name/Alias | Used For |
| :--- | :--- | :--- |
| `r0` | `zero` | Always returns 0 (hardwired to zero) |
| `r1` | `at` | Assembler temporary; reserved strictly for use by the assembler |
| `r2 - r3` | `v0 - v1` | Value (except floating-point) returned by a subroutine |
| `r4 - r7` | `a0 - a3` | Arguments; the first four parameters for a subroutine |
| `r8 - r15`, `r24 - r25` | `t0 - t7`, `t8 - t9` | Temporaries; subroutines may freely use these without saving their previous values |
| `r16 - r23` | `s0 - s7` | Subroutine "register variables"; if a subroutine writes to one, it must save the old value and restore it before exiting so the calling routine sees the values preserved |
| `r26 - r27` | `k0 - k1` | Reserved for use by the interrupt/trap handler; these may change unexpectedly ("under your feet") |
| `r28` | `gp` | Global pointer; runtime systems maintain this to give easy access to static or external variables |
| `r29` | `sp` | Stack pointer |
| `r30` | `s8 / fp` | 9th register variable; subroutines that need one can use this as a frame pointer |
| `r31` | `ra` | The link register used to hold the return address for jump-and-link (JAL) instructions |

---

## Stack Frames
- Used to store local variables, return address, frame pointer etc. for each new function
- Stack frames begin at a high address, and grow smaller; eg. 1000 -> 0.
- Args 1-4 have space reserved; further arguments must be manually pushed to the stack.
 
 ---
 
## Processes
- Each process consists of three segments
    - Text: contains the code
- Data: Global variables
    - Stack: Stack frames + local variables
- The stack grows down; the heap grows up.
 
### Multi-Threaded Processes
- threads are allocated space dynamically: on the heap
- Each thread gets its own unique stack, but the heap is global
    - IMPORTANTLY that makes variables on the stack SAFE, but the heap CRITICAL

 ### User-Process vs. Kernel-Process
- each user processes has an associated kernel process
- use the 'syscall barrier' to protect the kernel process from malicious user processes

#### User-Process Specific
- Each act independently, and scheduled by the OS; no concurrency risk

#### Kernel-Process Specific
- kernel process is the process which executes syscalls requested by the user
- kernel memory shared between all processes; potential concurrency issues
- nature of interupts important; whether syscalls can be interupted or not

---

## Threads

### Process Item Table
| Per Process Item | Per Thread Item |
| :----- | :----- |
| Address space | Program counter |
| Global variables | Registers |
| Open files | Stack |
| Child processes | State |
| Pending alarms | |
| Signals and signal handlers | |
| Accounting information | |

### Mutli-Threaded Scheduling
- OS has no knowledge of the implimentation of threads; only sees one process
- Each process schedules/manages its own threads independently
    - Introduces inter-process concurrency issues
    - Allows for multi-threading efficiency
- Implimented with runtime support libraries eg. Mutex, CVs, Semphores

#### Pros
- Much faster than kernel level; no need to syscall
- Dispatcher algorithms can be implimentation specific; both OS and application specific
- Can easily support many threads; kernel memory much more constrained

### Cons
- No interupts; must yield manually 
    - Thus, required a cooportative design, lest a single threat monopolise available CPU time
- Does not take advantage of multiple CPUs; to the OS, it is still single process

### Hybrid Model?
- Enforces 'hardware timer' limit; pikmin interrupts
- Every so often the OS forceably takes over and decides if running threats should yeild
- Software-level interrups

### Kernel-Provided Threads
- Threads are managed by the kernel, in the OS layer
- Centrally allocated; true paralellism
- Managed using PCBs (Process Control Blocks), who create associated TCBs (Thread Control Blocks)
- Ensures interupts are NOT system-wide; can be thread localised
- However computationally expensive; threads must be managed using syscalls using limited kernel memory

---

## Multiprogramming Implimentation

### Context Switch
- A switch betwen threads
    - involving the saving and restoring of state
- A switch between processes
    - involving the saving and restoring of BOTH the process itself and its associated threads

#### Skeleton Layout
1. Hardware stack's program counter, etc.
2. Hardware loads new program counter from interrupt vector
3. Assembly language procedure saves registers
4. Assembly language procesure sets up fresh stack
5. C interrupt service run (reads and buffers input)
6. Scheduler decides which process is to run next
7. C procedure returns to assembly code
8. Assembly lanuage procedure starts new process

#### Switch Occurences
- Can happen any time the OS takes over a process:
    - For a system call: mandatory if syscall blocks
    - On an exemption: mandatory if process cannot continue
    - on an interrupt: main purpose of a timer (pikmin) interrupt
