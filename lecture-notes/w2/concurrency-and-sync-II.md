# Week 2 - Concurrency and Synchronisation: Part II

## Definitions
DEF: Critical Region -> A region of code where shared resources are accessed.
DEF: Mutual Exclusion -> The process of quarantining a critical region, such that only one process can access.
DEF: Atmoic -> Code runs 'atomically' if it is unable to be interupted; mutually excluded.
DEF: Bounded Buffer -> a fixed size buffer to limit production and consumption to a managable level.
DEF: TSL -> Test-and-Set Lock


---

## Test-and-Set
### Overview
- A binary lock; 0 == 'section free', 1 == 'section in use'


### Pros
- Simple to impliment, simple to understand.
- Works for any number of processes or locks
- Available at user-level

### Cons
- Busy waiting; processes keep spinning: wastes CPU cycles
- Possible starvation; multiple threads are waiting when lock released: luck based allocation

--- 

## Sleep/Wakeup: Solution to Busy Wait
- Calls sleep to block, instead of CPU cycling
- When region is free, an 'event manager' processed wakes one up
- Not random: offers architecture to eliminate starvation

---

## Producer-Consumer Problem
### Overview
A producer fills a "bounded buffer", where a consumer uses the items.

### Issues
- Must maintain accurate count of items in buffer
- Producer:
	- should sleep when buffer full
  - should wake when buffer has space
- Consumer:
	- should sleep when buffer empty
  - should wake when buffer contains anything
  
### Solutions?
- Locking primative: theory of TSL
	- Issue? The processes of checking and sleeping must be atomic. 
  - Code below does NOT work: sleeps indefinitely.

```c
		acquire_lock(buf_lock)
    ...
    if (count == N)
    			sleep(prod);     
    ...
    release_lock(buf_lock) 
```
### Thus: Semaphores 
#### Overview
- Dijkstra (huge)
- Two primitives:
	- P(): proberen (wait/decrement) 
  - V(): verhogen (signal/increment)
- Combines TSL with a bounded buffer.
- Semaphore maintains processes queue; 'subs in' sleeping processes when current atomic process is finished.
- Wait and Signal operations are atomic :D
- The processes themselves call the primatives: semaphore manages requests.

#### Implimentation
```c
typedef struct {
	int count;
  struct process *L;
} sempaphore;
```
- Sempahore primatives NECESSARILY check if their count <= 0; ALWAYS true.

#### Issues
- can be error-prone; more conceptually difficult to manage
- MUST be in correct order, or code won't function.
- Difficult to debug.

--- 

## Monitors
### Overview
 - Language-level implimentation
 - Outlines blocks of code as 'monitors' containing 'procedure' methods.

### Pros
 - Easier to impliment: compiler vs. human
 - Enforces mutex mathematically
 
### Cons
 - Language support limited: NOT is OS/161, feature of Java
 - Overkill; blocks ANY processes from accessing ANY method inside the monitor if busy
 	- Mutexs MORE than just the critical section, rather the entire function.

---

## Condition Variables
 - Allows for processes to wait within a monitor, while still reqlinquishing control
 - x.wait() -> suspends process until another invokes signal
 - x.single() -> resumes ONE suspended process.
 - IMPORTANTLY can be paired with a Mutex OUTSIDE a montior
 - Relies on the idea of 'something to do'; if there's something to do, the process runs. Else, it wakes a producer.
 
### CVs vs. Semaphores
 - Semphores techincally solve all the issues CVs do, but in a more convoluted way
 - CVs don't require a count: they are a queue of sleeping processes
 - CVs must be paired with a Mutex.
 
---
