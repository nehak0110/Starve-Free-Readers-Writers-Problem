# Starve-Free-Readers-Writers-Problem
Assignment submitted in requirement for the course CSN-232 (operating system).

The readers–writers problems are examples of a common computing problem in concurrency. There are at least three variations of the problems, which deal with situations in which many concurrent threads of execution try to access the same shared resource at one time.Some threads may read and some may write, with the constraint that no thread may access the shared resource for either reading or writing while another thread is in the act of writing to it. (In particular, we want to prevent more than one thread modifying the shared resource simultaneously and allow for two or more readers to access the shared resource at the same time). A readers–writer lock is a data structure that solves one or more of the readers–writers problems.
The reader-writer problems deal with synchronizing multiple processes trying to read or write upon a shared data. The first and second problem provide a solution where either the reader or writer processes can possibly starve. The third readers-writers problem deals with an implementation where neither the reader or the writer process will ever starve

## What is a semaphore?
A semaphore is a variable or abstract data type used to control access to a common resource by multiple processes and avoid critical section problems in a concurrent system such as a multitasking operating system. A trivial semaphore is a plain variable that is changed (for example, incremented or decremented, or toggled) depending on programmer-defined conditions.For a starve-free implementation, we need a semaphore that has a First-In-First-Out manner of handling the waiting processes. i.e. - the process that call wait first are the ones are the process that are given the access to semaphore first.

### Requirements of starve free readers writers problem:

1. **Shared data** - like file and database, etc
2. **Semaphores**:
+ **mutex**: mutual exclusion semaphores for updating *readCount* and *writeCount* variables.
+ **resourceAvailable**: mutual exclusion semaphores for
checking whether the resource is available or not.
3. Initially,
+mutex = 1, resourceAvailable = 1
+readCount = 0, writeCount = 0

## First readers–writers problem

Suppose we have a shared memory area (critical section) with the basic constraints detailed above. It is possible to protect the shared data behind a mutual exclusion mutex, in which case no two threads can access the data at the same time. However, this solution is sub-optimal, because it is possible that a reader R1 might have the lock, and then another reader R2 requests access. It would be foolish for R2 to wait until R1 was done before starting its own read operation; instead, R2 should be allowed to read the resource alongside R1 because reads don't modify data, so concurrent reads are safe. This is the motivation for the first readers–writers problem, in which the constraint is added that no reader shall be kept waiting if the share is currently opened for reading. This is also called readers-preference, with its solution.
Before entering the critical section, every new reader must go through the entry section. However, there may only be a single reader in the entry section at a time. This is done to avoid race conditions on the readers (in this context, a race condition is a condition in which two or more threads are waking up simultaneously and trying to enter the critical section; without further constraint, the behavior is nondeterministic. E.g. two readers increment the readcount at the same time, and both try to lock the resource, causing one reader to block). To accomplish this, every reader which enters the <ENTRY Section> will lock the <ENTRY Section> for themselves until they are done with it. At this point the readers are not locking the resource. They are only locking the entry section so no other reader can enter it while they are in it. Once the reader is done executing the entry section, it will unlock it by signalling the mutex.
    
##  Second readers–writers problem
The first solution is suboptimal, because it is possible that a reader R1 might have the lock, a writer W be waiting for the lock, and then a reader R2 requests access. It would be unfair for R2 to jump in immediately, ahead of W; if that happened often enough, W would starve. Instead, W should start as soon as possible. This is the motivation for the second readers–writers problem, in which the constraint is added that no writer, once added to the queue, shall be kept waiting longer than absolutely necessary. This is also called writers-preference.
In this solution, preference is given to the writers. This is accomplished by forcing every reader to lock and release the readtry semaphore individually. The writers on the other hand don't need to lock it individually. Only the first writer will lock the readtry and then all subsequent writers can simply use the resource as it gets freed by the previous writer. The very last writer must release the readtry semaphore, thus opening the gate for readers to try reading.
    
## Third readers–writers problem
The solutions implied by both problem statements can result in starvation — the first one may starve writers in the queue, and the second one may starve readers. Therefore, the third readers–writers problem is sometimes proposed, which adds the constraint that no thread shall be allowed to starve; that is, the operation of obtaining a lock on the shared data will always terminate in a bounded amount of time.
    
 **Writers Code:**
```wait(mutex);
writeCount++;
signal(mutex);
wait(resourceAvailable);
// write the resource;
wait(mutex);
writeCount--;
signal(mutex);
signal(resourceAvailable);
  ```

 Here, writers come and call wait(mutex) before increasing the
writeCount so that it can avoid incorrectness of writeCount due
concurrent increment or decrement of writeCount. After
increasing writeCount, it checks for the resource, if resource is
busy, then it will wait otherwise go for writing.
Soon after writing is done, a process again checks for mutex so
that decrement can be done. Ultimately, it signals both mutex
and resourceAvailable so that other process can use it.
    
 **Readers Code:**
```wait(mutex);
if (writeCount > 0 or readCount == 0)
signal(mutex);
wait(resourceAvailable);
wait(mutex);
readCount ++;
signal(mutex);
// read the resource;
wait(mutex);
readCount--;
if(readCount == 0)
signal(resourceAvailable);
 signal(mutex);
   ```
Here, readers come and call wait(mutex). If allowed to enter, it
checks if writeCount > 0 or readCount == 0.
If writeCount > 0, then it signals mutex so that any process can
use it and at the same time it checks for the resource, if
available; it calls wait(mutex) because it is now getting available
resources so only thing needs to done is to increase readCount.
When done with updating readCount, the process enters to
read the resource.
Soon after reading, it requires to decrease the readCount
variable, so it calls wait(mutex); if allowed, then decrements the
variable.If readCount is still > 0, it means some processes are still
reading so the process will leave without calling
signal(resourceAvailable) and just call signal(mutex) so that
other processes can enter into reading.
    
    

## Proof of Correctness 

The rwt semaphore ensures that only a single writer can access the critical section at any moment of time thus ensuring mutual exclusion between the writers and also when the first reader try to access the critical section it has to acquire the rwt mutex lock to access the critical section thus ensuring mutual exclusion between the readers and writers.Before accessing the critical section any reader or writer have to first acquire the turn semaphore which uses a FIFO queue for the blocked processes. Thus as the queue uses a FIFO policy, every process has to wait for a finite amount of time before it can access the critical section thus meeting the requirement of bounded waiting.

##References
+ Abraham Silberschatz, Peter B. Galvin, Greg Gagne - Operating System Concepts
+ Wikipedia
+ https://rfc1149.net/blog/2011/01/07/the-third-readers-writers-problem/
