---
sticker: emoji//1f5a5-fe0f
tags: SistemOperasi
---
# Deadlocks
**Table of Content**
- [[#8.1 System Model|8.1 System Model]]
- [[#8.2 Deadlock in Multithreaded Applications|8.2 Deadlock in Multithreaded Applications]]
	- [[#8.2 Deadlock in Multithreaded Applications#8.2.1 Livelock|8.2.1 Livelock]]
- [[#8.3 Deadlock Characterization|8.3 Deadlock Characterization]]
	- [[#8.3 Deadlock Characterization#8.3.1 Necessary Conditions|8.3.1 Necessary Conditions]]
	- [[#8.3 Deadlock Characterization#8.3.2 Resource-Allocation Graph|8.3.2 Resource-Allocation Graph]]
- [[#8.4 Methods for Handling Deadlocks|8.4 Methods for Handling Deadlocks]]
- [[#8.5 Deadlock Prevention|8.5 Deadlock Prevention]]
	- [[#8.5 Deadlock Prevention#8.5.1 Mutual Exclusion|8.5.1 Mutual Exclusion]]
	- [[#8.5 Deadlock Prevention#8.5.2 Hold and Wait|8.5.2 Hold and Wait]]
	- [[#8.5 Deadlock Prevention#8.5.3 No Preemption|8.5.3 No Preemption]]
	- [[#8.5 Deadlock Prevention#8.5.4 Circular Wait|8.5.4 Circular Wait]]
- [[#8.6 Deadlock Avoidance|8.6 Deadlock Avoidance]]
	- [[#8.6 Deadlock Avoidance#8.6.1 Safe State|8.6.1 Safe State]]
	- [[#8.6 Deadlock Avoidance#8.6.2 Resources-Allocation-Graph Algorithm|8.6.2 Resources-Allocation-Graph Algorithm]]
	- [[#8.6 Deadlock Avoidance#8.6.3 Banker's Algorithm|8.6.3 Banker's Algorithm]]
			- [[#8.6.3.1 Safety Algorithm|8.6.3.1 Safety Algorithm]]
			- [[#8.6.3.2 Resource-Request Algorithm|8.6.3.2 Resource-Request Algorithm]]
- [[#8.7 Deadlock Detection|8.7 Deadlock Detection]]
	- [[#8.7 Deadlock Detection#8.7.1 Single Instance of Each Resource Type|8.7.1 Single Instance of Each Resource Type]]
	- [[#8.7 Deadlock Detection#Several Instances of a Resources Type|Several Instances of a Resources Type]]
	- [[#8.7 Deadlock Detection#8.7.3 Detection-Algorithm Usage|8.7.3 Detection-Algorithm Usage]]
- [[#8.8 Recovery from Deadlock|8.8 Recovery from Deadlock]]
	- [[#8.8 Recovery from Deadlock#8.8.1 Process and Thread Termination|8.8.1 Process and Thread Termination]]
	- [[#8.8 Recovery from Deadlock#8.8.2 Resource Preemption|8.8.2 Resource Preemption]]
---
In a computer environment where multiple tasks (threads) compete for limited resources, a problem called deadlock can occur. Deadlock happens when a thread needs a resource that's currently in use by another thread, leading to a standstill. We previously discussed this issue as a type of problem where nothing progresses in Chapter 6 Synchronization Tools.

An analogy to understand deadlock is a historical law in Kansas: when two trains met at a crossing, both had to stop, and neither could move until the other had passed.

This chapter explores ways for software developers and operating system programmers to prevent or resolve deadlocks. While some applications can identify potential deadlocks, operating systems generally don't have built-in tools to prevent them. Therefore, it's up to programmers to design their programs to avoid deadlocks. As technology advances and demands for multitasking and parallel processing grow, dealing with deadlock issues, along with other challenges related to system responsiveness, becomes increasingly important.
## 8.1 System Model
A system has a limited number of resources that various threads compete for. These resources can be categorized into different types, like CPU cycles, files, or I/O devices. Each type has a specific number of identical instances. If a thread requests a resource instance, it should receive any available instance of that type; otherwise, the resource types are defined incorrectly.

Common synchronization tools like mutex locks and semaphores are also considered system resources and can lead to deadlocks in contemporary computer systems. These locks are typically associated with specific data structures, and each lock instance is usually assigned its own resource class.

While this chapter primarily discusses kernel resources, threads can also use resources from other processes through interprocess communication, potentially causing deadlocks, but this is beyond the scope of the kernel's concern.

Threads must follow a specific sequence when using resources: they request a resource, use it (e.g., access a critical section in the case of a mutex lock), and then release it. A thread can request as many resources as it needs for its task but cannot request more resources than are available in the system. This sequence ensures orderly resource utilization in the system.

When threads in a system request and release resources, these actions are typically carried out through system calls like request() and release() for devices, open() and close() for files, or allocate() and free() for memory. Similarly, semaphore wait() and signal() operations, as well as mutex lock acquire() and release(), can be used for resource management.

The operating system keeps track of resource allocation through a system table, indicating whether resources are free or allocated and to which threads they are allocated. If a thread requests a resource already allocated to another, it is placed in a queue to wait for that resource.

Deadlock occurs when a set of threads is stuck because each is waiting for an event that only another thread in the set can trigger, often related to resource acquisition and release. While the primary concern is logical resources like mutex locks and semaphores, other events, such as network reading or interprocess communication (IPC), can also lead to deadlocks.

A classic example of a deadlock is the dining-philosophers problem, where philosophers represent threads and chopsticks symbolize resources. If all philosophers pick up their left chopstick simultaneously, none can proceed as they're all waiting for the right chopstick, causing a deadlock.

Developers of multithreaded applications must be cautious of deadlocks. Locking mechanisms presented in Chapter 6 aim to prevent race conditions, but developers must carefully manage how locks are acquired and released to avoid deadlock situations, as explained further.
## 8.2 Deadlock in Multithreaded Applications
Before delving into methods to identify and manage deadlock issues, let's illustrate how deadlock can arise in a multithreaded Pthread program that employs POSIX mutex locks. In this context:

- The `pthread_mutex_init()` function initializes a mutex in an unlocked state.
- Mutex locks are obtained using `pthread_mutex_lock()` and released with `pthread_mutex_unlock()`.
- If a thread tries to acquire a locked mutex, the `pthread_mutex_lock()` call will block that thread until the owner of the mutex lock invokes `pthread_mutex_unlock()`.
Two mutex locks are created and initialized in the following code example:
```c
pthread mutex t first mutex;  
pthread mutex t second mutex;  
pthread mutex init(&first mutex,NULL);  
pthread mutex init(&second mutex,NULL);
```
In this scenario, two threads, named thread one and thread two, are created, and both have access to two mutex locks. Thread one executes the function `do_work_one()`, while thread two executes `do_work_two()`, as illustrated in the code.
In this example, thread one tries to acquire the mutex locks in the sequence (1) first mutex and (2) second mutex. Concurrently, thread two attempts to obtain the mutex locks in the order (1) second mutex and (2) first mutex. 

The possibility of a deadlock arises when thread one secures the first mutex while thread two acquires the second mutex, leading to a situation where both threads are waiting for the mutex held by the other, resulting in a standstill or deadlock.
```c
/* thread one runs in this function */  
void *do work one(void *param)  
{  
pthread mutex lock(&first mutex);  
pthread mutex lock(&second mutex);  
/**  
* Do some work  
*/  
pthread mutex unlock(&second mutex);  
pthread mutex unlock(&first mutex);  
pthread exit(0);  
}  
/* thread two runs in this function */  
void *do work two(void *param)  
{  
pthread mutex lock(&second mutex);  
pthread mutex lock(&first mutex);  
/**  
* Do some work  
*/  
pthread mutex unlock(&first mutex);  
pthread mutex unlock(&second mutex);  
pthread exit(0);  
}
```
It's important to note that while deadlock is a possibility, it won't happen if "thread one" can acquire and release the mutex locks for the first and second mutexes before "thread two" attempts to acquire them. However, the occurrence of deadlock heavily relies on the scheduling of threads by the CPU scheduler. This example highlights a challenge with handling deadlocks: they can be challenging to identify and test for because they may only occur under specific scheduling conditions, making them somewhat unpredictable.
### 8.2.1 Livelock
Livelock is another form of liveness failure, similar to deadlock but with a different underlying issue. In a deadlock, all threads are stuck waiting for an event caused by another thread in the set. In contrast, livelock occurs when a thread repeatedly attempts an action that continuously fails. A common analogy for livelock is when two people try to pass each other in a narrow hallway. They keep moving in opposite directions, obstructing each other's path, and while they're not blocked entirely, they make no progress, essentially spinning in place.

Livelock, another form of liveness failure, can be demonstrated using the `pthread_mutex_trylock()` function in Pthreads. Unlike blocking locks, this function attempts to acquire a mutex without waiting. The code example modifies a previous case to use `pthread_mutex_trylock()`. This change can lead to livelock if one thread acquires the first mutex and the other acquires the second mutex. Subsequently, each thread attempts `pthread_mutex_trylock()`, fails, releases its lock, and repeats these steps endlessly.

Livelock typically arises when multiple threads repeatedly attempt the same failing action simultaneously. To mitigate this, each thread can be made to retry the action at random intervals, preventing simultaneous retries. This approach is akin to how Ethernet networks handle network collisions. Instead of immediately retransmitting after a collision, a host involved will wait for a random period before trying to transmit again.

While less common than deadlock, livelock remains a significant challenge in concurrent application design. Similar to deadlock, it may only manifest under specific scheduling conditions.
## 8.3 Deadlock Characterization
In the previous section we illustrated how deadlock could occur in multi-  
threaded programming using mutex locks. We now look more closely at con-  
ditions that characterize deadlock.
### 8.3.1 Necessary Conditions
Deadlock in a system can occur when four conditions simultaneously hold:
1. Mutual Exclusion: At least one resource must be in a nonsharable mode, meaning only one thread can use it at a time. If another thread requests this resource, it must wait until it's released.
2. Hold and Wait: A thread must already hold at least one resource while waiting to acquire additional resources held by other threads.
3. No Preemption: Resources cannot be forcibly taken away; a thread holding a resource must release it voluntarily after completing its task.
4. Circular Wait: There must be a circular chain of waiting threads, where T0 is waiting for a resource held by T1, T1 is waiting for a resource held by T2, and so on until Tn, which is waiting for a resource held by T0.
All four of these conditions must be met for a deadlock to occur. It's important to note that the circular-wait condition implies the hold-and-wait condition, so these conditions are interrelated. Although they are not entirely independent, it is useful to consider each condition separately for understanding and analysis purposes.
Livelock example in code:
```c
/* thread one runs in this function */  
void *do work one(void *param)  
{  
	int done = 0;  
	while (!done) {  
		pthread mutex lock(&first mutex);  
		if (pthread mutex trylock(&second mutex)) {  
		/**  
		* Do some work  
		*/  
		pthread mutex unlock(&second mutex);  
		pthread mutex unlock(&first mutex);  
		done = 1;  
		}  
		else  
			pthread mutex unlock(&first mutex);  
	}  
	pthread exit(0);  
}  
/* thread two runs in this function */  
void *do work two(void *param)  
{  
	int done = 0;  
	while (!done) {  
		pthread mutex lock(&second mutex);  
		if (pthread mutex trylock(&first mutex)) {  
		/**  
		* Do some work  
		*/  
		pthread mutex unlock(&first mutex);  
		pthread mutex unlock(&second mutex);  
		done = 1;  
		}  
		else { 
			pthread mutex unlock(&second mutex);
		}  
	}  
	pthread exit(0);
}
```
![[Pasted image 20230830193906.png]]
### 8.3.2 Resource-Allocation Graph
Deadlocks can be precisely described using a directed graph called a system resource-allocation graph. This graph consists of two types of vertices: threads (T = {T1, T2, ..., Tn}), representing all active threads, and resource types (R = {R1, R2, ..., Rm}), representing all resource types in the system.

- A directed edge from thread Ti to resource type Rj (Ti → Rj) signifies that thread Ti has requested an instance of resource type Rj and is waiting for it.
- A directed edge from resource type Rj to thread Ti (Rj → Ti) signifies that an instance of resource type Rj has been allocated to thread Ti.
- Ti → Rj is called a request edge, and Rj → Ti is called an assignment edge.

In graphical representation, threads are shown as circles, and resource types as rectangles with dots inside to represent multiple instances of a resource. A request edge points to the rectangle representing the resource type, while an assignment edge also specifies a particular dot within the rectangle.
![[Pasted image 20230831000831.png]]
In this graph, when a thread Ti requests a resource of type Rj, a request edge is added. If the request is fulfilled, it becomes an assignment edge. When the thread no longer needs the resource, the assignment edge is removed, indicating resource release. This graph provides a visual representation of resource allocation and can help in analyzing and identifying deadlock situations.  
The resource-allocation graph in Figure 8.4 represents the following situation:
- Sets:
    - Threads (T): T1, T2, T3
    - Resource types (R): R1, R2, R3, R4
    - Edges (E): T1 → R1, T2 → R3, R1 → T2, R2 → T2, R2 → T1, R3 → T3
- Resource Instances:
    - One instance of resource type R1
    - Two instances of resource type R2
    - One instance of resource type R3
    - Three instances of resource type R4
- Thread States:
    - Thread T1 is holding an instance of R2 and is waiting for an instance of R1.
    - Thread T2 is holding an instance of R1 and an instance of R2 and is waiting for an instance of R3.
    - Thread T3 is holding an instance of R3.
This configuration of threads and resources in the graph illustrates a complex scenario where threads hold and request various resources, forming a potential deadlock situation. It demonstrates the interconnectedness of threads and resources in the system.
In a resource-allocation graph, the presence of cycles is a critical factor in determining whether deadlock exists in a system:
1. **No Cycles:** If the graph contains no cycles, then no thread in the system is deadlocked.
2. **Cycle Present:** If the graph does contain a cycle, then a deadlock may exist.
    - If each resource type has exactly one instance, a cycle implies that a deadlock has occurred. Threads involved in the cycle are deadlocked. A cycle in this scenario is both a necessary and a sufficient condition for deadlock.
    - However, if resource types have multiple instances, the presence of a cycle doesn't necessarily indicate a deadlock. In this case, a cycle is necessary but not sufficient to confirm deadlock.

To illustrate this, consider the resource-allocation graph from [[Pasted image 20230831000831.png|Figure 8.4]]. If thread T3 requests an instance of resource type R2, it introduces a new request edge into the graph. Whether this results in a deadlock depends on the availability of resources and the specific resource allocation and release patterns of the threads involved. If there aren't enough resources to satisfy the request and create a cycle involving resource types with only one instance, then a deadlock may occur. Otherwise, deadlock may be avoided.
## 8.4 Methods for Handling Deadlocks
Deadlock problems can be addressed in one of three ways:
1. **Ignoring Deadlocks:** Many operating systems, including Linux and Windows, choose to ignore deadlocks entirely. It becomes the responsibility of kernel and application developers to handle deadlocks within their programs.
2. **Deadlock Prevention or Avoidance:** To ensure that deadlocks never occur, systems can employ either a deadlock prevention or a deadlock avoidance scheme. Deadlock prevention methods restrict how resource requests can be made, ensuring that at least one necessary condition for deadlock cannot hold. Deadlock avoidance requires advance information about which resources a thread will request and use, allowing the operating system to decide whether a request should be delayed based on resource availability and future requests.
3. **Detection and Recovery:** If neither prevention nor avoidance is employed, a system may enter a deadlock state. In this case, the system can use algorithms to detect deadlocks and recover from them. Detection algorithms examine the system's state to identify deadlocks, while recovery algorithms attempt to resolve the deadlock. These methods are discussed in further detail in Sections Deadlock Detection and Recovery from Deadlock.
In the absence of deadlock detection and recovery algorithms, a system might end up in a deadlock state without realizing it. This can lead to degraded system performance as resources are held by non-running threads. Ultimately, manual intervention is required to restart the system.
Most operating systems choose to ignore deadlocks due to considerations like cost and the infrequency of deadlock occurrences. Recovery methods for other liveness conditions, such as livelock, may also be repurposed for deadlock recovery in some cases.
## 8.5 Deadlock Prevention
To prevent deadlocks, it's crucial to disrupt at least one of the four necessary conditions that must hold for a deadlock to occur, as discussed in Section [[#8.3.1 Necessary Conditions]]. This preventive approach involves addressing each of these conditions individually to create a deadlock-free system. In the following sections, we will delve into each of these conditions and explore methods for breaking them to ensure that deadlocks cannot occur.
### 8.5.1 Mutual Exclusion
The mutual-exclusion condition must hold for a deadlock to potentially occur, meaning at least one resource in the system must be nonsharable. Sharable resources, which do not require exclusive access, cannot contribute to a deadlock. An example of a sharable resource is a read-only file. Multiple threads can access a read-only file simultaneously without waiting for each other.
However, it's essential to recognize that we cannot prevent deadlocks by eliminating the mutual-exclusion condition entirely. Some resources, like mutex locks, are inherently nonsharable. Multiple threads cannot simultaneously share such resources, making exclusive access a necessity. Therefore, the focus of deadlock prevention lies in addressing the other necessary conditions while acknowledging the existence of nonsharable resources.
### 8.5.2 Hold and Wait
To prevent the hold-and-wait condition in the system, it is necessary to ensure that when a thread requests a resource, it doesn't hold any other resources at the same time. Achieving this can be challenging, and there are two protocols that can be used:
1. **Preallocate All Resources:** In this protocol, each thread is required to request and be allocated all the resources it will need before it begins execution. However, this approach is impractical for most applications due to the dynamic and unpredictable nature of resource requests.
2. **Release All Resources Before Requesting More:** An alternative protocol allows a thread to request resources only when it has none. A thread can request some resources, use them, and then release all of them before requesting additional resources.
Both of these protocols have drawbacks. They can lead to low resource utilization, as resources may be allocated but remain unused for extended periods. Additionally, the potential for starvation exists, where a thread needing multiple popular resources may have to wait indefinitely because at least one of the required resources is always allocated to other threads. These challenges make achieving deadlock prevention through resource allocation policies a complex task.
### 8.5.3 No Preemption
The third necessary condition for deadlocks is that resources should not be preempted once they are allocated. To prevent this condition from holding, a protocol can be employed:
1. **Preempt Resources When Necessary:** If a thread is currently holding some resources and requests another resource that cannot be immediately allocated (requiring the thread to wait), all the resources the thread is holding are preempted. These preempted resources are then added to the list of resources for which the thread is waiting. The thread will only be restarted when it can regain both its original resources and the new ones it requested.
2. **Resource Allocation with Preemption:** When a thread requests resources, the system first checks if they are available. If they are, they are allocated. If not, the system checks if these resources are held by another thread that is waiting for additional resources. If so, the desired resources are preempted from the waiting thread and allocated to the requesting thread. If the resources are neither available nor held by a waiting thread, the requesting thread must wait. While waiting, some of the thread's resources may be preempted, but only if another thread requests them. The thread can restart only when it gets the new resources it requested and recovers any resources preempted while waiting.
It's important to note that this protocol is most applicable to resources whose state can be easily saved and restored later, such as CPU registers and database transactions. It is generally not suitable for resources like mutex locks and semaphores, which are often involved in deadlock scenarios and where preempting resources can lead to undesirable consequences.
### 8.5.4 Circular Wait
The circular-wait condition, the fourth necessary condition for deadlocks, offers a practical opportunity for prevention by invalidating one of the necessary conditions. This can be achieved by imposing a total ordering of all resource types and requiring that each thread requests resources in an increasing order according to this enumeration.
To implement this approach:
1. Assign a unique integer number to each resource type in the set R = {R1, R2, ..., Rm}. These numbers allow for comparing resources to determine their order.
2. Formally define a one-to-one function F: R → N, where N represents the set of natural numbers.
3. Create an ordering among all synchronization objects (resource types) in the system, ensuring that threads request resources following this order.
By enforcing this ordering scheme, you can prevent the circular-wait condition from occurring, thereby eliminating one of the necessary conditions for deadlocks and making the system more deadlock-resistant.  
To prevent deadlocks by breaking the circular-wait condition, a protocol can be implemented in which each thread is required to request resources in an increasing order of enumeration. This means that a thread can request an instance of resource Ri initially and then request an instance of resource Rj only if F(Rj) > F(Ri), where F(R) is a function that assigns a unique integer value to each resource type.
For example, if a thread wants to use both "first mutex" and "second mutex" concurrently, it must first request "first mutex" and then "second mutex" following the order defined by the function F.
Using these protocols ensures that the circular-wait condition cannot hold, as demonstrated through a proof by contradiction. If a circular wait were to exist, it would imply a conflicting order of resource requests, which is impossible under this protocol.
It's important to note that establishing a lock ordering doesn't automatically prevent deadlocks; it requires application developers to follow this ordering. Additionally, managing lock ordering can be challenging, especially in systems with numerous locks. Some developers use strategies like the `System.identityHashCode(Object)` method in Java to establish lock ordering.
Furthermore, imposing lock ordering may not guarantee deadlock prevention if locks can be acquired dynamically, as in scenarios where multiple threads invoke functions involving resource acquisition simultaneously. In such cases, race conditions and potential deadlocks can still occur. That is, one thread might invoke
`transaction(checking account, savings account, 25.0)`
and another might invoke
`transaction(savings account, checking account, 50.0)`

Code example:
```c
void transaction(Account from, Account to, double amount)  
{  
mutex lock1, lock2;  
lock1 = get lock(from);  
lock2 = get lock(to);  
acquire(lock1);  
acquire(lock2);  
withdraw(from, amount);  
deposit(to, amount);  
release(lock2);  
release(lock1);  
}
```
## 8.6 Deadlock Avoidance
Deadlock-prevention algorithms limit resource requests to prevent deadlocks, but they can lead to low device utilization and reduced system throughput. An alternative approach to avoid deadlocks involves requiring additional information about how resources will be requested. This approach allows the system to decide, for each request, whether a thread should wait based on a complete sequence of requests and releases provided for each thread. The system considers the available resources, resources allocated to each thread, and future resource requests and releases when making these decisions.
Different algorithms use varying amounts and types of information. The simplest and most practical model requires each thread to declare its maximum resource requirements for each resource type it may need. With this information, an algorithm can be constructed to ensure that the system never enters a deadlocked state. These deadlock-avoidance algorithms dynamically assess the resource allocation state to prevent circular-wait conditions. The resource-allocation state is determined by available and allocated resources as well as the maximum demands of the threads. In the following sections, two deadlock-avoidance algorithms will be explored in more detail.
> [!LINUX LOCKDEP TOOL]
> Although ensuring that resources are acquired in the proper order is the  
responsibility of kernel and application developers, certain software can be  
used to verify that locks are acquired in the proper order. To detect possible  
deadlocks, Linux provides lockdep, a tool with rich functionality that can be  
used to verify locking order in the kernel. lockdep is designed to be enabled  
on a running kernel as it monitors usage patterns of lock acquisitions and  
releases against a set of rules for acquiring and releasing locks. Two examples  
follow, but note that lockdep provides significantly more functionality than  
what is described here:  
• The order in which locks are acquired is dynamically maintained by the  
system. If lockdep detects locks being acquired out of order, it reports a  
possible deadlock condition.  
• In Linux, spinlocks can be used in interrupt handlers. A possible source  
of deadlock occurs when the kernel acquires a spinlock that is also used  
in an interrupt handler. If the interrupt occurs while the lock is being  
held, the interrupt handler preempts the kernel code currently holding  
the lock and then spins while attempting to acquire the lock, resulting  
in deadlock. The general strategy for avoiding this situation is to disable  
interrupts on the current processor before acquiring a spinlock that is  
also used in an interrupt handler. If lockdep detects that interrupts are  
enabled while kernel code acquires a lock that is also used in an interrupt  
handler, it will report a possible deadlock scenario.  
lockdep was developed to be used as a tool in developing or modifying  
code in the kernel and not to be used on production systems, as it can  
significantly slow down a system. Its purpose is to test whether software  
such as a new device driver or kernel module provides a possible source  
of deadlock. The designers of lockdep have reported that within a few  
years of its development in 2006, the number of deadlocks from system  
reports had been reduced by an order of magnitude.âŁž Although lockdep  
was originally designed only for use in the kernel, recent revisions of this  
tool can now be used for detecting deadlocks in user applications using  
Pthreads mutex locks. Further details on the lockdep tool can be found at  
https://www.kernel.org/doc/Documentation/locking/lockdep-design.txt.
### 8.6.1 Safe State
The conditions of safe state is like the following explanation:
- A state is considered "safe" if the system can allocate resources to each thread (up to its maximum requirement) in some order and avoid a deadlock.
- More formally, a system is in a safe state if there exists a "safe sequence" of threads <T1, T2, ..., Tn>, where for each Ti in the sequence, its resource requests can be satisfied by the currently available resources plus the resources held by all threads Tj, where j < i.
- In a safe state, if the resources needed by a thread are not immediately available, that thread can wait until all preceding threads in the sequence have finished, obtain its required resources, complete its task, return the allocated resources, and terminate.
- A deadlocked state is an "unsafe" state, but not all unsafe states are deadlocks. Unsafe states may lead to a deadlock, but they are not necessarily deadlocks.
- Unsafe states are influenced by thread behavior; the operating system cannot prevent threads from requesting resources in a way that leads to a deadlock when the system is in an unsafe state.
An example illustrates this concept with a system having twelve resources and three threads: T0, T1, and T2. At a specific time t0, T0 holds five resources, T1 holds two, and T2 holds two, leaving three free resources.
![![/#^Table]]
At time t0, the system is in a safe state with the sequence <T1, T0, T2>, satisfying the safety condition. This sequence allows each thread to obtain and release its resources in a way that keeps the system in a safe state.
A system can transition from a safe state to an unsafe state if resource allocation is not handled carefully. For example, if at time t1, thread T2 is allocated one more resource, the system becomes unsafe. In an unsafe state, a deadlock can occur if resource requests are not managed properly.
To avoid deadlocks, avoidance algorithms are designed to ensure that the system always remains in a safe state. These algorithms make decisions about whether a thread's resource request should be granted immediately or if the thread should wait. The request is granted only if it ensures that the system remains in a safe state.
While avoidance algorithms prevent deadlocks, they may result in lower resource utilization because threads may have to wait for resource allocation, even if resources are available.
### 8.6.2 Resources-Allocation-Graph Algorithm
In this deadlock avoidance algorithm, a resource-allocation system with only one instance of each resource type is considered. To prevent deadlocks, the system uses a resource-allocation graph that includes three types of edges: request edges, assignment edges, and claim edges.
- **Request edges**: Represented by an arrow from a thread to a resource type, indicating that the thread is actively requesting that resource.
- **Assignment edges**: Represented by an arrow from a resource type to a thread, indicating that the resource has been allocated to the thread.
- **Claim edges**: Represented by dashed lines, these edges indicate that a thread may request a resource type at some point in the future. Claim edges are introduced to allow threads to declare their potential future resource needs.
The resources must be claimed in advance, meaning that before a thread starts executing, all its claim edges must be established. However, you can relax this condition by allowing a claim edge to be added only if all the edges associated with that thread are claim edges.
When a thread requests a resource (e.g., T_i requests R_j), the request can be granted only if converting the corresponding claim edge T_i → R_j to an assignment edge R_j → T_i does not create a cycle in the resource-allocation graph. To check for the presence of a cycle, a cycle-detection algorithm is used, which typically requires O(n^2) operations, where n is the number of threads in the system.
If no cycle is found, granting the resource request will leave the system in a safe state. If a cycle is detected, then the allocation would put the system in an unsafe state, indicating that the thread making the request must wait until the resource becomes available. This algorithm ensures that resources are allocated in a way that avoids potential deadlock situations.
It's important to note that this algorithm requires proactive resource claiming by threads and introduces some waiting if a resource allocation could lead to a cycle in the graph. While it prevents deadlocks, it may also result in resource underutilization due to the need for conservative resource allocation.
### 8.6.3 Banker's Algorithm
![[Pasted image 20230830201018.png]]
The banker's algorithm is a deadlock avoidance algorithm used in resource allocation systems where multiple instances of each resource type are available. It's designed to ensure that resource allocation never leads to a deadlock situation. This algorithm is often compared to how a bank allocates its available cash to customers to ensure it can meet all their needs.

Here's an overview of how the banker's algorithm works:
1. **Initialization**: When a new thread enters the system, it must declare the maximum number of instances of each resource type it may need. This declaration ensures that no thread can request more resources than are available in the system.
2. **Data Structures**:
   - **Available**: A vector of length `m` (the number of resource types) indicates the number of available resources of each type. For example, if `Available[j]` equals `k`, then `k` instances of resource type `Rj` are available.
   - **Max**: An `n x m` matrix defines the maximum demand of each thread. If `Max[i][j]` equals `k`, then thread `Ti` may request at most `k` instances of resource type `Rj`.
   - **Allocation**: An `n x m` matrix defines the number of resources of each type currently allocated to each thread. If `Allocation[i][j]` equals `k`, then thread `Ti` is currently allocated `k` instances of resource type `Rj`.
   - **Need**: An `n x m` matrix indicates the remaining resource need of each thread. If `Need[i][j]` equals `k`, then thread `Ti` may need `k` more instances of resource type `Rj` to complete its task. Note that `Need[i][j]` equals `Max[i][j] - Allocation[i][j]`.
3. **Notation**: The algorithm uses notation to compare vectors and check resource availability. For instance, `X ≤ Y` means that vector `X` is less than or equal to vector `Y`. Similarly, `Y < X` means that vector `Y` is less than vector `X`.
4. **Resource Request and Allocation**:
   - When a thread requests a set of resources, the system checks if allocating these resources will leave the system in a safe state. A state is considered safe if there exists a sequence in which all threads can finish their tasks without encountering a deadlock.
   - The system checks if `Request <= Need`, where `Request` is the requested resources vector, and `Need` is the remaining need vector for that thread.
   - If `Request <= Need`, it further checks if `Request <= Available`. If these conditions are met, the resources are allocated.
   - If the resources can't be allocated immediately, the requesting thread must wait until they become available without violating the safety conditions.
5. **Resource Release**: When a thread has finished using its allocated resources, it releases them. The system then updates the `Available`, `Allocation`, and `Need` matrices accordingly.

The banker's algorithm ensures that resources are allocated in a way that avoids potential deadlock situations by checking and maintaining a safe state. However, it may result in some resource underutilization due to the conservative allocation approach.
##### 8.6.3.1 Safety Algorithm
The safety algorithm is used to determine whether a system is in a safe state, meaning that there exists a sequence of thread execution that avoids deadlock. Here's how the algorithm works:
1. **Initialization**:
   - Initialize two vectors:
     - `Work`: A vector of length `m` (the number of resource types) initialized to the number of available instances of each resource type. `Work[j]` represents the available instances of resource type `Rj`.
     - `Finish`: A vector of length `n` (the number of threads) initialized to `false` for all threads. `Finish[i]` indicates whether thread `Ti` has finished its execution (initially set to `false` for all threads).
2. **Finding a Thread to Execute**:
   - The algorithm looks for an index `i` such that both conditions are met:
     a. `Finish[i]` is `false`, meaning that the thread `Ti` has not finished executing yet.
     b. `Needi ≤ Work`, indicating that the resource need of thread `Ti` can be satisfied by the available resources (`Work` vector).
   - If no such `i` exists, it means that there is no thread whose resource needs can be satisfied with the available resources. In this case, the system is not in a safe state.
3. **Allocating Resources and Marking Thread as Finished**:
   - If an index `i` satisfying the conditions is found in step 2, the algorithm proceeds to allocate resources to thread `Ti`:
     - `Work = Work + Allocationi`: Increment the `Work` vector by the resources allocated to thread `Ti`.
     - `Finish[i] = true`: Mark thread `Ti` as finished.
   - After this allocation and marking, the algorithm loops back to step 2 to find the next thread that can execute.
4. **Safe State Check**:
   - If all threads have been marked as finished (`Finish[i] == true` for all `i`), then the system is in a safe state. This means that there exists a sequence in which all threads can complete their tasks without encountering a deadlock.
The safety algorithm iterates through the threads and allocates resources if the thread's needs can be satisfied with the currently available resources. If all threads can be marked as finished at the end of the algorithm, it indicates that the system is in a safe state.
However, it's important to note that the safety algorithm only checks if a given state is safe; it does not provide a solution for resource allocation. The banker's algorithm, which we discussed earlier, uses this safety algorithm as a fundamental part to ensure that resource allocation avoids deadlock while maintaining a safe state.
##### 8.6.3.2 Resource-Request Algorithm
The resource-request algorithm, is used to determine whether a thread's request for additional resources can be safely granted without leading to a deadlock. Here's how the algorithm works:
1. **Request Validation**:
   - When a thread `Ti` makes a request for resources, represented by the vector `Requesti`, the algorithm first validates the request.
   - If `Requesti ≤ Needi`, it means that the requested resources do not exceed the maximum claim of the thread, so the request is valid. Otherwise, an error condition is raised as the thread has exceeded its maximum claim.
2. **Resource Availability Check**:
   - After validating the request, the algorithm checks if the requested resources are currently available.
   - If `Requesti ≤ Available`, it means that there are enough available resources to satisfy the request, so the algorithm proceeds to the next step. Otherwise, the thread must wait because the requested resources are not currently available.
3. **Resource Allocation Simulation**:
   - At this point, the algorithm simulates the allocation of the requested resources to thread `Ti` by modifying the system's state. The following changes are made:
     - `Available = Available - Requesti`: Decrease the available resources by the requested amount.
     - `Allocationi = Allocationi + Requesti`: Increase the allocation to thread `Ti` by the requested resources.
     - `Needi = Needi - Requesti`: Reduce the remaining resource needs of thread `Ti` accordingly.
4. **Safety Check**:
   - After simulating the allocation, the algorithm checks if the resulting resource allocation state is safe using the safety algorithm described earlier.
   - If the new state is safe, it means that granting the request will not lead to a deadlock, so the transaction is completed, and thread `Ti` is allocated its requested resources.
5. **Safety Violation Handling**:
   - If the new state is not safe, indicating that granting the request may lead to a deadlock, the thread `Ti` must wait for its request. In this case, the previous resource-allocation state (before the simulation) is restored.
The resource-request algorithm ensures that resource requests are granted in a way that maintains system safety. If a request can be granted without jeopardizing system safety, the resources are allocated to the thread. If granting the request would lead to an unsafe state, the thread must wait, and the previous state is restored.
This algorithm is a fundamental component of the banker's algorithm, which aims to ensure that resource allocation avoids deadlock while maintaining a safe state in the system.
## 8.7 Deadlock Detection
If a system does not employ either a deadlock-prevention or a deadlock-  
avoidance algorithm, then a deadlock situation may occur. In this environment,  
the system may provide:  
• **An algorithm that examines the state of the system to determine whether  
a deadlock has occurred** 
• **An algorithm to recover from the deadlock**  
Next, we discuss these two requirements as they pertain to systems with  
only a single instance of each resource type, as well as to systems with several instances of each resource type. At this point, however, we note that a  
detection-and-recovery scheme requires overhead that includes not only the  
run-time costs of maintaining the necessary information and executing the  
detection algorithm but also the potential losses inherent in recovering from  
a deadlock.
### 8.7.1 Single Instance of Each Resource Type
![[Pasted image 20230830202832.png]]
The wait-for graph, is a useful concept for detecting deadlocks in systems where all resources have only a single instance. It provides a way to visualize and analyze the relationships between threads and their resource dependencies. Here's a summary of how it works:
1. **Resource-Allocation Graph**: Initially, the system maintains a resource-allocation graph, which shows the relationships between threads and resources. This graph contains nodes for both threads and resources and edges representing the allocation of resources to threads.
2. **Wait-For Graph Construction**: To create the wait-for graph, you remove the resource nodes from the resource-allocation graph and collapse appropriate edges. In the wait-for graph, an edge from Thread $T_i$ to Thread $T_j$ implies that $T_i$ is waiting for $T_j$ to release a resource that $T_i$ needs. Such an edge exists in the wait-for graph if and only if the corresponding resource-allocation graph contains edges $T_i$ → Resource $R_q$ and Resource $R_q$ → $T_j$ for some resource $R_q$.
3. **Cycle Detection**: A deadlock is detected in the system if and only if the wait-for graph contains a cycle. This cycle represents a circular waiting condition where threads are waiting for each other's resources, leading to a deadlock.
4. **Periodic Detection**: To detect deadlocks, the system periodically invokes an algorithm to search for cycles in the wait-for graph. Common cycle detection algorithms, such as depth-first search (DFS) or breadth-first search (BFS), can be used for this purpose. The algorithm's complexity is typically $O(n^2)$, where n is the number of vertices in the graph.
5. **Deadlock Report**: When a cycle is detected in the wait-for graph, it indicates a potential deadlock situation. The system can then generate a report or take appropriate action to resolve the deadlock, such as terminating one or more threads or releasing resources.
Tools like the BCC toolkit you mentioned provide practical implementations of deadlock detection for specific environments, like Pthreads mutex locks in Linux. These tools help system administrators and developers identify and address potential deadlocks before they can seriously impact system performance or availability.
The key advantage of this approach is that it doesn't require significant modifications to the application code. However, it does come with some overhead in terms of maintaining the wait-for graph and running periodic detection algorithms.
### 8.7.2 Several Instances of a Resources Type
The deadlock detection algorithm you've described is designed to work in resource-allocation systems with multiple instances of each resource type. It's a straightforward algorithm that determines whether the system is in a deadlocked state by investigating the current allocation and request statuses of threads and resources. Here's a summary of how it works:
1. **Initialization**: Initialize two vectors, `Work` and `Finish`. `Work` represents the available resources of each type, and `Finish` is a vector of length `n` (the number of threads). Set `Work` equal to `Available` (the available resources of each type). For each thread `i`, if `Allocationi` is not zero, set `Finish[i]` to `false`; otherwise, set it to `true`.
2. **Finding a Thread**: Find an index `i` such that:
   - `Finish[i]` is `false`, indicating that the thread is not yet finished.
   - `Requesti` is less than or equal to `Work`, indicating that the resources requested by `T_i` can be satisfied by the currently available resources.
   If no such `i` exists, go to step 4.
3. **Resource Allocation**: Update `Work` and `Finish`:
   - `Work` is updated by adding `Allocationi`.
   - Set `Finish[i]` to `true` to mark the thread as finished.
   Then, return to step 2 to find the next thread.
4. **Deadlock Detection**: If, after performing steps 2 and 3, there are still threads with `Finish[i]` equal to `false`, it indicates that the system is in a deadlocked state. Additionally, if `Finish[i]` is `false` for a particular thread `T_i`, it means that `T_i` is one of the threads involved in the deadlock.
The algorithm will iterate through these steps until either all threads are marked as finished (`Finish[i]` is `true` for all `i`), which means the system is not in a deadlock, or until it identifies at least one thread with `Finish[i]` equal to `false`, indicating a deadlock involving that thread.
The complexity of this algorithm is approximately O(m × n^2), where `m` is the number of resource types, and `n` is the number of threads. It examines every possible allocation sequence for the threads to determine if a deadlock exists.
While this algorithm doesn't prevent deadlocks, it helps detect them when they occur, allowing the system to take appropriate actions to resolve the deadlock, such as terminating one or more threads or releasing resources.
> [!Deadlock Detection Using Java Thread Dumps]
> Although Java does not provide explicit support for deadlock detection, a  
thread dump can be used to analyze a running program to determine if  
there is a deadlock. A thread dump is a useful debugging tool that displays a  
snapshot of the states of all threads in a Java application. Java thread dumps  
also show locking information, including which locks a blocked thread is  
waiting to acquire. When a thread dump is generated, the JVM searches the  
wait-for graph to detect cycles, reporting any deadlocks it detects. To generate  
a thread dump of a running application, from the command line enter:  
Ctrl-L (UNIX , Linux, or mac OS)  
Ctrl-Break (Windows)  
In the source-code download for this text, we provide a Java example of the  
program shown in Figure 8.1 and describe how to generate a thread dump  
that reports the deadlocked Java threads.
### 8.7.3 Detection-Algorithm Usage
The decision of when to invoke the deadlock detection algorithm depends on a trade-off between system performance, resource utilization, and the likelihood of deadlocks. Here are some considerations for determining when to invoke the detection algorithm:
1. **Frequency of Deadlocks**: If deadlocks occur frequently in the system, it may be necessary to invoke the detection algorithm more frequently. Frequent deadlocks mean that resources are often tied up, leading to reduced system efficiency. In such cases, it's beneficial to identify and resolve deadlocks as soon as possible.
2. **Resource Scarcity**: If resources are scarce, there's a higher likelihood of deadlocks occurring. In resource-constrained environments, it may be wise to invoke the detection algorithm more frequently to minimize the impact of deadlocks on system performance.
3. **Resource Request Frequency**: As you mentioned, deadlocks occur when a thread makes a resource request that cannot be immediately granted. If the system has a high rate of resource requests and frequent contention for resources, it might be necessary to check for potential deadlocks more often.
4. **Resource Graph Complexity**: The complexity of the resource allocation graph can also impact when to invoke the detection algorithm. If the graph often contains many cycles, it might be challenging to identify the specific thread that caused the deadlock. In such cases, more frequent checks can help pinpoint the cause.
5. **Overhead**: Invoking the deadlock detection algorithm incurs computational overhead. Frequent checks can consume significant system resources, impacting overall system performance. Therefore, you should balance the need for deadlock detection with the system's processing capacity.
6. **CPU Utilization**: Monitoring CPU utilization can provide insights into when to invoke the detection algorithm. A drop in CPU utilization may indicate that the system is experiencing deadlock-related issues. In such cases, invoking the algorithm can help diagnose and resolve the problem.
7. **Defined Intervals**: In some cases, it may be acceptable to invoke the detection algorithm at defined intervals, such as during system maintenance periods or when system load is expected to be lower. This approach reduces overhead but may result in longer periods of deadlock before detection.
Ultimately, the decision on when to invoke the deadlock detection algorithm should be based on a thorough understanding of your system's characteristics, resource availability, and the impact of deadlocks on system performance. It may require a balance between proactive monitoring and resource-efficient operation.
## 8.8 Recovery from Deadlock
When a detection algorithm determines that a deadlock exists, several alter-  
natives are available. One possibility is to inform the operator that a deadlock  
has occurred and to let the operator deal with the deadlock manually. Another  
possibility is to let the system recover from the deadlock automatically. There are two options for breaking a deadlock. One is simply to abort one or more  
threads to break the circular wait. The other is to preempt some resources from  
one or more of the deadlocked threads.
### 8.8.1 Process and Thread Termination
The decision of which deadlocked process to abort is a complex policy decision and should be based on careful consideration of various factors. As you mentioned, many factors can affect this decision, and there isn't a one-size-fits-all answer. Here are some factors to consider when determining which deadlocked process to terminate:
1. **Priority**: Processes with lower priority may be considered for termination before those with higher priority. This assumes that the system has a priority mechanism in place.
2. **Computation Progress**: Consider how much work each process has completed and how close it is to finishing its task. Terminating a process that has made little progress may be more cost-effective than aborting one that is almost complete.
3. **Resource Usage**: Evaluate the resources each process has consumed. If a process has consumed many resources that are difficult to preempt, it might be a candidate for termination.
4. **Resource Needs**: Consider how many additional resources each process needs to complete its task. Terminating a process that requires many resources might have a lower impact on the system.
5. **Number of Processes**: Analyze the total number of deadlocked processes. If there are many such processes, you might choose to abort the smallest or least critical ones first.
6. **Data Integrity**: Assess the impact of termination on data integrity. If a process was in the middle of a critical data update, its termination might corrupt the data. In such cases, it might be better to abort other processes.
7. **Cost of Recovery**: Evaluate the cost of recovering from the termination. Consider the effort required to clean up after process termination and whether any lost work needs to be redone.
8. **User Impact**: Consider the impact on users or stakeholders. Terminating processes that are more critical to users might have a higher cost in terms of user satisfaction or business impact.
9. **Fairness**: Ensure that the termination policy is fair and does not unfairly target specific users or processes.
10. **System Policies**: Follow any predefined system policies or guidelines for deadlock resolution. Some systems may have specific policies in place for handling deadlocks.
In practice, deadlock resolution policies are often system-specific and depend on the specific requirements and characteristics of the application and environment. It's essential to document and communicate the chosen policy clearly and to periodically review and adjust the policy based on system behavior and changing requirements. Additionally, involving system administrators and stakeholders in the decision-making process can lead to more informed and acceptable deadlock resolution outcomes.
### 8.8.2 Resource Preemption
Resource preemption is a method used to break deadlocks by taking resources away from some processes and allocating them to others. It's a challenging problem with several considerations, as you've pointed out. Let's discuss each of the three issues related to resource preemption:
**Selecting a Victim:**
- Choosing which resources and processes to preempt is a crucial decision. The goal is to minimize the overall cost and disruption caused by preemption.
- Cost factors may include the number of resources held by a process, the amount of processing time it has consumed, or any other relevant metrics.
- The selection algorithm should aim to break the deadlock cycle with the least impact on the system and its users.
- It's essential to establish a fair and efficient preemption policy to ensure that no process is continually selected as a victim, which can lead to starvation.
**Rollback:**
- When a resource is preempted from a process, that process is left in an incomplete or inconsistent state.
- Rolling back a process means returning it to a previous state where it can continue execution.
- Determining the safe state to which a process should be rolled back can be complex. The simplest approach is total rollback, where the process is aborted and restarted.
- To minimize disruption, it's preferable to roll back a process only as far as necessary to break the deadlock, but this requires maintaining detailed state information for all processes.
**Starvation:**
- Starvation occurs when the same process is repeatedly selected as a victim for resource preemption, preventing it from making progress.
- To prevent starvation, it's crucial to ensure that a process cannot be picked as a victim an unlimited number of times.
- Including the number of rollbacks in the cost factor is a common approach to addressing this issue. This way, processes that have been preempted multiple times receive higher priority for resource allocation.
Resource preemption is a complex mechanism, and the design of preemption policies can vary significantly depending on the system's specific requirements and constraints. These policies often involve trade-offs between fairness, system performance, and the complexity of implementation. Careful consideration and testing are necessary to ensure that resource preemption effectively resolves deadlocks without introducing other issues such as excessive system overhead or unfair resource allocation.