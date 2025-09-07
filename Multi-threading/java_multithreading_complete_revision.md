# Java Multithreading: Comprehensive Revision Guide (with More Examples & Explanations)

---

## 1. What is Multithreading?

**Multithreading** is the concurrent execution of two or more threads for maximum CPU utilization within a single JVM process.

### Explanation:
- Each thread represents a separate path of execution, but shares process resources (memory, files, etc).
- Used for responsive UIs, server concurrency, background tasks, parallel computation.

---

## 2. Java Thread Model

### Process vs Thread

- **Process:** Independent, has its own memory space. Two processes don’t share data.
- **Thread:** Lightweight sub-processes. Threads within the same process share memory and resources.

**Example:**  
A web server process may spawn multiple threads to handle multiple client requests simultaneously.

---

## 3. Creating and Running Threads

### 3.1. Extending the Thread Class

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}
MyThread t1 = new MyThread();
t1.start(); // Don't call t1.run() directly!
```
**Explanation:**  
Inheriting Thread allows you to override the `run()` method. Use `start()` to create a new thread in the JVM.

---

### 3.2. Implementing Runnable Interface

```java
class MyRunnable implements Runnable {
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}
Thread t2 = new Thread(new MyRunnable());
t2.start();
```
**Explanation:**  
Implementing Runnable is preferred as it allows you to extend another class. Thread is created by passing a Runnable instance.

---

### 3.3. Using Lambda Expressions (Java 8+)

```java
Thread t3 = new Thread(() -> System.out.println("Lambda Thread: " + Thread.currentThread().getName()));
t3.start();
```
**Explanation:**  
Lambdas provide a concise way to create Runnable instances for simple tasks.

---

### 3.4. Using Callable and Future

```java
import java.util.concurrent.*;

Callable<Integer> task = () -> {
    Thread.sleep(500);
    return 42;
};
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
System.out.println(future.get()); // 42
executor.shutdown();
```
**Explanation:**  
Callable can return a value (unlike Runnable) and throw checked exceptions. Future represents the result of an async computation.

---

## 4. Thread Lifecycle

**States:**
- **NEW:** Thread created, not started
- **RUNNABLE:** Eligible to run (running or ready)
- **BLOCKED:** Waiting for monitor lock
- **WAITING/TIMED_WAITING:** Waiting for action or time
- **TERMINATED:** Finished

**Example:**
```java
Thread t = new Thread(() -> {});
System.out.println(t.getState()); // NEW
t.start();
System.out.println(t.getState()); // RUNNABLE/BLOCKED/WAITING depending on timing
```

---

## 5. Thread Methods & Utilities

| Method                | Description                                         |
|-----------------------|-----------------------------------------------------|
| `start()`             | Starts thread execution, invokes `run()`           |
| `run()`               | Entry point of thread code                         |
| `sleep(ms)`           | Pauses thread for specified time                   |
| `join()`              | Waits for thread to die                            |
| `interrupt()`         | Interrupts a thread                                |
| `isAlive()`           | Checks if thread is alive                          |
| `setPriority(int)`    | Sets thread priority (1-10, default: 5)            |
| `setDaemon(boolean)`  | Makes thread daemon                                |
| `yield()`             | Hints to scheduler to let other threads run        |
| `currentThread()`     | Returns current thread                             |

**Example:**
```java
Thread t = new Thread(() -> {
    try { Thread.sleep(1000); } catch (InterruptedException e) {}
    System.out.println("Done!");
});
t.start();
t.join(); // Main thread waits for t to finish
```

**Explanation:**  
- `sleep()` pauses thread, but doesn't release lock if in synchronized block.
- `interrupt()` sets the interrupt flag; check via `Thread.interrupted()`.

---

## 6. Thread Communication (wait, notify, notifyAll)

**Why:**  
Coordinate shared resource access (e.g., producer-consumer).

**Example:**  
```java
class Shared {
    private int data;
    private boolean available = false;

    public synchronized void put(int value) throws InterruptedException {
        while (available) wait();
        data = value;
        available = true;
        notifyAll();
    }

    public synchronized int get() throws InterruptedException {
        while (!available) wait();
        available = false;
        notifyAll();
        return data;
    }
}
```
**Explanation:**  
- Use `wait()` to suspend and release lock until another thread calls `notify()`/`notifyAll()`.
- Always call inside a loop to handle spurious wakeups.

---

## 7. Synchronization

### 7.1. Synchronized Methods

```java
public synchronized void increment() {
    count++;
}
```
**Explanation:**  
Whole method is locked on the instance (`this`).

---

### 7.2. Synchronized Blocks

```java
public void increment() {
    synchronized(this) {
        count++;
    }
}
```
**Explanation:**  
Only the block is locked; more granular control, better performance.

---

### 7.3. Class-level Lock

```java
synchronized (MyClass.class) {
    // code
}
```
**Explanation:**  
Locks the class object, useful for static data.

---

## 8. Volatile Keyword

**Purpose:**  
Ensures visibility of variable changes across threads.

```java
private volatile boolean running = true;
```

**Example:**  
```java
volatile boolean running = true;
Thread t = new Thread(() -> {
    while (running) {
        // do work
    }
});
t.start();
Thread.sleep(1000);
running = false; // Thread t sees the update immediately
```

**Explanation:**  
Does NOT provide atomicity; just visibility.

---

## 9. Advanced Synchronization - Locks

**From `java.util.concurrent.locks`**

- **ReentrantLock:** Offers tryLock, timed, interruptible locks, fairness.
- **ReadWriteLock:** Multiple readers or one writer.

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

**TryLock Example:**
```java
if (lock.tryLock(100, TimeUnit.MILLISECONDS)) {
    try { /* critical section */ } finally { lock.unlock(); }
} else {
    // couldn't acquire lock
}
```

---

## 10. Atomic Variables

- Classes like `AtomicInteger`, `AtomicBoolean` offer atomic operations.

**Example:**  
```java
AtomicInteger count = new AtomicInteger(0);
count.incrementAndGet(); // Thread-safe increment
```

**Explanation:**  
No need for explicit synchronization for simple operations.

---

## 11. Deadlock, Starvation, and Livelock

### Deadlock Example:
```java
class A { synchronized void foo(B b) {...} }
class B { synchronized void bar(A a) {...} }
```
- Thread 1: a.foo(b)
- Thread 2: b.bar(a)
- Both threads hold one lock, waiting for the other: DEADLOCK.

**Avoid:**  
- Lock ordering, lock timeouts, use higher-level APIs.

### Starvation Example:
- Low priority thread never gets CPU due to high priority threads.

### Livelock Example:
- Threads keep yielding to each other, but no progress.

---

## 12. Daemon Threads

**Example:**
```java
Thread t = new Thread(() -> {
    while (true) { /* background work */ }
});
t.setDaemon(true);
t.start();
```
- JVM exits when only daemon threads remain.

**Explanation:**  
Used for background tasks (e.g., garbage collector).

---

## 13. Thread Priorities

```java
Thread t = new Thread(() -> {});
t.setPriority(Thread.MAX_PRIORITY); // 10
```
- Scheduler may prefer higher priority threads, but not guaranteed.

---

## 14. Thread Pools and Executor Framework

### 14.1. Why Thread Pools?

- Creating too many threads can exhaust system resources.
- Pools manage and reuse threads.

**Example:**
```java
ExecutorService pool = Executors.newFixedThreadPool(2);
for (int i = 0; i < 5; i++) {
    pool.submit(() -> System.out.println(Thread.currentThread().getName()));
}
pool.shutdown();
```

**Explanation:**  
Executor handles thread creation, queuing, scheduling, and shutdown.

---

### 14.2. ScheduledExecutorService

**Example:**
```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(
    () -> System.out.println("Tick: " + System.currentTimeMillis()),
    0, 2, TimeUnit.SECONDS
);
```
**Explanation:**  
Schedules tasks for future or periodic execution.

---

### 14.3. Callable and Future

**Example:**
```java
Callable<Integer> callable = () -> 123;
Future<Integer> future = pool.submit(callable);
System.out.println(future.get()); // 123
```
- `Future.get()` blocks until result is ready.

---

## 15. Fork/Join Framework (Java 7+)

**For recursive parallelism (e.g., quicksort, summing arrays).**

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
**Explanation:**  
Tasks are recursively split, work-stealing speeds up execution on multi-core CPUs.

---

## 16. Parallel Streams (Java 8+)

**Example:**
```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);
int sum = nums.parallelStream().mapToInt(Integer::intValue).sum();
```
**Explanation:**  
Parallelizes stream operations; easy parallelism for data processing.

---

## 17. Concurrent Collections

- `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`, `ConcurrentLinkedQueue`, etc.

**Example:**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("a", 1);
map.putIfAbsent("b", 2);
```
**Explanation:**  
Thread-safe, high performance, avoids global locking.

---

## 18. Blocking Queues (Producer-Consumer Pattern)

**Example:**
```java
BlockingQueue<Integer> queue = new LinkedBlockingQueue<>();

// Producer
new Thread(() -> {
    for (int i = 0; i < 5; i++) {
        try { queue.put(i); } catch (InterruptedException e) {}
    }
}).start();

// Consumer
new Thread(() -> {
    try {
        while (true) {
            Integer value = queue.take();
            System.out.println("Consumed: " + value);
        }
    } catch (InterruptedException e) {}
}).start();
```
**Explanation:**  
`put()` and `take()` block automatically, handling producer-consumer logic without explicit wait/notify.

---

## 19. ThreadLocal

**Example:**
```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
threadLocal.set(42);
System.out.println(threadLocal.get()); // Each thread sees its own value
```
**Explanation:**  
Used for per-thread data, e.g., user sessions, DB connections.

---

## 20. Immutability and Thread Safety

- **Immutable Objects:** Their state cannot change after construction (e.g., `String`, `Integer`).
- **Benefits:** Naturally thread-safe, no synchronization needed.
- **Example:**  
```java
final class Point {
    private final int x, y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    // No setters; object is immutable
}
```

---

## 21. Best Practices

- Use high-level concurrency utilities (`ExecutorService`, concurrent collections, etc).
- Minimize lock scope.
- Avoid manual thread management; prefer thread pools.
- Always shut down executors to free resources.
- Be cautious with shared mutable state.

---

## 22. Common Multithreading Interview Questions

1. Explain the difference between process and thread.
2. What is a race condition? How do you prevent it?
3. How does `volatile` differ from `synchronized`?
4. Why use thread pools?
5. Producer-consumer with `BlockingQueue`?
6. Use cases for `ThreadLocal`.
7. What is a deadlock? How do you prevent it?
8. Difference between `Callable` and `Runnable`.
9. What is the Fork/Join framework? Work-stealing?
10. How to increment a counter in a thread-safe way?
11. What are Java's concurrent collections?

---

## 23. Summary Table

| Concept             | Key Points / Example                              |
|---------------------|---------------------------------------------------|
| Thread Creation     | `Thread`, `Runnable`, Lambda, `Callable`          |
| Synchronization     | `synchronized`, `volatile`, Locks, Atomics        |
| Communication       | `wait()`, `notify()`, `notifyAll()`, BlockingQueue|
| Thread Pools        | Executors framework, fixed/cached/scheduled pools |
| Deadlock            | Multiple locks, lock ordering, tryLock            |
| Daemon Threads      | Background, JVM exits when only daemons           |
| ThreadLocal         | Per-thread storage, isolated from other threads   |
| Fork/Join           | Divide-and-conquer parallelism                    |
| Parallel Streams    | Easy parallelism for collections                  |
| Concurrent Coll.    | Thread-safe Maps, Queues, Lists                   |

---

## 24. Resources

- [Official Java Concurrency Tutorials](https://docs.oracle.com/javase/tutorial/essential/concurrency/)
- [Java Concurrency in Practice](https://jcip.net/)
- [Baeldung Java Concurrency Guide](https://www.baeldung.com/java-concurrency)
- [Java Concurrency Utilities](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/package-summary.html)

---

**This guide covers all essential Java multithreading concepts, with detailed explanations and practical code samples for each.**