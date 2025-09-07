# 100+ Tricky Java Multithreading Interview Questions

---

### 1. What is the difference between a process and a thread?

**Detailed Explanation:**  
A **process** is an independent program in execution, having its own memory space, code, data, and system resources. Each process runs in its own isolated environment, meaning one process cannot directly access variables or data structures of another process.  
A **thread** is a lightweight unit of execution within a process. All threads within the same process share the process’s address space and resources (like open files and memory), but each thread has its own execution stack and program counter.  
Threads are used for tasks that require concurrent execution within the same application, like handling multiple user requests in a web server, whereas processes are used for running separate applications.  
**In Java, multithreading refers to the concurrent execution of multiple threads within a single process (JVM).**

---

### 2. How do you create a thread in Java?

**Detailed Explanation:**  
There are three primary ways to create threads in Java:
- **Extending the `Thread` class:**  
  You define a subclass of `Thread` and override its `run()` method, which contains the code to execute in the new thread.
  ```java
  class MyThread extends Thread {
      public void run() {
          System.out.println("Hello from MyThread!");
      }
  }
  MyThread t1 = new MyThread();
  t1.start(); // Creates a new thread and calls run()
  ```
  *Note:* You should always start the thread with `start()`, not `run()`, otherwise it will run in the current thread.

- **Implementing the `Runnable` interface:**  
  Create a class that implements `Runnable` and pass an instance to a `Thread` object.
  ```java
  class MyRunnable implements Runnable {
      public void run() {
          System.out.println("Hello from MyRunnable!");
      }
  }
  Thread t2 = new Thread(new MyRunnable());
  t2.start();
  ```
  This approach is preferred if you want to inherit from another class, as Java does not support multiple inheritance.

- **Using Lambda Expressions (Java 8+):**  
  For short, simple tasks, you can use a lambda expression:
  ```java
  Thread t3 = new Thread(() -> System.out.println("Hello from Lambda!"));
  t3.start();
  ```

- **Using Callable and Future (for tasks that return results):**
  ```java
  Callable<Integer> c = () -> 42;
  ExecutorService executor = Executors.newSingleThreadExecutor();
  Future<Integer> f = executor.submit(c);
  System.out.println(f.get()); // 42
  executor.shutdown();
  ```
  `Callable` can return a value and throw exceptions.

---

### 3. What is the difference between `Runnable` and `Callable`?

**Detailed Explanation:**
- `Runnable` is an interface with a single method `void run()`. It does not return a result and cannot throw checked exceptions.
  ```java
  public interface Runnable {
      void run();
  }
  ```
- `Callable` is a generic interface with the method `V call() throws Exception`. It can return a result and throw checked exceptions.
  ```java
  public interface Callable<V> {
      V call() throws Exception;
  }
  ```
- Usage:
    - `Runnable` is used with `Thread` or `ExecutorService`.
    - `Callable` is used mainly with `ExecutorService` and returns a `Future`, which can be used to get results from asynchronous tasks.

**Example:**
```java
Callable<Integer> task = () -> 123;
Future<Integer> future = Executors.newSingleThreadExecutor().submit(task);
System.out.println(future.get()); // prints 123
```

---

### 4. What is thread safety? How do you achieve it in Java?

**Detailed Explanation:**  
**Thread safety** means that shared data structures are accessed by multiple threads in a way that guarantees correct results, regardless of the thread scheduling or timing.  
A class or method is thread-safe if it can be safely used by multiple threads at the same time without causing data corruption or inconsistent results.

**How to achieve thread safety:**
- **Synchronization:** Use the `synchronized` keyword to ensure that only one thread at a time can execute a block of code that modifies shared data.
  ```java
  public synchronized void increment() {
      count++;
  }
  ```
- **Locks:** Use explicit locks like `ReentrantLock` for finer control over synchronization.
  ```java
  Lock lock = new ReentrantLock();
  lock.lock();
  try {
      // critical section
  } finally {
      lock.unlock();
  }
  ```
- **Thread-safe classes:** Use classes from `java.util.concurrent` (e.g., `ConcurrentHashMap`, `CopyOnWriteArrayList`).
- **Immutability:** Make objects immutable (no setters, only final fields) so their state cannot change after construction.
- **Atomic variables:** Use classes like `AtomicInteger` for lock-free thread-safe operations.
- **Thread confinement:** Restrict access to an object to a single thread, e.g., using `ThreadLocal`.

---

### 5. What is a race condition? How do you avoid it?

**Detailed Explanation:**  
A **race condition** occurs when two or more threads access shared data and try to change it at the same time. Because the thread scheduling algorithm can swap between threads at any time, you don't know the order in which the threads will attempt to access the shared data. Therefore, the result of the changes in data is dependent on the thread scheduling, leading to unpredictable results.

**Example:**
```java
class Counter {
    private int count = 0;
    public void increment() {
        count++; // not atomic!
    }
}
```
If two threads call `increment()` simultaneously, both may read the same value and increment it, resulting in one increment being lost.

**How to avoid race conditions:**
- Use synchronization to ensure only one thread modifies the shared data at a time.
  ```java
  public synchronized void increment() { count++; }
  ```
- Use atomic classes:
  ```java
  AtomicInteger count = new AtomicInteger(0);
  count.incrementAndGet();
  ```
- Design systems to avoid sharing mutable data when possible.

---

### 6. What does the `volatile` keyword do?

**Detailed Explanation:**  
The `volatile` keyword in Java ensures that updates to a variable are immediately visible to all threads. When a field is declared as volatile, any write to that field is immediately made visible to other threads, and all reads of that field are from main memory, not a thread's local cache.

**Limitations:**
- `volatile` guarantees visibility, not atomicity.
- For compound actions (like incrementing a counter), `volatile` alone is not sufficient.

**Example:**
```java
private volatile boolean running = true;
```
If one thread sets `running = false`, other threads reading this variable will immediately see the updated value.

---

### 7. What is the difference between `synchronized` and `volatile`?

**Detailed Explanation:**
- **`volatile`:**
    - Only guarantees visibility of changes to variables across threads.
    - Does NOT guarantee atomicity.
    - Suitable for flags and simple variables.
- **`synchronized`:**
    - Guarantees both visibility and atomicity.
    - Enforces mutual exclusion (only one thread executes the synchronized block at a time).
    - Can be used to protect critical sections involving multiple variables.

**Example:**
```java
// Volatile for a flag
private volatile boolean running = true;

// Synchronized for critical section
public synchronized void increment() {
    count++;
}
```

---

### 8. Explain the lifecycle of a thread.

**Detailed Explanation:**  
Java threads have the following lifecycle states:
- **NEW:** Thread instance is created but not started.
- **RUNNABLE:** After calling `start()`, the thread is ready to run, waiting for CPU time.
- **RUNNING:** The thread is actually executing.
- **BLOCKED:** The thread is blocked, waiting to acquire a monitor lock.
- **WAITING:** The thread is waiting indefinitely for another thread to perform a particular action (e.g., `wait()`).
- **TIMED_WAITING:** The thread is waiting for a specific period (e.g., `sleep()`, `wait(timeout)`).
- **TERMINATED:** The thread has finished execution.

**Transitions occur due to method calls like `start()`, `wait()`, `notify()`, `sleep()` and completion of `run()`.

---

### 9. What is a deadlock? Give an example.

**Detailed Explanation:**  
A **deadlock** is a situation where two or more threads are waiting forever for each other to release a resource, resulting in all threads being blocked.

**Example:**
```java
class A {
    synchronized void foo(B b) {
        b.last();
    }
    synchronized void last() {}
}
class B {
    synchronized void bar(A a) {
        a.last();
    }
    synchronized void last() {}
}

A a = new A(); B b = new B();
Thread t1 = new Thread(() -> a.foo(b));
Thread t2 = new Thread(() -> b.bar(a));
t1.start();
t2.start();
```
- Thread 1 locks `a` and waits for `b`.
- Thread 2 locks `b` and waits for `a`.
- Both threads are blocked forever.

**Prevention:**
- Acquire locks in a fixed global order.
- Use lock timeouts (`tryLock()`).
- Keep lock scope as small as possible.

---

### 10. How do you prevent deadlocks?

**Detailed Explanation:**
- **Lock Ordering:** Always acquire locks in the same global order in every thread.
- **Lock Timeout:** Use `tryLock()` with a timeout to avoid waiting forever.
  ```java
  if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
      try {
          if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
              // work
          }
      } finally {
          lock1.unlock();
      }
  }
  ```
- **Lock Hierarchies:** Structure your locking to avoid circular waits.
- **Avoid Nested Locks:** Minimize or eliminate nested locking if possible.
- **Deadlock Detection:** Use thread dump analysis, JVM monitoring tools.

---

(…continue for all other questions, giving a clear, example-driven, and detailed answer for each, including code and real-world implications…)

---

### 11. What is a livelock?

**Detailed Explanation:**  
A _livelock_ is when two or more threads continually change state in response to each other but make no progress. Unlike deadlock, threads are not blocked but are too busy responding to each other to finish work.

_Example:_ Two threads repeatedly yielding to each other, never making progress.

---

### 12. What is starvation?

**Detailed Explanation:**  
_Starvation_ happens when a thread is perpetually denied access to resources and thus unable to make progress. This can occur if thread priorities are poorly managed or if locks are unfair.

---

### 13. What is the difference between user threads and daemon threads?

**Detailed Explanation:**
- **User threads:** Regular threads; JVM waits for all user threads to finish before exiting.
- **Daemon threads:** Background threads (e.g., garbage collector); JVM does not wait for these to finish. If only daemon threads remain, JVM exits.

_Set a thread as daemon before starting:_
```java
Thread t = new Thread(() -> { /* ... */ });
t.setDaemon(true);
t.start();
```

---

### 14. How do you create a daemon thread?

**Detailed Explanation:**  
Before calling `start()`, invoke `setDaemon(true)` on the thread object.

---

### 15. What is the purpose of `Thread.yield()`?

**Detailed Explanation:**  
`Thread.yield()` hints to the thread scheduler that the current thread is willing to yield its current use of CPU, allowing other threads of equal priority to execute. However, there's no guarantee the scheduler will act on this hint.

---

### 16. What is the difference between `wait()` and `sleep()`?

**Detailed Explanation:**
- `sleep(long ms)`: Pauses the thread for a specified time, does **not** release any locks.
- `wait()`: Causes the current thread to wait until another thread invokes `notify()`/`notifyAll()` on the same object. Releases the monitor lock so other threads can enter synchronized blocks.

---

### 17. How does `notify()` differ from `notifyAll()`?

**Detailed Explanation:**
- `notify()`: Wakes up one thread waiting on the object's monitor.
- `notifyAll()`: Wakes up all threads waiting on the object's monitor.
  Use `notifyAll()` when multiple threads may be waiting for different conditions.

---

### 18. Can you start a thread twice?

**Detailed Explanation:**  
No. Once a thread has been started and terminated, it cannot be started again. Calling `start()` a second time throws `IllegalThreadStateException`.

---

### 19. What is thread priority?

**Detailed Explanation:**  
Each thread has a priority (1 to 10). The scheduler uses this as a hint to determine thread execution order—higher priority threads may get more CPU time, but this is not guaranteed.  
Use `setPriority(int)` to change a thread's priority.

---

### 20. How do you implement the producer-consumer problem?

**Detailed Explanation:**  
Use `BlockingQueue` for safe communication between producer and consumer threads:
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(10);

// Producer
new Thread(() -> {
    for (int i = 0; i < 100; i++) {
        queue.put(i); // blocks if full
    }
}).start();

// Consumer
new Thread(() -> {
    while (true) {
        Integer val = queue.take(); // blocks if empty
        System.out.println(val);
    }
}).start();
```
This approach handles synchronization and waiting automatically.

---

### 21. What is a BlockingQueue?

**Detailed Explanation:**  
A `BlockingQueue` is a thread-safe queue supporting operations that wait for the queue to become non-empty when retrieving, and non-full when storing.  
Useful for producer-consumer and message-passing scenarios.

---

### 22. What are the benefits of using thread pools?

**Detailed Explanation:**
- **Thread reuse:** Avoids the overhead of creating new threads for each task.
- **Resource management:** Limits the number of active threads, preventing resource exhaustion.
- **Improved performance:** Threads are reused, reducing latency for new tasks.
- **Task scheduling:** Supports queuing and scheduling of tasks.

---

### 23. How do you use ExecutorService?

**Detailed Explanation:**  
`ExecutorService` provides a thread pool for managing and executing tasks.
```java
ExecutorService executor = Executors.newFixedThreadPool(4);
executor.submit(() -> doWork());
executor.shutdown();
```
Supports submitting `Runnable` or `Callable` tasks, and retrieving results via `Future`.

---

### 24. What does `shutdown()` vs `shutdownNow()` do in ExecutorService?

**Detailed Explanation:**
- `shutdown()`: Initiates an orderly shutdown. No new tasks are accepted; existing tasks are allowed to complete.
- `shutdownNow()`: Attempts to stop all executing tasks immediately and returns a list of tasks awaiting execution.

---

### 25. What is a Future?

**Detailed Explanation:**  
A `Future` represents the result of an asynchronous computation. You can:
- Check if the computation is complete (`isDone()`).
- Cancel the computation (`cancel()`).
- Get the result (`get()`), blocking if necessary until result is available.

---

### 26. How do you get the result from a Callable task?

**Detailed Explanation:**  
Submit the `Callable` to an `ExecutorService`, which returns a `Future`. Call `get()` on the `Future` to retrieve the result:
```java
Future<Integer> future = executor.submit(() -> 42);
Integer result = future.get(); // waits if necessary
```

---

### 27. What is Fork/Join framework?

**Detailed Explanation:**  
The Fork/Join framework (since Java 7) is designed for parallelism using divide-and-conquer algorithms. It works by recursively splitting tasks into subtasks using `ForkJoinPool` and `RecursiveTask`/`RecursiveAction`, and then combining their results.

---

### 28. How do you use a RecursiveTask in Fork/Join?

**Detailed Explanation:**  
Extend `RecursiveTask<V>` (for tasks returning a result), implement `compute()` to split or solve the task, and use `fork()`/`join()` to run subtasks.

**Example:**
```java
class SumTask extends RecursiveTask<Integer> {
    int[] arr; int start, end;
    protected Integer compute() {
        if (end - start <= 10) {
            int sum = 0;
            for (int i = start; i < end; i++) sum += arr[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(arr, start, mid);
        SumTask right = new SumTask(arr, mid, end);
        left.fork();
        return right.compute() + left.join();
    }
}
ForkJoinPool pool = new ForkJoinPool();
int result = pool.invoke(new SumTask(arr, 0, arr.length));
```

---

### 29. What is `ThreadLocal`?

**Detailed Explanation:**  
`ThreadLocal` provides thread-local variables; each thread gets its own, independently initialized copy. Useful for user session data, avoiding shared mutable state, and database connections.

**Example:**
```java
ThreadLocal<Integer> local = ThreadLocal.withInitial(() -> 0);
local.set(42);
System.out.println(local.get()); // Each thread sees its own value
```

---

### 30. What are atomic variables? Give an example.

**Detailed Explanation:**  
Classes in `java.util.concurrent.atomic` provide lock-free, thread-safe operations for single variables. They use low-level CPU instructions (CAS—Compare And Swap).

**Example:**
```java
AtomicInteger count = new AtomicInteger(0);
int newVal = count.incrementAndGet();
```
Atomic classes include `AtomicInteger`, `AtomicLong`, `AtomicReference`, etc.

---

### 31. What is a `CountDownLatch`?

**Detailed Explanation:**  
A `CountDownLatch` is a synchronization aid that allows one or more threads to wait until a set of operations in other threads completes.  
It is initialized with a given count. Each time a thread calls `countDown()`, the count decreases. Threads waiting on `await()` will block until the count reaches zero.

**Example:**
```java
CountDownLatch latch = new CountDownLatch(3);
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // perform some work
        latch.countDown();
    }).start();
}
latch.await(); // main thread waits until all 3 threads call countDown()
System.out.println("All workers finished");
```
**Use Case:**  
Waiting for several threads to finish before proceeding (e.g., starting a service only after all dependencies are ready).

---

### 32. What is a `CyclicBarrier`?

**Detailed Explanation:**  
A `CyclicBarrier` allows a group of threads to wait for each other to reach a common barrier point. Once all threads have called `await()`, the barrier is broken and all threads proceed. It is “cyclic” because it can be reused after the waiting threads are released.

**Example:**
```java
CyclicBarrier barrier = new CyclicBarrier(3, () -> System.out.println("All threads reached the barrier!"));
for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        // do work
        barrier.await(); // waits for others
        // proceed after barrier is tripped
    }).start();
}
```
**Use Case:**  
Useful for parallel computations where threads must periodically wait for each other.

---

### 33. What is a `Semaphore`?

**Detailed Explanation:**  
A `Semaphore` controls access to a shared resource by multiple threads by maintaining a set of permits. Threads acquire permits before accessing the resource and release them afterwards. If no permits are available, acquiring threads block.

**Example:**
```java
Semaphore sem = new Semaphore(2); // Only 2 threads can access the resource at a time
for (int i = 0; i < 5; i++) {
    new Thread(() -> {
        sem.acquire();
        try {
            // critical section
        } finally {
            sem.release();
        }
    }).start();
}
```
**Use Case:**  
Limiting the number of concurrent accesses to a resource (e.g., database connections).

---

### 34. What is `ReentrantLock`?

**Detailed Explanation:**  
`ReentrantLock` is a mutual exclusion lock with the same basic behavior as the `synchronized` keyword but with added flexibility.
- “Reentrant” means a thread can acquire the same lock multiple times.
- Supports features such as fairness, tryLock, lockInterruptibly, and explicit unlocking.

**Example:**
```java
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```
**Use Case:**  
When you need more flexible locking than `synchronized` (e.g., timed or interruptible lock acquisition).

---

### 35. What’s the difference between `synchronized` and `ReentrantLock`?

**Detailed Explanation:**
- **`synchronized`:**
    - Simple to use (just a keyword).
    - Lock is automatically released when exiting the block or method.
    - No fairness, no tryLock, cannot interrupt waiting thread.

- **`ReentrantLock`:**
    - Explicit lock/unlock (must unlock manually).
    - Supports tryLock (non-blocking), timed lock, interruptible lock, and fairness policies.
    - Better suited for complex locking scenarios.

**Example:**  
If you need to attempt to acquire a lock but not wait:
```java
if (lock.tryLock()) {
    try { /* ... */ } finally { lock.unlock(); }
}
```

---

### 36. Can you use `wait()` outside a synchronized block? Why or why not?

**Detailed Explanation:**  
No, you cannot call `wait()` (or `notify()`, `notifyAll()`) outside a synchronized context.  
**Reason:**  
These methods require the calling thread to own the monitor (lock) of the object. If you try to call them without synchronization, Java throws an `IllegalMonitorStateException`.

**Example (Incorrect):**
```java
Object lock = new Object();
lock.wait(); // Throws IllegalMonitorStateException
```

**Correct:**
```java
synchronized(lock) {
    lock.wait();
}
```

---

### 37. What is double-checked locking? Why use `volatile` with it?

**Detailed Explanation:**  
Double-checked locking reduces the overhead of acquiring a lock by first checking the locking condition without synchronization. Only if the check passes is synchronization used.

**Typical use case:** Lazy initialization of a singleton.

**Why `volatile`?**
Prior to Java 5, double-checked locking could fail due to the Java Memory Model. Declaring the singleton instance as `volatile` ensures that writes to the instance are visible to all threads and prevents instruction reordering.

**Example:**
```java
private volatile Singleton instance;
public Singleton getInstance() {
    if (instance == null) {
        synchronized(this) {
            if (instance == null) {
                instance = new Singleton();
            }
        }
    }
    return instance;
}
```

---

### 38. What is immutability? Why is it useful in concurrency?

**Detailed Explanation:**  
An immutable object’s state cannot change after construction.
- All fields are final and set in the constructor.
- No setters or mutators.

**Why useful?**
- Thread-safe by design: since state cannot change, there’s no risk of data races.
- Can be freely shared among threads without synchronization.

**Example:**
```java
final class Point {
    private final int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    // No setters
}
```

---

### 39. What is a thread dump? How do you analyze one?

**Detailed Explanation:**  
A thread dump is a snapshot of all threads in a JVM, showing their stack traces and states.  
**How to get one:**
- Use `jstack <pid>` or send `kill -3 <pid>` to the JVM process.
- In IDEs or tools like VisualVM.

**Analysis:**
- Look for `BLOCKED` threads to spot deadlocks.
- Examine stack traces to find long-running operations or lock contention.
- Useful for diagnosing performance bottlenecks and deadlocks.

---

### 40. What are the possible thread states in Java?

**Detailed Explanation:**
- **NEW**: Thread created, not yet started.
- **RUNNABLE**: Running or ready to run.
- **BLOCKED**: Blocked waiting for a monitor lock.
- **WAITING**: Waiting indefinitely for another thread’s action.
- **TIMED_WAITING**: Waiting for another thread’s action for up to a specified waiting time.
- **TERMINATED**: Thread has completed execution.

**You can get a thread's state using:**
```java
Thread.State state = thread.getState();
```

---

### 41. What is the difference between `start()` and `run()` methods?

**Detailed Explanation:**
- `start()`: Creates a new thread and calls the thread’s `run()` method in that new thread.
- `run()`: Executes the code in the current thread (won’t create a new thread).

**Example:**
```java
Thread t = new Thread(() -> System.out.println(Thread.currentThread().getName()));
t.start(); // Output: "Thread-0" (new thread)
t.run();   // Output: e.g., "main" (current thread)
```

---

### 42. What is `ThreadGroup`?

**Detailed Explanation:**  
`ThreadGroup` allows grouping multiple threads for collective management (e.g., interrupting all threads in a group). It is largely considered obsolete and is not recommended for new development, as modern concurrency utilities offer better control.

**Example:**
```java
ThreadGroup group = new ThreadGroup("MyGroup");
Thread t1 = new Thread(group, () -> {});
```

---

### 43. What is a thread pool leak? How do you prevent it?

**Detailed Explanation:**  
A thread pool leak occurs when tasks are submitted but never complete or are never removed, causing the pool’s work queue to fill up and reject new tasks, or the JVM to run out of threads.  
**Prevention:**
- Make sure tasks are not blocking indefinitely.
- Always shut down executors (call `shutdown()` or `shutdownNow()`).
- Use bounded queues.
- Monitor pool statistics.

---

### 44. Can you restart a thread?

**Detailed Explanation:**  
No. Once a thread has completed execution (TERMINATED), it cannot be started again. Attempting to do so throws `IllegalThreadStateException`. You must create a new `Thread` object.

---

### 45. How does thread scheduling work in Java?

**Detailed Explanation:**  
Thread scheduling is managed by the underlying OS and JVM implementation. Java provides thread priorities as hints, but the actual scheduling policy (e.g., time slicing, preemption) is platform-dependent.  
**You cannot rely on Java to schedule threads in a specific order.**

---

### 46. What is thread starvation? How do you avoid it?

**Detailed Explanation:**  
Thread starvation occurs when a thread is unable to gain regular access to resources and cannot make progress, often due to other threads monopolizing those resources.

**Avoidance:**
- Use fair locks (`new ReentrantLock(true)`) to ensure FIFO order.
- Avoid thread priorities unless truly needed.
- Design thread pools with fairness in mind.

---

### 47. How do you interrupt a thread? What happens?

**Detailed Explanation:**  
Call `interrupt()` on a thread.
- If the thread is sleeping or waiting, it throws `InterruptedException`.
- If running, the thread’s interrupted flag is set; it should check `Thread.interrupted()` or `isInterrupted()` and terminate itself gracefully.

**Example:**
```java
Thread t = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        // work
    }
});
t.start();
t.interrupt(); // sets interrupted flag
```

---

### 48. How does `CopyOnWriteArrayList` work?

**Detailed Explanation:**  
On every modification (add, set, remove), `CopyOnWriteArrayList` creates a new copy of the underlying array.
- Safe for concurrent reads (no need for synchronization).
- Expensive for frequent writes (due to copying).
- Iterators are weakly consistent (do not throw `ConcurrentModificationException`).

**Use Case:**  
Ideal for collections with many reads and few writes.

---

### 49. What is a `ConcurrentHashMap`?

**Detailed Explanation:**  
A `ConcurrentHashMap` is a thread-safe implementation of a hash map that allows concurrent reads and updates without locking the entire map.

**Mechanism:**
- Uses internal partitioning (segments or bins) to minimize locking.
- Multiple threads can update different segments concurrently.

**Example:**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.computeIfAbsent("b", k -> 2);
```

---

### 50. Why shouldn't you use `HashMap` in multithreaded code?

**Detailed Explanation:**  
`HashMap` is **not** thread-safe. Concurrent modification can:
- Corrupt the internal structure (leading to infinite loops or data loss).
- Cause unpredictable results.
  Use `ConcurrentHashMap` or synchronized wrappers for multi-threaded access.

---

### 50. Why shouldn't you use `HashMap` in multithreaded code?

**Detailed Explanation:**  
`HashMap` is **not** thread-safe. If multiple threads modify a `HashMap` concurrently (for example, adding or removing entries), its internal data structure can become corrupted. This may result in data loss, infinite loops, or even the JVM hanging.  
To safely use maps in multithreaded environments, use `ConcurrentHashMap` for high concurrency or `Collections.synchronizedMap()` for simple needs.

---

### 51. What is the difference between `ConcurrentHashMap` and `Collections.synchronizedMap()`?

**Detailed Explanation:**
- `ConcurrentHashMap` allows concurrent read and write access without locking the entire map. It uses internal partitioning ("buckets" or "segments") so multiple threads can work independently, offering high throughput.
- `Collections.synchronizedMap()` wraps a normal map with synchronized methods. All accesses (reads and writes) are synchronized, so only one thread can access the map at a time, limiting concurrency.

**Summary:**  
Use `ConcurrentHashMap` for better performance in highly concurrent applications.

---

### 52. What are parallel streams? When to use or avoid them?

**Detailed Explanation:**
- **Parallel streams** in Java 8+ allow you to process collections in parallel using multiple threads, leveraging multi-core CPUs.
  ```java
  list.parallelStream().forEach(System.out::println);
  ```
- **Use parallel streams** when:
    - The operation is CPU-bound (not I/O-bound).
    - The tasks are independent and stateless.
    - The processing per element is significant enough to overcome the overhead of splitting and joining.

- **Avoid parallel streams** when:
    - Processing is I/O-bound or tasks interact with shared mutable state.
    - The collection is small or the computation is trivial (parallel overhead may slow things down).
    - You need predictable thread management (use `ExecutorService` instead).

---

### 53. How does a `ReadWriteLock` work?

**Detailed Explanation:**  
A `ReadWriteLock` allows multiple threads to read a resource concurrently (as long as no thread is writing), but only one thread to write at a time (and no other thread—reader or writer—can access during write).

**Example:**
```java
ReadWriteLock rwLock = new ReentrantReadWriteLock();
rwLock.readLock().lock();
// perform read
rwLock.readLock().unlock();
rwLock.writeLock().lock();
// perform write
rwLock.writeLock().unlock();
```
**Benefit:**  
Greatly improves performance in read-heavy applications.

---

### 54. When should you use `CountDownLatch` vs `CyclicBarrier`?

**Detailed Explanation:**
- **`CountDownLatch`** is used for one-time events: threads wait until a count reaches zero, and then proceed. It cannot be reset.
- **`CyclicBarrier`** is reusable: a set of threads wait at the barrier until all have arrived, then all proceed. Useful for repeated synchronization points.

**Use cases:**
- Use `CountDownLatch` to wait for a number of threads/tasks to finish (e.g., service startup).
- Use `CyclicBarrier` for repeated coordination (e.g., simulation steps).

---

### 55. What is the difference between busy-waiting and blocking?

**Detailed Explanation:**
- **Busy-waiting**: The thread repeatedly checks a condition in a loop, consuming CPU cycles (e.g., `while(!done) {}`).
- **Blocking**: The thread is suspended and does not use CPU until it is notified or a condition is met (`wait()`, `sleep()`, `BlockingQueue.take()`).

**Summary:**  
Blocking is generally preferred as it allows the CPU to be used for other tasks.

---

### 56. What is a race condition? How does `AtomicInteger` help?

**Detailed Explanation:**  
A race condition occurs when two or more threads access a shared variable simultaneously and try to change it, causing unpredictable results.  
`AtomicInteger` provides thread-safe, lock-free operations for integers using atomic hardware instructions (like Compare-And-Swap).

**Example:**
```java
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet(); // atomic, thread-safe
```
No need for explicit synchronization.

---

### 57. Why is using `stop()`, `suspend()`, and `resume()` discouraged?

**Detailed Explanation:**  
These methods are deprecated because they are unsafe:
- `stop()` can leave shared resources in inconsistent states (no chance to clean up).
- `suspend()` can cause deadlocks (if a thread is suspended while holding a lock, other threads may be blocked forever).
- `resume()` may resume threads at inopportune times.

**Use interruption and flags instead for safe thread termination and pausing.**

---

### 58. What is thread affinity?

**Detailed Explanation:**  
Thread affinity (or CPU pinning) is binding a thread to a specific CPU core so it always runs on that core.  
Java does not provide direct support for thread affinity; it’s sometimes managed by the OS or native code for performance reasons in real-time or high-performance computing.

---

### 59. How do you profile thread contention in Java?

**Detailed Explanation:**
- Use tools such as VisualVM, Java Flight Recorder, JConsole, or command-line tools like `jstack`.
- Look for threads in BLOCKED or WAITING state, and analyze where threads are waiting for locks.
- Use profiling to identify hotspots in code (e.g., long-held locks, excessive synchronization).

---

### 60. What is priority inversion?

**Detailed Explanation:**  
Priority inversion happens when a high-priority thread is waiting on a lock held by a lower-priority thread, and a medium-priority thread preempts the low-priority thread, causing the high-priority thread to wait longer.

**Solution:**  
Some real-time systems use priority inheritance protocols, but Java's standard locks do not address this directly.

---

### 61. How does the JVM handle uncaught exceptions in threads?

**Detailed Explanation:**  
If a thread throws an uncaught exception, it terminates. The JVM prints a stack trace to the console.  
You can set a custom `UncaughtExceptionHandler` to log or handle errors:

**Example:**
```java
Thread t = new Thread(() -> { throw new RuntimeException("Oops"); });
t.setUncaughtExceptionHandler((thr, ex) -> System.out.println("Thread " + thr + " threw " + ex));
t.start();
```

---

### 62. What is a ThreadFactory?

**Detailed Explanation:**  
A `ThreadFactory` is an interface for creating new threads with custom configuration (name, priority, daemon status, etc.) used by executors and thread pools.

**Example:**
```java
ThreadFactory factory = r -> {
    Thread t = new Thread(r);
    t.setName("my-thread");
    return t;
};
ExecutorService exec = Executors.newFixedThreadPool(2, factory);
```

---

### 63. What is the difference between `Runnable` and `Thread`?

**Detailed Explanation:**
- `Runnable` is a functional interface representing a task with a `run()` method.
- `Thread` is a class that represents a worker executing a task (can be created by passing a `Runnable` to its constructor).

**Separation of concerns:**  
Use `Runnable` for code logic, `Thread` for execution.

---

### 64. How can you safely stop a thread?

**Detailed Explanation:**
- Set a volatile flag (e.g., `running = false`) that the thread checks in its loop.
- Or, call `interrupt()` and handle `InterruptedException` appropriately.

**Example:**
```java
volatile boolean running = true;
Thread t = new Thread(() -> {
    while (running && !Thread.currentThread().isInterrupted()) {
        // work
    }
});
t.start();
running = false; // or t.interrupt();
```

---

### 65. What are spurious wakeups?

**Detailed Explanation:**  
A spurious wakeup is when a thread waiting on a monitor (`wait()`) wakes up for no apparent reason (without `notify()` or `notifyAll()`).  
**Best practice:** Always call `wait()` inside a loop that checks the waiting condition.

**Example:**
```java
synchronized(lock) {
    while (!ready) {
        lock.wait();
    }
    // proceed when ready is true
}
```

---

### 66. How do you implement a thread-safe singleton?

**Detailed Explanation:**
- Use the "Initialization-on-demand holder" idiom:
  ```java
  public class Singleton {
      private Singleton() {}
      private static class Holder {
          private static final Singleton INSTANCE = new Singleton();
      }
      public static Singleton getInstance() {
          return Holder.INSTANCE;
      }
  }
  ```
- Or use an `enum` (see below for Q125).

Both are thread-safe and lazy-initialized.

---

### 67. Can a thread acquire multiple locks?

**Detailed Explanation:**  
Yes, a thread can nest multiple synchronized blocks or acquire multiple explicit locks.  
**Caution:** This increases the risk of deadlock (see Q9/Q10), so always acquire locks in a consistent order.

---

### 68. How does `Thread.sleep()` affect synchronization?

**Detailed Explanation:**  
`Thread.sleep()` pauses the thread for a given time but does **not** release any locks the thread holds.  
Other threads cannot acquire those locks during the sleep period.

---

### 69. What is the purpose of `join()`?

**Detailed Explanation:**  
`join()` causes the current thread to wait until another thread has completed.  
Useful for waiting for worker threads to finish before proceeding.

**Example:**
```java
Thread t = new Thread(() -> /* work */);
t.start();
t.join(); // waits for t to finish before continuing
```

---

### 70. How do you handle thread leaks?

**Detailed Explanation:**  
A thread leak occurs when threads are created but never terminated.  
**Prevention:**
- Always shut down thread pools (`shutdown()`).
- Avoid creating unbounded threads (use bounded queues).
- Use monitoring to detect orphaned threads.

---

### 71. What is the difference between `invokeAll()` and `invokeAny()` in ExecutorService?

**Detailed Explanation:**
- `invokeAll(Collection<Callable<T>> tasks)`: Submits a group of tasks. Waits for all to finish, returns a list of `Future<T>`.
- `invokeAny(Collection<Callable<T>> tasks)`: Submits a group of tasks. Returns the result of the first successfully completed task, cancels the rest.

---

### 72. What is work stealing in Fork/Join?

**Detailed Explanation:**  
In the Fork/Join framework, if a worker thread runs out of tasks, it "steals" tasks from the queues of other threads. This balances the workload and improves CPU utilization.

---

### 73. What is the difference between synchronous and asynchronous execution?

**Detailed Explanation:**
- **Synchronous**: The caller waits for the task to complete and gets the result immediately.
- **Asynchronous**: The caller initiates the task and continues without waiting for it to finish; the result is obtained later (e.g., via `Future`).

---

### 74. How do you implement thread-safe lazy initialization?

**Detailed Explanation:**  
Use the holder idiom or double-checked locking with `volatile` for singleton initialization (see Q37/Q66).

**Example:**
```java
public class Lazy {
    private static class Holder { static final Lazy INSTANCE = new Lazy(); }
    public static Lazy getInstance() { return Holder.INSTANCE; }
}
```

---

### 75. What is a fair lock?

**Detailed Explanation:**  
A fair lock ensures that lock acquisition happens in the order requested (FIFO).
- `new ReentrantLock(true)` creates a fair lock.
- Fair locks prevent thread starvation but may have lower throughput than unfair locks (default).

---

### 76. Explain the "producer-consumer with wait/notify" pattern.

**Detailed Explanation:**  
The producer adds items to a shared buffer. If the buffer is full, it waits.  
The consumer removes items. If the buffer is empty, it waits.

**Example:**
```java
class Buffer {
    private int data;
    private boolean available = false;
    public synchronized void put(int val) throws InterruptedException {
        while (available) wait();
        data = val; available = true; notifyAll();
    }
    public synchronized int get() throws InterruptedException {
        while (!available) wait();
        available = false; notifyAll(); return data;
    }
}
```
**Use `BlockingQueue` in modern code for simplicity and safety.**

---

### 77. When should you use `notifyAll()` instead of `notify()`?

**Detailed Explanation:**  
Use `notifyAll()` when multiple threads could be waiting for different conditions; you want to ensure that all waiting threads get a chance to check their waiting condition.

**Example:**  
If you have both producers and consumers waiting, `notifyAll()` ensures both can wake up.

---

### 78. Can you synchronize a static method?

**Detailed Explanation:**  
Yes. Declaring a static method as `synchronized` locks the `Class` object (not any instance), so only one thread can execute any static synchronized method of the class at a time.

---

### 79. What is a thread group? Is it recommended?

**Detailed Explanation:**  
A `ThreadGroup` is a legacy Java API for grouping threads together for bulk operations (interrupt, enumerate, etc.).  
Not recommended for new code. Use `ExecutorService` and modern concurrency utilities instead.

---

### 80. How do you detect deadlocks in Java?

**Detailed Explanation:**
- Take a thread dump using `jstack` or kill -3 <pid>.
- Analyze for threads in BLOCKED state and check their stack traces for lock ownership/waiting.
- Use `ThreadMXBean.findDeadlockedThreads()` in code.
- Use monitoring tools like VisualVM/JConsole.

---

### 81. What is a "barrier" in concurrency?

**Detailed Explanation:**  
A barrier is a synchronization point where a set of threads must all reach before any can proceed.  
`CyclicBarrier` (see Q32) is the Java implementation for this.

---

### 82. Can you force a thread to be scheduled?

**Detailed Explanation:**  
No. The thread scheduler is controlled by the JVM/OS. You can only suggest priorities, use `yield()`, or `sleep()`, but cannot force scheduling.

---

### 83. What is false sharing?

**Detailed Explanation:**  
False sharing occurs when threads on different CPUs modify variables that reside on the same cache line. This leads to performance degradation due to unnecessary cache coherency traffic.

**Solution:**
- Use padding or @Contended annotation (Java 8+) to separate frequently updated variables.

---

### 84. Explain Thread starvation and how to avoid it.

**Detailed Explanation:**  
Thread starvation is when one or more threads are unable to gain regular access to resources.  
Avoid by:
- Using fair locks/queues.
- Avoiding thread priorities unless necessary.
- Ensuring all threads have equal opportunity for execution.

---

### 85. What is the impact of thread priorities?

**Detailed Explanation:**  
Thread priorities are suggestions to the scheduler; higher-priority threads may get more CPU time, but this is not guaranteed and is platform-dependent.  
Do not rely on them for correctness.

---

### 86. What is a "thread dump"? How is it useful?

**Detailed Explanation:**  
A thread dump is a snapshot of all threads' states and stack traces in a JVM process at a point in time.  
Useful for diagnosing deadlocks, high CPU, and other concurrency issues.

---

### 87. Can you use `synchronized` on a constructor?

**Detailed Explanation:**  
No. The object is not fully constructed, and no other thread can access it until construction completes. Synchronizing a constructor is meaningless.

---

### 88. What is the difference between `Semaphore` and `CountDownLatch`?

**Detailed Explanation:**
- `Semaphore`: Controls access to a resource via permits; permits can be acquired/released multiple times.
- `CountDownLatch`: Waits for a set of events to occur (count reaches zero); cannot be reset or incremented.

---

### 89. What is the role of the `final` keyword in concurrency?

**Detailed Explanation:**
- Final fields make objects immutable after construction, which is naturally thread-safe.
- Final variables cannot be reassigned, ensuring thread-safety for constants.

---

### 90. When is it safe to use `Collections.synchronizedList()`?

**Detailed Explanation:**  
When you need a thread-safe list and do not require high concurrency (many threads reading/writing simultaneously).  
All methods are synchronized, so only one thread accesses the list at a time.

**For high-concurrency, prefer `CopyOnWriteArrayList`.**

---

### 91. How does the JVM allocate memory for threads?

**Detailed Explanation:**  
Each thread is given its own stack (for local variables, call frames, etc.). The Java heap and method area are shared among all threads.  
Stack size can be tuned via JVM options (`-Xss`).

---

### 92. Can you interrupt a thread blocked on I/O?

**Detailed Explanation:**  
Usually, you cannot reliably interrupt a thread blocked on native I/O (like socket or file I/O). Some NIO (New I/O) channels support interruption.  
For best results, design I/O operations to be non-blocking or have timeouts.

---

### 93. What is the difference between busy waiting and using `wait()`?

**Detailed Explanation:**
- **Busy waiting:** Thread repeatedly checks a condition in a loop, consuming CPU.
- **wait():** Thread suspends and releases the monitor; does not consume CPU until notified.

---

### 94. What is the purpose of the `UncaughtExceptionHandler`?

**Detailed Explanation:**  
To handle uncaught exceptions thrown by a thread (exceptions not caught in run()).  
You can log errors, clean up resources, or restart threads.

**Example:**
```java
Thread t = new Thread(() -> { throw new RuntimeException(); });
t.setUncaughtExceptionHandler((thr, ex) -> System.out.println("Thread died: " + ex));
t.start();
```

---

### 95. How does `CopyOnWriteArrayList` perform under frequent writes?

**Detailed Explanation:**  
Poorly. Each write operation creates a full copy of the underlying array, causing high memory and CPU usage.  
It is designed for scenarios with many reads and few writes.

---

### 96. What happens if a thread throws an uncaught exception?

**Detailed Explanation:**
- The thread terminates.
- The JVM prints the stack trace to standard error.
- Other threads are unaffected unless the exception is in the main thread and no other non-daemon threads are running.

---

### 97. How do you perform parallel computation on a collection in Java 8+?

**Detailed Explanation:**  
Use `parallelStream()`:
```java
List<Integer> list = Arrays.asList(1,2,3,4);
list.parallelStream().map(x -> x * x).forEach(System.out::println);
```
This splits processing across available CPU cores.

---

### 98. What is the difference between `invokeLater` and `invokeAndWait` in Swing?

**Detailed Explanation:**
- `invokeLater`: Schedules code to run on the Event Dispatch Thread (EDT) asynchronously.
- `invokeAndWait`: Schedules code to run on the EDT and waits for completion.

**Use for updating UI from background threads.**

---

### 99. What is the difference between `ReentrantLock` and `StampedLock`?

**Detailed Explanation:**
- `ReentrantLock`: Standard lock with reentrancy, fairness.
- `StampedLock`: Supports optimistic locking, read/write locks, and is designed for high-performance read-heavy scenarios.  
  However, it is NOT reentrant and can be more complex to use.

---

### 100. Why do we use `ExecutorService` instead of creating threads directly?

**Detailed Explanation:**
- Thread reuse: Avoids overhead of thread creation.
- Resource management: Limits number of concurrent threads.
- Task scheduling: Queues tasks for execution.
- Better error handling: Centralized exception handling and shutdown.

**Modern concurrent code should use `ExecutorService` for robustness and scalability.**

---

### 101. What is the difference between `ReentrantLock` and `StampedLock`?

**Detailed Explanation:**
- **`ReentrantLock`**:
    - A classic lock supporting reentrancy (the same thread can acquire the lock multiple times).
    - Offers `lock()`, `tryLock()`, fairness, and condition variables.
    - Simple to use for classic mutual exclusion.

- **`StampedLock`**:
    - Introduced in Java 8 for advanced scenarios.
    - Supports three modes: write lock (exclusive), read lock (shared), and optimistic read (non-blocking).
    - Optimistic reads allow non-blocking access if no write is in progress, greatly improving performance in read-heavy situations.
    - Not reentrant: a thread holding a lock cannot reacquire it.

**Use Case:**  
Use `StampedLock` for high-performance, read-heavy workloads, where optimistic reading is beneficial. Use `ReentrantLock` for classic mutual exclusion with reentrancy.

---

### 102. Why do we use `ExecutorService` instead of creating threads directly?

**Detailed Explanation:**
- **Thread reuse**: Creating new threads for every task is expensive; `ExecutorService` reuses threads from a pool.
- **Resource management**: Controls the maximum number of concurrent threads, preventing resource exhaustion.
- **Task queuing and scheduling**: Automatically manages queuing and scheduling of tasks.
- **Error handling**: Provides centralized error and exception management and organized shutdown procedures.
- **Scalability**: Supports scaling to handle many tasks efficiently and allows configuration of thread pool size and policies.

_Example:_
```java
ExecutorService executor = Executors.newFixedThreadPool(5);
executor.submit(() -> doWork());
executor.shutdown();
```

---

### 103. Can you store thread-specific data in a static field?

**Detailed Explanation:**  
No. Static fields are shared across all instances and all threads. They are part of the class, not the thread, so using a static field for thread-specific data leads to data races and overwriting.  
**Solution:**  
Use `ThreadLocal` to store data unique to each thread.

**Example:**
```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
threadLocal.set(42); // Each thread has its own value
```

---

### 104. How do you ensure a resource is always released in multithreaded code?

**Detailed Explanation:**  
Always release locks and resources in a `finally` block so that they are freed even if an exception occurs.

**Example:**
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```

---

### 105. What is the difference between `invokeAll` and `submit` in ExecutorService?

**Detailed Explanation:**
- `submit(Callable/Runnable)`: Submits a single task, returns a `Future` for its result.
- `invokeAll(Collection<Callable<T>> tasks)`: Submits a batch of tasks, returns a list of Futures. Waits until all tasks are complete.
- `invokeAny(Collection<Callable<T>> tasks)`: Submits a batch of tasks, returns the result of one that completes successfully (cancels the rest).

---

### 106. What is the concept of thread confinement?

**Detailed Explanation:**  
Thread confinement means restricting the use of data to a single thread. No other thread can access or modify it, eliminating the need for synchronization.

**Example:**
- Local variables inside a method are confined to the executing thread.
- `ThreadLocal` variables are confined to one thread.

---

### 107. Can you explain the ABA problem in concurrency?

**Detailed Explanation:**  
The ABA problem occurs when a thread reads a value A from a variable, then another thread changes it from A to B and back to A. The first thread, upon seeing A again, assumes nothing has changed, but in reality the value was modified.  
This is a problem in lock-free algorithms using Compare-And-Swap (CAS).

**Solution:**  
Use version stamps or `AtomicStampedReference` to track changes.

---

### 108. What is SpinLock and when is it used?

**Detailed Explanation:**  
A SpinLock is a lock where the thread repeatedly checks if the lock is available (busy-waiting) instead of blocking. It's efficient only if threads are expected to wait for a very short time and context switching is expensive.

**Java Example:**  
Java does not provide SpinLock directly, but can be implemented:
```java
AtomicBoolean lock = new AtomicBoolean(false);
while (!lock.compareAndSet(false, true)) {
    // spin
}
try {
   // critical section
} finally {
   lock.set(false);
}
```

---

### 109. How do you test multithreaded code?

**Detailed Explanation:**
- Use unit tests with concurrency utilities (e.g., `CountDownLatch`, `CyclicBarrier`) to coordinate threads.
- Use stress tests to expose race conditions.
- Employ code analysis tools (FindBugs, SonarQube) for potential concurrency bugs.
- Use thread sanitizer tools and logging to detect race conditions and deadlocks.
- Write deterministic tests by controlling thread execution with synchronization.

---

### 110. What is the difference between `synchronized(this)` and `synchronized(Class.class)`?

**Detailed Explanation:**
- `synchronized(this)`: Locks the current instance. Only one thread can execute synchronized blocks on the same object.
- `synchronized(Class.class)`: Locks the class object. Used for static methods or blocks; only one thread per class can execute.

**Example:**
```java
synchronized(MyClass.class) { /* static data */ }
synchronized(this) { /* instance data */ }
```

---

### 111. When should you use `AtomicReference`?

**Detailed Explanation:**  
When you need to perform atomic operations (like compare-and-swap) on object references (not just primitive values).  
Useful for building non-blocking data structures.

**Example:**
```java
AtomicReference<MyObject> ref = new AtomicReference<>(null);
ref.compareAndSet(null, new MyObject());
```

---

### 112. What is the difference between `Executor.submit()` and `Executor.execute()`?

**Detailed Explanation:**
- `execute(Runnable)`: Executes Runnable tasks, returns void.
- `submit(Runnable/Callable)`: Executes task, returns a `Future` that can be used to check status, cancel, or get result.

**Use `submit()` if you need to track task completion or result.**

---

### 113. Can you use `wait()` and `notify()` with `ReentrantLock`?

**Detailed Explanation:**  
No. `wait()` and `notify()`/`notifyAll()` are methods for intrinsic locks (used with `synchronized`).  
For `ReentrantLock`, use its associated `Condition` object’s `await()` and `signal()`/`signalAll()` methods.

**Example:**
```java
Lock lock = new ReentrantLock();
Condition cond = lock.newCondition();
lock.lock();
try {
    cond.await();
    // ...
    cond.signal();
} finally {
    lock.unlock();
}
```

---

### 114. What is a thread-safe way to increment a counter?

**Detailed Explanation:**
- Use `AtomicInteger` for atomic, lock-free increment:
  ```java
  AtomicInteger counter = new AtomicInteger(0);
  counter.incrementAndGet();
  ```
- Or, use synchronization:
  ```java
  synchronized(this) { counter++; }
  ```

---

### 115. What is the difference between concurrency and parallelism?

**Detailed Explanation:**
- **Concurrency:** Multiple tasks are in progress at the same time (may or may not be simultaneously).
- **Parallelism:** Multiple tasks are executed **simultaneously**, typically on multiple processors/cores.

**All parallelism is concurrency, but not all concurrency is parallelism.**

---

### 116. What is the effect of calling `start()` twice on the same thread?

**Detailed Explanation:**  
Throws an `IllegalThreadStateException`. A thread can only be started once in its lifetime.

---

### 117. What is optimistic locking?

**Detailed Explanation:**  
Optimistic locking assumes that multiple threads can complete their operations without interfering with each other. Before committing changes, a thread checks if the resource has been modified.  
If so, the operation is retried.  
Used in `StampedLock` and in databases.

---

### 118. What is the difference between `invokeAll()` and `invokeAny()` in ForkJoinPool?

**Detailed Explanation:**
- `invokeAll()`: Executes all tasks, waits for completion, returns all results.
- `invokeAny()`: Executes all tasks, returns the result of one that completes successfully, cancels the rest.

---

### 119. How do you avoid false sharing?

**Detailed Explanation:**
- Use padding (dummy variables) to ensure frequently updated variables are not placed in the same cache line.
- In Java 8+, use `@Contended` annotation (with JVM option `-XX:-RestrictContended`).

---

### 120. What is the purpose of `Thread.interrupted()`?

**Detailed Explanation:**
- `Thread.interrupted()`: Checks and **clears** the interrupted status of the current thread.
- `isInterrupted()`: Checks, **does not clear**.

Use this to detect and respond to interrupt signals in task loops.

---

### 121. How can you make a collection thread-safe?

**Detailed Explanation:**
- Use synchronized wrappers: `Collections.synchronizedList(new ArrayList<>())`
- Use concurrent collections: `ConcurrentHashMap`, `CopyOnWriteArrayList`, etc.

---

### 122. How can you make a method atomic?

**Detailed Explanation:**
- Use `synchronized` keyword on the method or block.
- Use atomic classes (`AtomicInteger`, etc.) for specific operations.

---

### 123. Why can't constructors be synchronized?

**Detailed Explanation:**  
Other threads cannot access the object being constructed until the constructor finishes, so synchronizing a constructor is pointless.

---

### 124. What is the difference between eager and lazy initialization in singleton?

**Detailed Explanation:**
- **Eager:** Singleton instance is created at class loading time.
  ```java
  public class Singleton {
      private static final Singleton INSTANCE = new Singleton();
      public static Singleton getInstance() { return INSTANCE; }
  }
  ```
- **Lazy:** Singleton instance is created only when requested.
  ```java
  public class Singleton {
      private static Singleton instance;
      public static synchronized Singleton getInstance() {
          if (instance == null) instance = new Singleton();
          return instance;
      }
  }
  ```

---

### 125. What is a thread-safe singleton pattern?

**Detailed Explanation:**  
The best way is to use an enum, which is thread-safe, handles serialization automatically, and guarantees only one instance.

**Example:**
```java
public enum Singleton {
    INSTANCE;
}
```
_Or, use the holder idiom or double-checked locking as described in earlier answers._

---
