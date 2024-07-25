# Multi Threading

- [Basics Reading](https://www.liaoxuefeng.com/wiki/1252599548343744/1304521607217185)

#### Tread Creation

##### 1. Extending `Thread` Class

- This approach is more straightforward but limits your ability to extend another class.

```java
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}

public class Main {
    public static void main(String[] args) {
        MyThread myThread = new MyThread();
        myThread.start();
    }
}
```

##### 2. Implementing the `Runnable` Interface

- This approach is similar to the first one but it supports multiple inheritance.

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread is running");
    }
}

public class Main {
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable); // Creating a new Thread through myRunnable
        thread.start();
    }
}
```

##### 3. Implementing `Callable` Interface

- The `Callable` interface in Java is similar to `Runnable`, but it can return a result and throw a checked exception. `Callable` is useful when you need the thread to return a value or throw an exception.
- It requires an `ExecutorService` or another mechanism to manage the execution of the `Callable` tasks in separate threads.
  - Thread doesn't have a constructor expecting a `Callable` argument, so we typically use `ExecutorService` to excute `Callable` tasks.

```java
class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        // Simulate some work
        Thread.sleep(1000);
        return "Thread completed!";
    }
}

public class Main {
    public static void main(String[] args) {
        // Create a Callable object
        MyCallable myCallable = new MyCallable();
        
        // Use an ExecutorService to manage the thread
        ExecutorService executorService = Executors.newSingleThreadExecutor();
        
        // Submit the Callable to the ExecutorService and get a Future object
        Future<String> future = executorService.submit(myCallable);
        
        try {
            // Get the result of the Callable (this will block until the Callable is done)
            String result = future.get();
            System.out.println(result);
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        
        // Shut down the ExecutorService
        executorService.shutdown();
    }
}
```

##### Questions:

- What is the difference between t.start() and t.run()?

  - t.start starts a new thread to excute the task(run())
  - t.run() excute the task in the current thread.

- Wht is the differecence between Callable and Runnbale?

  - runnable has no return value; You can use Thread to excute a runnable.

  - callable has return value; Thread class doesn't support callable.

- Can we use new Thread(lambda)? is it equal to implement Runnable? Why?

  - **Yes**
  - **Reason**: 
    - **Functional Interface**: `Runnable` is a functional interface with a single abstract method `run()`. A lambda expression can be used to provide an implementation of this method.
    - **Thread Target**: When you pass a lambda expression to the `Thread` constructor, it is treated as an instance of `Runnable` with the lambda providing the `run` method's implementation.

#### Interrupt, Deamon, Join

##### Interrupt

The `interrupt` method is used to signal a thread that it should stop what it is doing and do something else. But the interrupt signal in Java needs to **be handled explicitly**. When a thread is interrupted, it does not stop immediately; rather, an interrupt flag is set on the thread. It is up to the thread to check this flag and decide how to handle the interruption.

```java
public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                while (!Thread.currentThread().isInterrupted()) { // check the interrupt flag
                    System.out.println("Thread is running");
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                System.out.println("Thread was interrupted");
            }
        });

        thread.start();
        
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        thread.interrupt();
    }
}
```



##### Daemon

A daemon thread is a thread that does not prevent the JVM from exiting when the program finishes. They are typically used for background tasks such as garbage collection. You can make a thread a daemon by calling `setDaemon(true)` before the thread is started.

```java
public class Main {
    public static void main(String[] args) {
        Thread daemonThread = new Thread(() -> {
            while (true) {
                System.out.println("Daemon thread is running");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
				
      	// If Daemon is set to true, when the main thread finished, the process will exit.
      	// If Deamon is not set, when the main thread finished, the daemon thread will keep running.
        daemonThread.setDaemon(true); 
        daemonThread.start();

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Main thread is finished");
    }
}
```

##### Join

The `join` method allows one thread to wait for the completion of another thread. This is useful when you need to ensure that a thread has finished its execution before continuing with the next steps in the main thread or another thread.

```java
public class Main {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            System.out.println("Thread is running");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Thread completed");
        });

        thread.start();

        try {
            thread.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Main thread finished after child thread");
    }
}
```



#### Synchronized

In Java, the `synchronized` keyword is used to control access to **a block of code** or **method** by multiple threads. It ensures that only one thread can execute the synchronized block or method at a time, thus preventing race conditions and ensuring thread safety.

##### Synchronized Method

```java
class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

public class Main {
    public static void main(String[] args) {
        Counter counter = new Counter();

        // Creating multiple threads that call the synchronized method
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Count: " + counter.getCount());
    }
}
```

If `counter.increment()` method is not `sychoronized`, the final result would be less or equal than 2000.

##### Synchronized Block

A synchronized block is more fine-grained than a synchronized method. It allows you to synchronize only a portion of the method, potentially improving performance.

```java
class Counter {
    private int count = 0;
    private final Object lock = new Object();

    public void increment() {
        synchronized (lock) {
            count++;
        }
    }

    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Counter counter = new Counter();

        // Creating multiple threads that call the synchronized block
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        });

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Count: " + counter.getCount());
    }
}
```

- **Lock Object**: When using synchronized blocks, it's common practice to use a private final lock object. This prevents external code from locking on the same object, which could lead to deadlocks.

- **Static Methods**: For static methods, synchronization locks on the `Class` object of the class:

  ```java
  class Counter{
    public static synchronized void staticSyncMethod() {
        // synchronized code
    }
    // or
    public static void staticSyncMethod() {
        synchronized (Counter.class) {
            // synchronized code
        }
    }
  }
  ```

- **Reentrancy**: The synchronized keyword in Java is reentrant. If a thread that already holds the lock tries to acquire it again, it can proceed without being blocked.



#### DeadLock

##### Prerequisites to have a dead lock:

- Mutual Exclusion: At least one resource must be held in a non-shareable mode (be locked). That is, only one thread can use the resource at any given time.

- No preemption: A thread holding at least one resource is waiting to acquire additional resources that are currently being held by other threads.

- Hold and wait: Resources cannot be forcibly taken away from threads. A resource can only be released voluntarily by the thread holding it, after that thread has finished using it.

- Circular wait: There exists a set of threads $\{T1,T2,…,Tn\}$ such that $T_1$ is waiting for a resource held by $T_2$, $T_2$ is waiting for a resource held by $T_3$, and so on, with $T_n$ waiting for a resource held by $T_1$.

##### How to avoid deadlock

1. Lock Ordering

   - Establish a global order in which all threads must acquire locks. This **prevents circular wait** conditions, which are a common cause of deadlocks. In this example: to acquire resource2, you must acquire resource1 first.

     ```java
     class Resource {
         private final String name;
     
         public Resource(String name) {
             this.name = name;
         }
     
         public String getName() {
             return name;
         }
     }
     
     public class Main {
         private static final Resource resource1 = new Resource("Resource 1");
         private static final Resource resource2 = new Resource("Resource 2");
     
         public static void main(String[] args) {
             Thread t1 = new Thread(() -> {
                 synchronized (resource1) {
                     System.out.println("Thread 1: locked " + resource1.getName());
                     try { Thread.sleep(50); } catch (InterruptedException e) { e.printStackTrace(); }
                     synchronized (resource2) {
                         System.out.println("Thread 1: locked " + resource2.getName());
                     }
                 }
             });
     
             Thread t2 = new Thread(() -> {
                 synchronized (resource1) { // Acquire resource1 first
                     System.out.println("Thread 2: locked " + resource1.getName());
                     try { Thread.sleep(50); } catch (InterruptedException e) { e.printStackTrace(); }
                     synchronized (resource2) {
                         System.out.println("Thread 2: locked " + resource2.getName());
                     }
                 }
             });
     
             t1.start();
             t2.start();
         }
     }
     ```

2. Lock Timeout (`tryLock`)

   - Use the `tryLock` method with a timeout to avoid waiting indefinitely for a lock. This allows a thread to back off and try again later if it can't acquire all the required locks.

     ```java
     import java.util.concurrent.locks.Lock;
     import java.util.concurrent.locks.ReentrantLock;
     import java.util.concurrent.TimeUnit;
     
     public class Main {
         private static final Lock lock1 = new ReentrantLock();
         private static final Lock lock2 = new ReentrantLock();
     
         public static void main(String[] args) {
             Thread t1 = new Thread(() -> {
                 try {
                     if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
                         try {
                             System.out.println("Thread 1: locked lock1");
                             try { Thread.sleep(50); } catch (InterruptedException e) { e.printStackTrace(); }
                             if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
                                 try {
                                     System.out.println("Thread 1: locked lock2");
                                 } finally {
                                     lock2.unlock();
                                 }
                             } else {
                                 System.out.println("Thread 1: could not lock lock2");
                             }
                         } finally {
                             lock1.unlock();
                         }
                     } else {
                         System.out.println("Thread 1: could not lock lock1");
                     }
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
             });
     
             Thread t2 = new Thread(() -> {
                 try {
                     if (lock2.tryLock(100, TimeUnit.MILLISECONDS)) {
                         try {
                             System.out.println("Thread 2: locked lock2");
                             try { Thread.sleep(50); } catch (InterruptedException e) { e.printStackTrace(); }
                             if (lock1.tryLock(100, TimeUnit.MILLISECONDS)) {
                                 try {
                                     System.out.println("Thread 2: locked lock1");
                                 } finally {
                                     lock1.unlock();
                                 }
                             } else {
                                 System.out.println("Thread 2: could not lock lock1");
                             }
                         } finally {
                             lock2.unlock();
                         }
                     } else {
                         System.out.println("Thread 2: could not lock lock2");
                     }
                 } catch (InterruptedException e) {
                     e.printStackTrace();
                 }
             });
     
             t1.start();
             t2.start();
         }
     }
     ```

3. Avoid Holding Multiple Locks

   - Reduce the need to hold multiple locks at once. If possible, design your system to acquire and release locks in a way that avoids holding more than one lock simultaneously.

4. DeadLock Detection

   - Implement a mechanism to detect and recover from deadlocks. This is more complex and often involves monitoring the system for cycles in the resource allocation graph.

#### Notify and Wait

In Java, `wait()` and `notify()` are methods used for inter-thread communication. They are part of the `Object` class, meaning every object in Java can be used as a monitor that can call these methods

- **`wait()`**: This method causes the current thread to release the lock it holds on an object and enter the waiting state. The thread remains in the waiting state until another thread calls `notify()` or `notifyAll()` on the same object.
- **`notify()`**: This method wakes up a single thread that is waiting on the object's monitor. If multiple threads are waiting, one is chosen to be awakened.
- **`notifyAll()`**: This method wakes up all the threads that are waiting on the object's monitor.

```java
// Consumer - Producer 
import java.util.LinkedList;
import java.util.Queue;

class SharedResource {
    private final Queue<Integer> queue = new LinkedList<>();
    private final int MAX_SIZE = 5;

    public synchronized void produce(int value) throws InterruptedException {
        while (queue.size() == MAX_SIZE) {
            wait(); // Wait until there is space in the queue
        }
        queue.add(value);
        System.out.println("Produced: " + value);
        notify(); // Notify the consumer that new data is available
    }

    public synchronized void consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // Wait until there is data to consume
        }
        int value = queue.poll();
        System.out.println("Consumed: " + value);
        notify(); // Notify the producer that space is available in the queue
    }
}

class Producer implements Runnable {
    private final SharedResource resource;

    public Producer(SharedResource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        int value = 0;
        while (true) {
            try {
                resource.produce(value++);
                Thread.sleep(100); // Simulate time taken to produce
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

class Consumer implements Runnable {
    private final SharedResource resource;

    public Consumer(SharedResource resource) {
        this.resource = resource;
    }

    @Override
    public void run() {
        while (true) {
            try {
                resource.consume();
                Thread.sleep(150); // Simulate time taken to consume
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) {
        SharedResource resource = new SharedResource();
        Thread producerThread = new Thread(new Producer(resource));
        Thread consumerThread = new Thread(new Consumer(resource));

        producerThread.start();
        consumerThread.start();
    }
}
```



#### `ReentrantLock`

我们知道Java语言直接提供了 synchronized 关键字用于加锁，但这种锁一是很重，二是获取时必须一直等待，没有额外的尝试机制。
 `java.util.concurrent.locks` 包提供的 `ReentrantLock` 用于替代 `synchronized` 加锁

##### Key Features

###### **Reentrant**

 A thread that owns the lock can re-acquire it without any issue.

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final Lock lock = new ReentrantLock();

    public void criticalSection() {
        lock.lock(); // Acquire the lock
        try {
            // Critical section
            System.out.println("Thread " + Thread.currentThread().getName() + " is in critical section");
        } finally {
            lock.unlock(); // Ensure the lock is released
        }
    }

    public static void main(String[] args) {
        Main main = new Main();
        Runnable task = main::criticalSection;

        Thread t1 = new Thread(task, "T1");
        Thread t2 = new Thread(task, "T2");

        t1.start();
        t2.start();
    }
}

```

###### **Fairness**

 You can create a fair lock by passing `true` to the constructor, which ensures that threads acquire the lock in the order they requested it.

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final Lock lock = new ReentrantLock(true); // Fair lock

    public void criticalSection() {
        lock.lock();
        try {
            System.out.println("Thread " + Thread.currentThread().getName() + " is in critical section");
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        Main main = new Main();
        Runnable task = main::criticalSection;

        Thread t1 = new Thread(task, "T1");
        Thread t2 = new Thread(task, "T2");
        Thread t3 = new Thread(task, "T3");

        t1.start();
        t2.start();
        t3.start();
    }
}

```



###### **Interruptible Lock Acquisition**

You can interrupt a thread waiting to acquire a lock. `lock.lockInterruptbily()`

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final Lock lock = new ReentrantLock();

    public void criticalSection() {
        try {
            lock.lockInterruptibly(); // Acquire the lock interruptibly
            try {
                System.out.println("Thread " + Thread.currentThread().getName() + " is in critical section");
                Thread.sleep(1000);
            } finally {
                lock.unlock();
            }
        } catch (InterruptedException e) {
            System.out.println("Thread " + Thread.currentThread().getName() + " was interrupted");
        }
    }

    public static void main(String[] args) {
        Main main = new Main();
        Runnable task = main::criticalSection;

        Thread t1 = new Thread(task, "T1");
        Thread t2 = new Thread(task, "T2");

        t1.start();
        t2.start();

        // Interrupt t2 after some time
        try {
            Thread.sleep(200);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        t2.interrupt();
    }
}
```

###### **Timed Lock Attempts**

You can try to acquire a lock within a given time frame, avoiding indefinite blocking.

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final Lock lock = new ReentrantLock();

    public void criticalSection() {
        try {
            if (lock.tryLock(500, TimeUnit.MILLISECONDS)) { // Try to acquire the lock within 500 milliseconds
                try {
                    System.out.println("Thread " + Thread.currentThread().getName() + " acquired the lock");
                    Thread.sleep(1000);
                } finally {
                    lock.unlock();
                }
            } else {
                System.out.println("Thread " + Thread.currentThread().getName() + " could not acquire the lock");
            }
        } catch (InterruptedException e) {
            System.out.println("Thread " + Thread.currentThread().getName() + " was interrupted");
        }
    }

    public static void main(String[] args) {
        Main main = new Main();
        Runnable task = main::criticalSection;

        Thread t1 = new Thread(task, "T1");
        Thread t2 = new Thread(task, "T2");

        t1.start();
        t2.start();
    }
}
```

###### **Condition Variables**

 Allows multiple condition variables associated with a single lock, enabling more fine-grained control over thread communication.

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class Main {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private boolean conditionMet = false;

    public void awaitCondition() {
        lock.lock();
        try {
            while (!conditionMet) {
                System.out.println("Thread " + Thread.currentThread().getName() + " is waiting");
                condition.await();
            }
            System.out.println("Thread " + Thread.currentThread().getName() + " proceeds");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void signalCondition() {
        lock.lock();
        try {
            conditionMet = true;
            condition.signalAll();
            System.out.println("Condition met, all waiting threads are notified");
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        Main main = new Main();
        Runnable waitingTask = main::awaitCondition;
        Runnable signalingTask = main::signalCondition;

        Thread t1 = new Thread(waitingTask, "T1");
        Thread t2 = new Thread(waitingTask, "T2");
        Thread t3 = new Thread(signalingTask, "Signaler");

        t1.start();
        t2.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        t3.start();
    }
}
```

##### Questions

- What is the main difference between `synchronized` and `ReentrantLock`?
  - **`synchronized`**: Easier to use, with automatic lock management and intrinsic locks. Suitable for simple synchronization needs without advanced requirements like fairness or timed lock attempts.
  - **`ReentrantLock`**: More flexible and powerful, with features like fairness, timed lock attempts, and multiple condition variables. Suitable for complex synchronization needs where additional control and features are required. Allow interrupted locks.

#### **Read-Write-Lock**

It allows multiple threads to read a shared resource concurrently while ensuring that only one thread can write to the resource at a time. This is particularly useful in scenarios where reads are much more frequent than writes, as it improves concurrency and performance.

- **Read Lock**: Allows multiple threads to read the resource simultaneously.

- **Write Lock**: Allows only one thread to write to the resource, and blocks all other read and write operations.

#### **Stamped-Lock**

It provides an alternative to `ReentrantReadWriteLock` with a focus on improving the performance of read-heavy workloads. `StampedLock` supports three modes for accessing resources: Writing, Reading, and Optimistic Reading.

- **Write Lock**: Exclusive lock, similar to the write lock in `ReentrantReadWriteLock`.
- **Read Lock**: Shared lock, similar to the read lock in `ReentrantReadWriteLock`.
- **Optimistic Read Lock**: A non-blocking read lock that allows reading without acquiring the traditional read lock. It is faster but must be validated to ensure consistency.

**Performance**:

- `StampedLock` is designed for performance in read-heavy scenarios by minimizing contention between readers and writers.
- The optimistic read lock allows for faster reads by avoiding the overhead of acquiring and releasing traditional locks.

**Validation**:

- The optimistic read lock must be validated using the `validate` method to ensure the read was not invalidated by a concurrent write.
- If validation fails, the read operation must fall back to acquiring a traditional read lock.



#### **Concurrent Collections**

| interface | non-thread-safe         | thread-safe                              |
| --------- | ----------------------- | ---------------------------------------- |
| List      | ArrayList               | CopyOnWriteArrayList                     |
| Map       | HashMap                 | ConcurrentHashMap                        |
| Set       | HashSet / TreeSet       | CopyOnWriteArraySet                      |
| Queue     | ArrayDeque / LinkedList | ArrayBlockingQueue / LinkedBlockingQueue |
| Deque     | ArrayDeque / LinkedList | LinkedBlockingDeque                      |



#### Atomic

Java's `java.util.concurrent.atomic` package provides classes that support lock-free thread-safe programming on single variables. These classes include `AtomicBoolean`, `AtomicInteger`, `AtomicLong`, and `AtomicReference`. **They allow you to perform atomic operations on single variables without using explicit synchronization.**

```java
public class AtomicDemo {
    private static AtomicInteger atomicInteger = new AtomicInteger(1);
  
    public static void main(String[] args) {
        System.out.println(atomicInteger.getAndIncrement()); //count++ vs ++count
        System.out.println(atomicInteger.get());//count
	} 
}
```

##### Key Methods

- **incrementAndGet() / decrementAndGet()**: Atomically increments or decrements the current value by 1.
- **getAndIncrement() / getAndDecrement()**: Atomically increments or decrements the current value by 1, returning the previous value.
- **compareAndSet(expectedValue, newValue)**: Atomically sets the value to `newValue` if the current value equals the `expectedValue`.
- **get()**: Gets the current value.
- **set(newValue)**: Sets the value to `newValue`.



#### Thread Pool

Thread Pool is a collection of fixed number of threads. These threads remain active and are ready to excute asynchronous callbacks on behalf of the applications. The thread pool can efficiently manage the number of threads and the overhead of thread creation and destruction is minimized.

##### Cons of Thread Pool

1. New type of Deadlock:

   This happens in situation where all the executing threads are waiting for the results from a thread that is waiting in the queue but it doesn't have available worker thread to use.

2. Thread leakage:

   **Improper Task Handling**: If tasks submitted to the pool do not properly release resources or complete their execution, it can lead to thread leaks, where threads remain indefinitely in the pool without doing useful work. 

   In a thread pool, thread leakage can lead to low efficiency. 

3. Resource Trashing

   This refers to a state where the system spends more time managing threads (context switching) and competing for resources (such as CPU, memory, and I/O) than executing the actual tasks. This typically happens when there are too many active threads relative to the available system resources.

##### Steps to Use a Thread pool

1. Create a thread pool
2. submit tasks to the thread pool
3. Shut down the the thread pool

##### Use examples

`ExecutorService` is an interface, commonly used implementations are:

###### `FixedThreadPool`

The number of threads is fixed and can't change.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class FixedThreadPoolExample {
    public static void main(String[] args) {
        // Create a fixed thread pool with 3 threads
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        // Submit tasks to the thread pool
        for (int i = 0; i < 10; i++) {
            executorService.submit(() -> {
                System.out.println(Thread.currentThread().getName() + " is executing a task");
                try {
                    Thread.sleep(1000); // Simulate some work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        // Shut down the thread pool
        executorService.shutdown();
        try {
            // Wait for all tasks to complete before continuing
            if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
                executorService.shutdownNow(); // Force shutdown if tasks are not completed
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
        }

        System.out.println("All tasks are completed");
    }
}
```

###### `CachedThreadPool`

A cached thread pool creates new threads as needed but reuses previously constructed threads when they are available. If a thread has been idle for 60 seconds, it is terminated and removed from the pool.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class CachedThreadPoolExample {
    public static void main(String[] args) {
        // Create a cached thread pool
        ExecutorService executorService = Executors.newCachedThreadPool();

        // Submit tasks to the thread pool
        for (int i = 0; i < 10; i++) {
            executorService.submit(() -> {
                System.out.println(Thread.currentThread().getName() + " is executing a task");
                try {
                    Thread.sleep(1000); // Simulate some work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        // Shut down the thread pool
        executorService.shutdown();
        try {
            // Wait for all tasks to complete before continuing
            if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
                executorService.shutdownNow(); // Force shutdown if tasks are not completed
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
        }

        System.out.println("All tasks are completed");
    }
}
```

###### `SingleThreadExecutor`

A single thread executor uses a single worker thread to execute tasks sequentially. If the single thread terminates due to a failure during execution, a new one will take its place.

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class SingleThreadExecutorExample {
    public static void main(String[] args) {
        // Create a single thread executor
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        // Submit tasks to the thread pool
        for (int i = 0; i < 5; i++) {
            executorService.submit(() -> {
                System.out.println(Thread.currentThread().getName() + " is executing a task");
                try {
                    Thread.sleep(1000); // Simulate some work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        // Shut down the thread pool
        executorService.shutdown();
        try {
            // Wait for all tasks to complete before continuing
            if (!executorService.awaitTermination(60, TimeUnit.SECONDS)) {
                executorService.shutdownNow(); // Force shutdown if tasks are not completed
            }
        } catch (InterruptedException e) {
            executorService.shutdownNow();
        }

        System.out.println("All tasks are completed");
    }
}
```

##### Questions

1. What's the difference between `SingleThreadExecutor` and `FixedThreadPool` with a thread size 1?

   - `newSingleThreadExecutor()`: Automatically replaces the thread if it terminates unexpectedly, ensuring continuous execution of tasks. Ensures that there is always an active thread to execute tasks.

   - `newFixedThreadPool(1)`: Does not replace the thread if it terminates due to a failure. The pool stays in a failed state. If the single thread terminates, it does not automatically get replaced, and the pool needs manual intervention to restart.

2. How many ways we can create a thread for task?
   - `new Thread(new Task("name"))` Task could be a`Runnable`
   - `es.submit(new Task("name"))` Task could be a `Runnable` or `Callable`

#### Future

A `Future` in Java is an interface that represents the result of an asynchronous computation. The return type of `es.submit` method is a `Future`.  

##### Key Methods of the Future Interface

- **`get()`**:
  - Waits if necessary for the computation to complete and then retrieves its result.
  - Throws `InterruptedException`, `ExecutionException`, or `CancellationException`.
- **`get(long timeout, TimeUnit unit)`**:
  - Waits for at most the given time for the computation to complete and then retrieves its result, if available.
  - Throws `TimeoutException` in addition to the exceptions mentioned above.
- **`isDone()`**:
  - Returns `true` if the computation is complete.

- **`isCancelled()`**:
  - Returns `true` if the computation was cancelled before it completed.

- **`cancel(boolean mayInterruptIfRunning)`**:
  - Attempts to cancel the computation. `mayInterruptIfRunning` specifies whether the thread executing the task should be interrupted.
