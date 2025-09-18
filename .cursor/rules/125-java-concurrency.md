---
author: Juan Antonio Breña Moral
version: 0.11.0-SNAPSHOT
---
# Java rules for Concurrency objects

## Role

You are a Senior software engineer with extensive experience in Java software development

## Goal

Effective Java concurrency relies on understanding thread safety fundamentals, using `java.util.concurrent` utilities, and managing thread pools with `ExecutorService`. Key practices include implementing concurrent design patterns like Producer-Consumer, leveraging `CompletableFuture` for asynchronous tasks, and ensuring thread safety through immutability and safe publication. Performance aspects like lock contention and memory consistency must be considered. Thorough testing, including stress tests and thread dump analysis, is crucial. Modern Java offers virtual threads for enhanced scalability, structured concurrency for simplified task management, and scoped values for safer thread-shared data as alternatives to thread-locals.

### Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Master Thread Safety Fundamentals**: Understand and correctly apply core concepts such as synchronization (locks, conditions), atomic operations (`java.util.concurrent.atomic`), thread-safe collections (`java.util.concurrent`), immutability, and the Java Memory Model to ensure data integrity and prevent race conditions or deadlocks.
2.  **Efficient Thread Pool Management**: Utilize `ExecutorService` for robust thread management. Choose appropriate thread pool implementations and configure them with suitable sizing, keep-alive times, queue capacities, and rejection policies based on the application's workload. Implement graceful shutdown procedures.
3.  **Leverage Concurrent Design Patterns**: Implement established patterns like Producer-Consumer (using `BlockingQueue`) and Publish-Subscribe to structure concurrent applications effectively, promoting decoupling, scalability, and maintainability.
4.  **Embrace Asynchronous Programming with `CompletableFuture`**: Employ `CompletableFuture` to compose and manage asynchronous computations in a non-blocking way. Chain dependent tasks, combine results from multiple futures, and handle exceptions gracefully to build responsive and efficient applications.
5.  **Prioritize Immutability and Safe Publication**: Design classes to be immutable whenever feasible to inherently achieve thread safety. Ensure that shared mutable objects are safely published (e.g., via `volatile`, static initializers, or proper synchronization) so that their state is consistently visible to all threads.
6.  **Optimize for Performance, Considering Concurrency overheads**: Be mindful of performance implications such as lock contention (minimize scope, use finer-grained locks), memory consistency (understand happens-before, use `volatile` where appropriate), context switching overhead (size thread pools carefully), and potential issues like false sharing.
7.  **Thorough Testing and Debugging**: Rigorously test concurrent code. This includes unit tests for thread-safe components, integration tests for interactions, and stress tests to reveal race conditions or deadlocks. Utilize thread dump analysis, proper logging, and concurrency testing tools.
8.  **Adopt Modern Java Concurrency Features for Enhanced Development**:
*   **Virtual Threads (Project Loom)**: Embrace virtual threads via `Executors.newVirtualThreadPerTaskExecutor()` for I/O-bound tasks to dramatically increase scalability with minimal resource overhead. Avoid pooling virtual threads.
*   **Structured Concurrency**: Use `StructuredTaskScope` to simplify the management of multiple related concurrent tasks as a single unit of work, improving error handling, cancellation, and resource management.
*   **Scoped Values**: Prefer `ScopedValue` over `ThreadLocal` for sharing immutable data robustly and efficiently across tasks within a dynamically bounded scope, especially when working with virtual threads.
9.  **Cooperative Cancellation and Interruption Discipline**: Design tasks to be cancellable; always respond to interruption promptly. Do not swallow `InterruptedException`; either propagate it or restore the interrupt flag with `Thread.currentThread().interrupt()`. Prefer time-bounded operations (`orTimeout`, `completeOnTimeout`, timeouts on blocking calls), use `Future.cancel(true)`, prefer `Lock.lockInterruptibly()`/`tryLock(timeout, unit)` where applicable, and ensure cleanup on cancellation.
10. **Backpressure and Overload Protection**: Prevent unbounded work queues and cascading failures by using bounded queues, appropriate rejection policies (e.g., `CallerRunsPolicy` for graceful shedding), semaphores/bulkheads to cap concurrency, request rate limiting, and the `Flow` (Reactive Streams) API when stream backpressure is needed.
11. **Deadlock Avoidance and Lock Hygiene**: Establish global lock ordering, minimize lock scope, avoid holding locks while calling out to untrusted code, favor non-blocking algorithms or `tryLock` with timeouts where practical, and avoid nested synchronization across unrelated components.
12. **Correct Use of ForkJoin and Parallel Streams**: Reserve ForkJoin/parallel streams for CPU-bound, short-lived, side-effect-free tasks. Avoid blocking I/O within the common pool; if blocking is unavoidable, use `ForkJoinPool.ManagedBlocker` or a dedicated executor. Do not rely on `parallelStream()` in request-scoped code paths unless measured and justified.
13. **Avoid Virtual-Thread Pinning**: With virtual threads, avoid holding intrinsic locks (`synchronized`) around blocking calls that may pin to a carrier thread. Prefer `ReentrantLock` (which cooperates with parking), keep critical sections small, and use JFR (VirtualThreadPinned) to detect pinning hot spots.
14. **Observability for Concurrency**: Name threads consistently, set `UncaughtExceptionHandler`s, expose metrics (queue depths, pool sizes, task latencies, rejection counts), propagate contextual data with `ScopedValue` instead of `ThreadLocal`, and instrument with Thread dumps and JFR for diagnosis.
15. **Timeouts, Retries, and Idempotency**: Always bound remote calls with timeouts; implement bounded, jittered retries where appropriate and ensure operations are idempotent to avoid duplicate side effects under retries/cancellation.
16. **Use Fit-for-Purpose Concurrency Primitives**: Prefer `LongAdder/LongAccumulator` under high contention counters, `CopyOnWriteArrayList` for read-mostly observer lists, `StampedLock`/`ReadWriteLock` for read-heavy data, and high-level utilities (`Semaphore`, `CountDownLatch`, `Phaser`) where they model coordination more clearly than manual `wait/notify`.

## Constraints

Before applying any recommendations, ensure the project is in a valid state by running Maven compilation. Compilation failure is a BLOCKING condition that prevents any further processing.

- **MANDATORY**: Run `./mvnw compile` or `mvn compile` before applying any change
- **PREREQUISITE**: Project must compile successfully and pass basic validation checks before any optimization
- **CRITICAL SAFETY**: If compilation fails, IMMEDIATELY STOP and DO NOT CONTINUE with any recommendations
- **BLOCKING CONDITION**: Compilation errors must be resolved by the user before proceeding with any object-oriented design improvements
- **NO EXCEPTIONS**: Under no circumstances should design recommendations be applied to a project that fails to compile

## Examples

### Table of contents

- Example 1: Thread Safety Fundamentals
- Example 2: Thread Pool Management
- Example 3: Concurrent Design Patterns
- Example 4: Asynchronous Programming with CompletableFuture
- Example 5: Thread Safety Guidelines (Immutability & Safe Publication)
- Example 6: Performance Considerations in Concurrency
- Example 7: Testing and Debugging Concurrent Code
- Example 8: Embrace Virtual Threads for Enhanced Scalability
- Example 9: Simplify Concurrent Code with Structured Concurrency
- Example 10: Manage Thread-Shared Data with Scoped Values
- Example 11: Cooperative Cancellation and Interruption
- Example 12: Overload Protection and Backpressure
- Example 13: ForkJoin and ManagedBlocker for Blocking Operations
- Example 14: Avoid Pinning with Virtual Threads
- Example 15: Phased Execution with Phaser
- Example 16: Synchronization with CyclicBarrier
- Example 17: Data Exchange with Exchanger

### Example 1: Thread Safety Fundamentals

Title: Understand and Apply Core Thread Safety Concepts
Description: Ensure data integrity and correct behavior in multi-threaded environments by using thread-safe data structures and appropriate synchronization mechanisms. - Prefer `java.util.concurrent` collections over older synchronized wrappers. - Utilize immutable objects to eliminate risks of concurrent modification. - Employ thread-local variables for state confined to a single thread. - Use atomic classes (`java.util.concurrent.atomic`) for lock-free operations on single variables. - Choose flexible locking with `ReentrantLock` or `ReadWriteLock` for more complex scenarios. - Favor `java.util.concurrent` utilities over manual `wait()/notify()`.

**Good example:**

```java
import java.util.Map;
import java.util.Queue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.concurrent.atomic.AtomicReference;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

// Dummy classes for context
class Task {}
class Event {}
class State {
    public State(String initialState) {} // Constructor for initial state
}
class UserContext {}
class Item {}

class ThreadSafetyExamples {
    // Preferred concurrent collections
    Map<String, String> concurrentMap = new ConcurrentHashMap<>();
    Queue<Task> taskQueue = new ConcurrentLinkedQueue<>();
    BlockingQueue<Event> eventQueue = new LinkedBlockingQueue<>();

    // Atomic variables
    AtomicInteger counter = new AtomicInteger(0);
    AtomicReference<State> stateRef = new AtomicReference<>(new State("initial"));

    // Thread-local storage
    private static final ThreadLocal<UserContext> userContext =
        ThreadLocal.withInitial(UserContext::new);

    // Using ReentrantLock
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition(); // Example condition
    private int itemCount = 0; // Example shared resource
    private final int MAX_ITEMS = 10; // Example capacity

    private boolean isFull() {
        return itemCount >= MAX_ITEMS;
    }
     private boolean isEmpty() { // Example helper
        return itemCount <= 0;
    }


    public void addItem(Item item) throws InterruptedException { // Added throws for await
        lock.lock();
        try {
            while (isFull()) {
                System.out.println("Queue is full, waiting to add item...");
                notFull.await(); // Wait if queue is full
            }
            // Add item logic here
            itemCount++;
            System.out.println("Item added. Count: " + itemCount);
            // Potentially signal other conditions (e.g., notEmpty.signalAll())
        } finally {
            lock.unlock();
        }
    }

    // Using ReadWriteLock
    private final ReadWriteLock rwLock = new ReentrantReadWriteLock();
    private final Lock readLock = rwLock.readLock();
    private final Lock writeLock = rwLock.writeLock();
    private String sharedData = "Initial Data";

    public String readData() {
        readLock.lock();
        try {
            return sharedData;
        } finally {
            readLock.unlock();
        }
    }

    public void writeData(String data) {
        writeLock.lock();
        try {
            sharedData = data;
            System.out.println("Data written: " + data);
        } finally {
            writeLock.unlock();
        }
    }
}
```

**Bad example:**

```java
// AVOID: Poor thread safety practices
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class UnsafeCounter {
    // BAD: Using non-thread-safe collections
    private Map<String, String> unsafeMap = new HashMap<>();
    private List<String> unsafeList = new ArrayList<>();

    // BAD: Using plain int without synchronization
    private int counter = 0;
    private String status = "INIT";

    public void incrementCounter() {
        // RACE CONDITION: Multiple threads can read same value
        counter++; // Not atomic - can lose updates
        unsafeMap.put("lastCount", String.valueOf(counter)); // Can corrupt map
    }

    // BAD: Inconsistent synchronization
    public synchronized void updateCounter(int value) {
        counter = value; // Synchronized
    }

    public int getCounter() {
        return counter; // NOT synchronized - can read stale value
    }

    // BAD: Synchronizing on mutable object
    private String lockObject = "lock";

    public void badSynchronization() {
        synchronized (lockObject) { // WRONG: string can be changed
            // Critical section
        }
        lockObject = "newLock"; // Now synchronization is broken!
    }
}
```

### Example 2: Thread Pool Management

Title: Manage Thread Pools Effectively with ExecutorService
Description: Utilize ExecutorService for robust thread management. Choose appropriate thread pool implementations, configure them properly, and implement graceful shutdown procedures.

**Good example:**

```java
// GOOD: Proper thread pool management
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadPoolManager {
    private final ExecutorService fixedPool;
    private final ScheduledExecutorService scheduler;
    private final ThreadPoolExecutor customPool;

    public ThreadPoolManager() {
        // Fixed thread pool with named threads
        this.fixedPool = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors(),
            new CustomThreadFactory("worker")
        );

        // Scheduled thread pool for periodic tasks
        this.scheduler = Executors.newScheduledThreadPool(
            1,
            new CustomThreadFactory("scheduler")
        );

        // Custom thread pool with fine-grained control
        this.customPool = new ThreadPoolExecutor(
            2,                                    // core pool size
            4,                                    // maximum pool size
            60L,                                  // keep alive time
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(100),       // bounded queue
            new CustomThreadFactory("custom"),
            new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
        );
    }

    public void submitTask(Runnable task) {
        fixedPool.submit(task);
    }

    public void schedulePeriodicTask(Runnable task, long period) {
        scheduler.scheduleAtFixedRate(task, 0, period, TimeUnit.SECONDS);
    }

    public void shutdown() {
        shutdownExecutorService(fixedPool, "FixedPool");
        shutdownExecutorService(scheduler, "Scheduler");
        shutdownExecutorService(customPool, "CustomPool");
    }

    private void shutdownExecutorService(ExecutorService executor, String name) {
        executor.shutdown(); // Disable new tasks
        try {
            // Wait for existing tasks to terminate
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow(); // Cancel currently executing tasks

                // Wait for tasks to respond to being cancelled
                if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                    System.err.println("Pool " + name + " did not terminate");
                }
            }
        } catch (InterruptedException ie) {
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }

    // Custom thread factory for better thread naming and error handling
    private static class CustomThreadFactory implements ThreadFactory {
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        CustomThreadFactory(String namePrefix) {
            this.namePrefix = namePrefix + "-thread-";
        }

        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, namePrefix + threadNumber.getAndIncrement());
            t.setDaemon(false);
            t.setUncaughtExceptionHandler((thread, ex) -> {
                System.err.println("Thread " + thread.getName() + " threw exception: " + ex);
            });
            return t;
        }
    }
}
```

**Bad example:**

```java
// AVOID: Poor thread pool management
import java.util.concurrent.Executors;
import java.util.concurrent.ExecutorService;

public class PoorThreadPoolManager {
    // BAD: Unbounded thread pools can cause resource exhaustion
    private ExecutorService cachedPool = Executors.newCachedThreadPool();

    // BAD: Single thread with unbounded queue
    private ExecutorService singleThread = Executors.newSingleThreadExecutor();

    public void submitTask(Runnable task) {
        // BAD: No error handling or resource management
        cachedPool.submit(task);
    }

    public void submitManyTasks() {
        for (int i = 0; i < 10000; i++) {
            // BAD: Can overwhelm the system
            cachedPool.submit(() -> {
                try {
                    Thread.sleep(10000); // Long-running task
                } catch (InterruptedException e) {
                    // BAD: Ignoring interruption
                }
            });
        }
    }

    // BAD: No proper shutdown
    public void shutdown() {
        cachedPool.shutdown(); // What if tasks don't finish?
        singleThread.shutdown();
        // No waiting for termination
        // No handling of interrupted exception
        // No forced shutdown if graceful shutdown fails
    }

    // BAD: Creating new thread for each task
    public void executeTask(Runnable task) {
        new Thread(task).start(); // Expensive and uncontrolled
    }

    // BAD: No thread naming or error handling
    // Default thread names are not descriptive
    // Uncaught exceptions terminate threads silently
}
```

### Example 3: Concurrent Design Patterns

Title: Implement Producer-Consumer and Publish-Subscribe
Description: Leverage established patterns like Producer-Consumer and Publish-Subscribe to structure concurrent applications effectively, promoting decoupling, scalability, and maintainability.

**Good example:**

```java
// GOOD: Producer-Consumer pattern with BlockingQueue
import java.util.Set;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

// Producer-Consumer implementation
public class ProducerConsumerExample {
    private final BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);
    private final ExecutorService executor = Executors.newFixedThreadPool(4);
    private volatile boolean running = true;

    public void startProcessing() {
        // Start multiple consumers
        for (int i = 0; i < 2; i++) {
            executor.submit(this::consumer);
        }
    }

    public void produce(Task task) throws InterruptedException {
        if (running) {
            queue.put(task); // Blocks if queue is full
        }
    }

    private void consumer() {
        while (running || !queue.isEmpty()) {
            try {
                Task task = queue.poll(1, TimeUnit.SECONDS);
                if (task != null) {
                    processTask(task);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }

    private void processTask(Task task) {
        System.out.println("Processing: " + task + " on " + Thread.currentThread().getName());
        // Simulate work
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    public void shutdown() {
        running = false;
        executor.shutdown();
    }

    private static class Task {
        private final String data;
        public Task(String data) { this.data = data; }
        @Override public String toString() { return "Task(" + data + ")"; }
    }
}

// Publish-Subscribe implementation
public class EventBus {
    private final ConcurrentHashMap<String, Set<EventListener>> listeners = new ConcurrentHashMap<>();
    private final ExecutorService notificationExecutor = ForkJoinPool.commonPool();

    public void subscribe(String topic, EventListener listener) {
        listeners.computeIfAbsent(topic, k -> ConcurrentHashMap.newKeySet())
                 .add(listener);
    }

    public void unsubscribe(String topic, EventListener listener) {
        Set<EventListener> topicListeners = listeners.get(topic);
        if (topicListeners != null) {
            topicListeners.remove(listener);
        }
    }

    public void publish(String topic, Event event) {
        Set<EventListener> topicListeners = listeners.get(topic);
        if (topicListeners != null && !topicListeners.isEmpty()) {
            // Notify listeners asynchronously
            topicListeners.forEach(listener ->
                notificationExecutor.submit(() -> {
                    try {
                        listener.onEvent(event);
                    } catch (Exception e) {
                        System.err.println("Error notifying listener: " + e.getMessage());
                    }
                })
            );
        }
    }

    private static class Event {
        private final String data;
        public Event(String data) { this.data = data; }
        @Override public String toString() { return "Event(" + data + ")"; }
    }

    @FunctionalInterface
    private interface EventListener {
        void onEvent(Event event);
    }
}
```

**Bad example:**

```java
// AVOID: Poor concurrent pattern implementation
import java.util.ArrayList;
import java.util.List;

public class BadProducerConsumer {
    // BAD: Using non-thread-safe collection
    private List<String> tasks = new ArrayList<>();
    private boolean running = true;

    public void produce(String task) {
        // RACE CONDITION: Multiple producers can corrupt the list
        synchronized (this) {
            tasks.add(task);
            notify(); // BAD: Should use notifyAll()
        }
    }

    public void consume() {
        while (running) {
            String task = null;
            synchronized (this) {
                while (tasks.isEmpty() && running) {
                    try {
                        wait(); // BAD: Can miss notifications
                    } catch (InterruptedException e) {
                        // BAD: Not handling interruption properly
                        return;
                    }
                }
                if (!tasks.isEmpty()) {
                    task = tasks.remove(0); // BAD: Inefficient removal from front
                }
            }

            if (task != null) {
                // BAD: Processing inside synchronized block would be even worse
                processTask(task);
            }
        }
    }

    // BAD: No proper shutdown mechanism
    public void stop() {
        running = false; // Consumers might not wake up
    }
}

// BAD: Synchronous event handling
public class BadEventBus {
    private Map<String, List<EventListener>> listeners = new HashMap<>();

    public void subscribe(String topic, EventListener listener) {
        // BAD: Not thread-safe
        listeners.computeIfAbsent(topic, k -> new ArrayList<>()).add(listener);
    }

    public void publish(String topic, String event) {
        List<EventListener> topicListeners = listeners.get(topic);
        if (topicListeners != null) {
            // BAD: Synchronous notification blocks publisher
            for (EventListener listener : topicListeners) {
                try {
                    listener.onEvent(event);
                } catch (Exception e) {
                    // BAD: One failing listener affects others
                    throw new RuntimeException("Event handling failed", e);
                }
            }
        }
    }
}
```

### Example 4: Asynchronous Programming with CompletableFuture

Title: Compose Non-blocking Asynchronous Operations
Description: Employ CompletableFuture to compose and manage asynchronous computations in a non-blocking way. Chain dependent tasks, combine results from multiple futures, and handle exceptions gracefully.

**Good example:**

```java
// GOOD: CompletableFuture for asynchronous processing
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.stream.Collectors;

public class AsyncService {
    private final ExecutorService customExecutor = Executors.newFixedThreadPool(4);

    public CompletableFuture<String> processDataAsync(String input) {
        return CompletableFuture
            .supplyAsync(() -> validateInput(input), customExecutor)
            .thenApplyAsync(this::transformData, customExecutor)
            .thenApply(this::formatResult)
            .exceptionally(this::handleError)
            .whenComplete((result, ex) -> {
                if (ex != null) {
                    System.err.println("Processing failed for: " + input);
                } else {
                    System.out.println("Successfully processed: " + input);
                }
            });
    }

    public CompletableFuture<List<String>> processMultipleAsync(List<String> inputs) {
        List<CompletableFuture<String>> futures = inputs.stream()
            .map(this::processDataAsync)
            .collect(Collectors.toList());

        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList()))
            .exceptionally(ex -> {
                System.err.println("Batch processing failed: " + ex.getMessage());
                return List.of("ERROR");
            });
    }

    public CompletableFuture<String> combineResults(String input1, String input2) {
        CompletableFuture<String> future1 = processDataAsync(input1);
        CompletableFuture<String> future2 = processDataAsync(input2);

        return future1.thenCombine(future2, (result1, result2) ->
            "Combined: " + result1 + " + " + result2);
    }

    public CompletableFuture<String> getFirstSuccessful(List<String> inputs) {
        CompletableFuture<String>[] futures = inputs.stream()
            .map(this::processDataAsync)
            .toArray(CompletableFuture[]::new);

        return CompletableFuture.anyOf(futures)
            .thenApply(result -> (String) result);
    }

    private String validateInput(String input) {
        if (input == null || input.trim().isEmpty()) {
            throw new IllegalArgumentException("Input cannot be null or empty");
        }
        // Simulate validation work
        try { Thread.sleep(100); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        }
        return input.trim();
    }

    private String transformData(String input) {
        // Simulate transformation work
        try { Thread.sleep(200); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException(e);
        }
        return "transformed_" + input;
    }

    private String formatResult(String input) {
        return "[" + input + "]";
    }

    private String handleError(Throwable throwable) {
        System.err.println("Error occurred: " + throwable.getMessage());
        return "ERROR: " + throwable.getClass().getSimpleName();
    }

    public void shutdown() {
        customExecutor.shutdown();
        try {
            if (!customExecutor.awaitTermination(5, TimeUnit.SECONDS)) {
                customExecutor.shutdownNow();
            }
        } catch (InterruptedException e) {
            customExecutor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```

**Bad example:**

```java
// AVOID: Blocking operations and poor error handling
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class BadAsyncService {

    public String processDataBlocking(String input) {
        // BAD: Using CompletableFuture but blocking immediately
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            return processInput(input);
        });

        try {
            return future.get(); // BLOCKING! Defeats the purpose
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public List<String> processMultipleBlocking(List<String> inputs) {
        List<String> results = new ArrayList<>();

        // BAD: Sequential processing instead of parallel
        for (String input : inputs) {
            CompletableFuture<String> future = CompletableFuture.supplyAsync(() ->
                processInput(input));

            try {
                results.add(future.get()); // BLOCKING in loop
            } catch (Exception e) {
                // BAD: One failure stops everything
                throw new RuntimeException("Processing failed", e);
            }
        }

        return results;
    }

    public CompletableFuture<String> badErrorHandling(String input) {
        return CompletableFuture.supplyAsync(() -> {
            if ("fail".equals(input)) {
                throw new RuntimeException("Simulated failure");
            }
            return processInput(input);
        });
        // BAD: No error handling - exceptions will propagate
    }

    public void badChaining(String input) {
        // BAD: Not chaining properly, creating nested futures
        CompletableFuture<CompletableFuture<String>> nestedFuture =
            CompletableFuture.supplyAsync(() -> {
                return CompletableFuture.supplyAsync(() -> {
                    return processInput(input);
                });
            });

        // Now you have a nested CompletableFuture - hard to work with
    }

    // BAD: Resource leak - no executor shutdown
    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    public CompletableFuture<String> processWithLeakedExecutor(String input) {
        return CompletableFuture.supplyAsync(() -> processInput(input), executor);
        // BAD: Executor never gets shut down
    }

    private String processInput(String input) {
        try {
            Thread.sleep(1000); // Simulate work
        } catch (InterruptedException e) {
            // BAD: Not handling interruption properly
        }
        return "processed_" + input;
    }
}
```

### Example 5: Thread Safety Guidelines (Immutability & Safe Publication)

Title: Ensure Thread Safety through Immutability and Safe Publication
Description: Minimize concurrency risks by designing classes to be immutable and ensuring shared objects are safely published. - **Immutability**: - Make fields `final` whenever possible. - Ensure all fields are initialized during construction. - Do not provide setters for mutable state. - Use defensive copying for mutable components (like `List` or `Date`) passed into constructors or returned from getters, or use unmodifiable collections. - Consider using the builder pattern for complex immutable objects. - **Safe Publication**: - Ensure that shared objects are correctly published to other threads (e.g., by initializing them in static initializers, storing them in `volatile` fields, or using proper synchronization). - `java.util.concurrent` collections and atomics handle safe publication internally for their state.

**Good example:**

```java
// GOOD: Immutable class with safe publication
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

// Immutable class
public final class ImmutableValue {
    private final int value;
    private final List<String> items;

    public ImmutableValue(int value, List<String> items) {
        this.value = value;
        // Defensive copy to ensure immutability
        this.items = List.copyOf(items);
    }

    public int getValue() {
        return value;
    }

    public List<String> getItems() {
        return items; // Already immutable
    }
}

// Safe publication example
class SafePublicationExample {
    private static final Map<String, String> cache = new ConcurrentHashMap<>();

    public static String getData(String key) {
        return cache.computeIfAbsent(key, k -> {
            // Safe publication through ConcurrentHashMap
            return "Data for " + k;
        });
    }
}
```

**Bad example:**

```java
// AVOID: Mutable class with unsafe publication
import java.util.ArrayList;
import java.util.List;

public class MutableValue {
    private int value;
    private List<String> items; // Mutable field

    public MutableValue(int value, List<String> items) {
        this.value = value;
        this.items = items; // Direct reference - not safe
    }

    public List<String> getItems() {
        return items; // Returns mutable reference
    }

    // BAD: Unsafe publication
    public static MutableValue instance;

    public static void initialize() {
        // Unsafe publication - other threads might see partially constructed object
        instance = new MutableValue(42, new ArrayList<>());
    }
}
```

### Example 6: Performance Considerations in Concurrency

Title: Optimize Concurrent Code for Performance
Description: Be mindful of performance implications in concurrent applications. - **Lock Contention**: Reduce contention by narrowing lock scopes, using finer-grained locks (lock striping), or exploring optimistic locking or lock-free data structures. - **Memory Consistency**: Understand the Java Memory Model (JMM). Use `volatile` for visibility of single variables across threads. Be aware of happens-before relationships established by synchronization. - **Context Switching**: Excessive threads can lead to high context-switching overhead. Size thread pools appropriately. - **False Sharing**: Be aware of false sharing when mutable fields accessed by different threads reside on the same cache line.

**Good example:**

```java
// GOOD: Performance-optimized concurrent code
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.List;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

// Lock striping for better performance
class StripedMap<K, V> {
    private static final int STRIPE_COUNT = 16;
    private final List<Lock> stripes;
    private final List<Map<K, V>> buckets;

    public StripedMap() {
        stripes = new ArrayList<>(STRIPE_COUNT);
        buckets = new ArrayList<>(STRIPE_COUNT);
        for (int i = 0; i < STRIPE_COUNT; i++) {
            stripes.add(new ReentrantLock());
            buckets.add(new HashMap<>());
        }
    }

    private int getStripeIndex(K key) {
        return Math.abs(key.hashCode() % STRIPE_COUNT);
    }

    public V get(K key) {
        int index = getStripeIndex(key);
        Lock lock = stripes.get(index);
        lock.lock();
        try {
            return buckets.get(index).get(key);
        } finally {
            lock.unlock();
        }
    }

    public V put(K key, V value) {
        int index = getStripeIndex(key);
        Lock lock = stripes.get(index);
        lock.lock();
        try {
            return buckets.get(index).put(key, value);
        } finally {
            lock.unlock();
        }
    }
}

// Memory consistency with volatile
class MemoryConsistencyExample {
    private volatile boolean flag = false;
    private int data = 0;

    public void writer() {
        data = 42;
        flag = true; // Volatile write establishes happens-before
    }

    public void reader() {
        while (!flag) {
            // Spin wait
        }
        // Guaranteed to see data = 42
        System.out.println("Data: " + data);
    }
}
```

**Bad example:**

```java
// AVOID: Performance-harming concurrent code
import java.util.HashMap;
import java.util.Map;

public class PoorPerformance {
    private final Map<String, String> map = new HashMap<>();

    // BAD: Coarse-grained locking
    public synchronized String get(String key) {
        return map.get(key);
    }

    public synchronized void put(String key, String value) {
        map.put(key, value);
    }

    // BAD: Non-volatile field without synchronization
    private boolean flag = false;
    private int data = 0;

    public void writer() {
        data = 42;
        flag = true; // Other threads might not see this change
    }

    public void reader() {
        while (!flag) {
            // Might loop forever
        }
        // Might see stale data
        System.out.println("Data: " + data);
    }
}
```

### Example 7: Testing and Debugging Concurrent Code

Title: Rigorously Test and Debug Concurrent Applications
Description: Thoroughly test concurrent code using appropriate tools and techniques to identify race conditions, deadlocks, and other concurrency issues. - Write unit tests for thread-safe components with proper synchronization testing. - Use stress tests to reveal race conditions and deadlocks under load. - Implement integration tests for concurrent interactions between components. - Utilize thread dump analysis and profiling tools for debugging. - Apply proper logging strategies for concurrent environments. - Use testing frameworks like JUnit 5 with parallel execution capabilities.

**Good example:**

```java
// GOOD: Proper testing of concurrent code
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.RepeatedTest;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

class ConcurrentTestExample {

    @Test
    @RepeatedTest(100) // Run multiple times to catch race conditions
    void testThreadSafeCounter() throws InterruptedException {
        ThreadSafeCounter counter = new ThreadSafeCounter();
        int numberOfThreads = 10;
        int incrementsPerThread = 1000;

        ExecutorService executor = Executors.newFixedThreadPool(numberOfThreads);
        CountDownLatch latch = new CountDownLatch(numberOfThreads);

        for (int i = 0; i < numberOfThreads; i++) {
            executor.submit(() -> {
                try {
                    for (int j = 0; j < incrementsPerThread; j++) {
                        counter.increment();
                    }
                } finally {
                    latch.countDown();
                }
            });
        }

        assertTrue(latch.await(10, TimeUnit.SECONDS));
        executor.shutdown();

        assertEquals(numberOfThreads * incrementsPerThread, counter.getValue());
    }

    @Test
    void testProducerConsumer() throws InterruptedException {
        BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
        List<String> consumed = Collections.synchronizedList(new ArrayList<>());

        ExecutorService executor = Executors.newFixedThreadPool(4);
        CountDownLatch latch = new CountDownLatch(2);

        // Producer
        executor.submit(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    queue.put("item-" + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();
            }
        });

        // Consumer
        executor.submit(() -> {
            try {
                for (int i = 0; i < 5; i++) {
                    String item = queue.take();
                    consumed.add(item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            } finally {
                latch.countDown();
            }
        });

        assertTrue(latch.await(5, TimeUnit.SECONDS));
        executor.shutdown();
        assertEquals(5, consumed.size());
    }

    private static class ThreadSafeCounter {
        private final AtomicInteger count = new AtomicInteger(0);

        public void increment() {
            count.incrementAndGet();
        }

        public int getValue() {
            return count.get();
        }
    }
}
```

**Bad example:**

```java
// AVOID: Inadequate testing of concurrent code
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class PoorConcurrentTesting {

    @Test
    void testCounter() {
        // BAD: Only testing single-threaded scenario
        UnsafeCounter counter = new UnsafeCounter();
        counter.increment();
        assertEquals(1, counter.getValue());

        // This test passes but doesn't verify thread safety
    }

    @Test
    void testConcurrentAccess() throws InterruptedException {
        UnsafeCounter counter = new UnsafeCounter();

        // BAD: Creating threads but not waiting for completion
        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        }).start();

        new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                counter.increment();
            }
        }).start();

        // BAD: Not waiting for threads to complete
        Thread.sleep(100); // Unreliable timing

        // This assertion might pass or fail randomly
        // assertEquals(2000, counter.getValue());
    }

    private static class UnsafeCounter {
        private int count = 0;

        public void increment() {
            count++; // Not thread-safe
        }

        public int getValue() {
            return count;
        }
    }
}
```

### Example 8: Embrace Virtual Threads for Enhanced Scalability

Title: Use Virtual Threads for I/O-bound Tasks
Description: Leverage virtual threads (Project Loom) for I/O-bound tasks to dramatically increase scalability with minimal resource overhead. Avoid pooling virtual threads and use structured concurrency where appropriate.

**Good example:**

```java
// GOOD: Using virtual threads for scalable I/O operations
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.ScopedValue;
import java.util.concurrent.StructuredTaskScope;
import java.util.stream.IntStream;

public class VirtualThreadExample {

    // Use virtual thread executor for I/O-bound tasks
    private final ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

    public void handleManyIORequests() {
        List<Future<String>> futures = IntStream.range(0, 10000)
            .mapToObj(i -> virtualExecutor.submit(() -> performIOOperation("task-" + i)))
            .toList();

        // Collect results
        futures.forEach(future -> {
            try {
                String result = future.get();
                System.out.println("Completed: " + result);
            } catch (Exception e) {
                System.err.println("Task failed: " + e.getMessage());
            }
        });
    }

    // Virtual threads are perfect for blocking I/O operations
    private String performIOOperation(String taskId) {
        try {
            // Simulate I/O operation (database call, web request, etc.)
            Thread.sleep(1000); // This would block a platform thread
            return "Result for " + taskId;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return "Interrupted: " + taskId;
        }
    }

    // Using scoped values with virtual threads (Java 20+)
    private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();

    public void processWithScopedValue(String userId, List<String> tasks) {
        // Run with scoped value
        ScopedValue.where(USER_ID, userId)
            .run(() -> {
                tasks.parallelStream().forEach(task -> {
                    virtualExecutor.submit(() -> {
                        // Access scoped value safely
                        String currentUserId = USER_ID.get();
                        performTaskForUser(currentUserId, task);
                    });
                });
            });
    }

    private void performTaskForUser(String userId, String task) {
        System.out.println("Processing task " + task + " for user " + userId +
                          " on thread " + Thread.currentThread());
    }

    // Structured concurrency for managing related tasks
    public String fetchUserDataWithStructuredConcurrency(String userId) throws Exception {
        // Use static factory method instead of constructor
        try (var scope = StructuredTaskScope.open()) {

            Future<String> profile = scope.fork(() -> fetchUserProfile(userId));
            Future<String> preferences = scope.fork(() -> fetchUserPreferences(userId));
            Future<String> history = scope.fork(() -> fetchUserHistory(userId));

            scope.join();           // Wait for all tasks
            scope.throwIfFailed();  // Propagate any failures

            // All tasks completed successfully
            return combineUserData(profile.resultNow(),
                                 preferences.resultNow(),
                                 history.resultNow());
        }
    }

    private String fetchUserProfile(String userId) {
        // Simulate I/O operation
        try { Thread.sleep(100); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Profile for " + userId;
    }

    private String fetchUserPreferences(String userId) {
        try { Thread.sleep(150); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Preferences for " + userId;
    }

    private String fetchUserHistory(String userId) {
        try { Thread.sleep(200); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "History for " + userId;
    }

    private String combineUserData(String profile, String preferences, String history) {
        return String.format("User data: %s, %s, %s", profile, preferences, history);
    }

    public void shutdown() {
        virtualExecutor.shutdown();
    }
}
```

**Bad example:**

```java
// AVOID: Misusing virtual threads
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadLocal;

public class BadVirtualThreadUsage {

    // BAD: Creating thread pools for virtual threads
    private final ExecutorService virtualPool = Executors.newFixedThreadPool(100,
        Thread.ofVirtual().factory()); // Don't pool virtual threads!

    // BAD: Using virtual threads for CPU-intensive tasks
    public void performCPUIntensiveWork() {
        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

        for (int i = 0; i < 1000; i++) {
            virtualExecutor.submit(() -> {
                // BAD: Virtual threads are not suitable for CPU-bound work
                double result = 0;
                for (int j = 0; j < 1_000_000; j++) {
                    result += Math.sqrt(j) * Math.sin(j);
                }
                return result;
            });
        }
    }

    // BAD: Using platform thread patterns with virtual threads
    public void badResourceManagement() {
        // Creating virtual threads manually instead of using executor
        for (int i = 0; i < 10000; i++) {
            Thread.ofVirtual().start(() -> {
                performIOOperation();
                // BAD: No proper cleanup or error handling
            });
        }
    }

    // BAD: Blocking operations that shouldn't be used with virtual threads
    public void problematicBlocking() {
        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

        virtualExecutor.submit(() -> {
            try {
                synchronized (this) { // BAD: Synchronized blocks can pin virtual threads
                    Thread.sleep(1000); // This pins the virtual thread to platform thread
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
    }

    // BAD: Using ThreadLocal instead of ScopedValue
    private static final ThreadLocal<String> USER_CONTEXT = new ThreadLocal<>();

    public void badContextPropagation() {
        USER_CONTEXT.set("user123");

        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();
        virtualExecutor.submit(() -> {
            // BAD: ThreadLocal values don't propagate to virtual threads properly
            String userId = USER_CONTEXT.get(); // Likely null
            performTaskForUser(userId);
        });
    }

    private void performIOOperation() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    private void performTaskForUser(String userId) {
        System.out.println("Processing for user: " + userId);
    }
}
```

### Example 9: Simplify Concurrent Code with Structured Concurrency

Title: Manage Related Tasks as a Single Unit
Description: Use `StructuredTaskScope` to simplify the management of multiple related concurrent tasks as a single unit of work, improving error handling, cancellation, and resource management. - Use the static `StructuredTaskScope.open()` factory method instead of constructors. - Use `StructuredTaskScope.ShutdownOnFailure()` for fail-fast behavior. - Use `StructuredTaskScope.ShutdownOnSuccess()` for racing tasks. - Implement `Joiner.onTimeout()` method for custom timeout handling. - Ensure proper resource cleanup with try-with-resources. - Handle task results and exceptions appropriately. - Combine structured concurrency with virtual threads for optimal performance.

**Good example:**

```java
// GOOD: Structured concurrency for managing related tasks
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.Future;
import java.util.List;

class StructuredConcurrencyExample {

    // Fetch user data from multiple sources
    public UserData fetchCompleteUserData(String userId) throws Exception {
        // Use static factory method instead of constructor
        try (var scope = StructuredTaskScope.open()) {

            // Fork multiple related tasks
            Future<String> profileFuture = scope.fork(() -> fetchUserProfile(userId));
            Future<String> settingsFuture = scope.fork(() -> fetchUserSettings(userId));
            Future<String> ordersFuture = scope.fork(() -> fetchUserOrders(userId));

            // Wait for all tasks to complete or fail
            scope.join();
            scope.throwIfFailed();

            // All tasks completed successfully
            return new UserData(
                profileFuture.resultNow(),
                settingsFuture.resultNow(),
                ordersFuture.resultNow()
            );
        }
    }

    // Race multiple service calls and return first success
    public String fetchDataFromAnySource(String query) throws Exception {
        // Use static factory method for ShutdownOnSuccess
        try (var scope = StructuredTaskScope.open()) {

            // Fork competing tasks
            scope.fork(() -> fetchFromPrimaryService(query));
            scope.fork(() -> fetchFromSecondaryService(query));
            scope.fork(() -> fetchFromCacheService(query));

            // Wait for first successful completion
            scope.join();
            return scope.result();
        }
    }

    // Batch processing with structured concurrency
    public List<String> processBatch(List<String> items) throws Exception {
        // Use static factory method instead of constructor
        try (var scope = StructuredTaskScope.open()) {

            List<Future<String>> futures = items.stream()
                .map(item -> scope.fork(() -> processItem(item)))
                .toList();

            scope.join();
            scope.throwIfFailed();

            return futures.stream()
                .map(Future::resultNow)
                .toList();
        }
    }

    private String fetchUserProfile(String userId) {
        // Simulate I/O operation
        try { Thread.sleep(100); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Profile for " + userId;
    }

    private String fetchUserSettings(String userId) {
        try { Thread.sleep(150); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Settings for " + userId;
    }

    private String fetchUserOrders(String userId) {
        try { Thread.sleep(200); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Orders for " + userId;
    }

    private String fetchFromPrimaryService(String query) {
        try { Thread.sleep(300); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Primary result for: " + query;
    }

    private String fetchFromSecondaryService(String query) {
        try { Thread.sleep(200); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Secondary result for: " + query;
    }

    private String fetchFromCacheService(String query) {
        try { Thread.sleep(50); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Cached result for: " + query;
    }

    private String processItem(String item) {
        try { Thread.sleep(100); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Processed: " + item;
    }

    // Custom Joiner with timeout handling
    public String fetchWithTimeoutHandling(String query) throws Exception {
        try (var scope = StructuredTaskScope.open(new CustomTimeoutJoiner<String>())) {

            scope.fork(() -> {
                try { Thread.sleep(5000); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException(e);
                }
                return "Slow result for: " + query;
            });

            scope.fork(() -> {
                try { Thread.sleep(2000); } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    throw new RuntimeException(e);
                }
                return "Fast result for: " + query;
            });

            scope.join();
            return scope.result();
        }
    }

    // Custom Joiner implementation with timeout handling
    static class CustomTimeoutJoiner<T> implements StructuredTaskScope.Joiner<T, T> {
        private volatile T result;

        @Override
        public boolean onFork(StructuredTaskScope.Subtask<? extends T> subtask) {
            return true; // Continue with all subtasks
        }

        @Override
        public T onJoin() {
            return result;
        }

        @Override
        public T onTimeout() {
            // Provide default result when timeout occurs
            return (T) "Default result due to timeout";
        }

        @Override
        public boolean needsCompletion() {
            return result == null;
        }
    }

    record UserData(String profile, String settings, String orders) {}
}
```

**Bad example:**

```java
// AVOID: Manual task management without structured concurrency
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

class BadStructuredConcurrency {

    // BAD: Manual task management with resource leaks
    public UserData fetchUserDataManually(String userId) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        try {
            Future<String> profileFuture = executor.submit(() -> fetchUserProfile(userId));
            Future<String> settingsFuture = executor.submit(() -> fetchUserSettings(userId));
            Future<String> ordersFuture = executor.submit(() -> fetchUserOrders(userId));

            // BAD: No proper error handling
            return new UserData(
                profileFuture.get(),
                settingsFuture.get(),
                ordersFuture.get()
            );
        } catch (Exception e) {
            // BAD: Poor error handling
            throw new RuntimeException(e);
        } finally {
            // BAD: Improper shutdown
            executor.shutdown();
        }
    }

    // BAD: No cancellation when one task fails
    public String fetchFromAnySourceBadly(String query) {
        ExecutorService executor = Executors.newFixedThreadPool(3);

        try {
            Future<String> primary = executor.submit(() -> fetchFromPrimaryService(query));
            Future<String> secondary = executor.submit(() -> fetchFromSecondaryService(query));
            Future<String> cache = executor.submit(() -> fetchFromCacheService(query));

            // BAD: All tasks continue even if one succeeds
            while (true) {
                if (primary.isDone() && !primary.isCancelled()) {
                    return primary.get();
                }
                if (secondary.isDone() && !secondary.isCancelled()) {
                    return secondary.get();
                }
                if (cache.isDone() && !cache.isCancelled()) {
                    return cache.get();
                }
                Thread.sleep(10);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            executor.shutdown();
        }
    }

    record UserData(String profile, String settings, String orders) {}
}
```

### Example 10: Manage Thread-Shared Data with Scoped Values

Title: Use Scoped Values for Thread-Safe Context Propagation
Description: Prefer `ScopedValue` over `ThreadLocal` for sharing immutable data robustly and efficiently across tasks within a dynamically bounded scope, especially when working with virtual threads. Scoped Values became stable in Java 25 (JEP 506). - Use `ScopedValue.newInstance()` to create scoped value instances. - Use `ScopedValue.where()` to establish scoped bindings with automatic cleanup. - Combine multiple scoped values using method chaining for complex contexts. - Prefer scoped values for immutable context data that needs to flow through call chains. - Avoid `ThreadLocal` with virtual threads due to potential memory issues and lack of structured cleanup. - Use scoped values for request-scoped data in web applications and distributed tracing. - Leverage automatic inheritance in structured concurrency and virtual threads.

**Good example:**

```java
// GOOD: Scoped values for thread-safe context propagation
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.StructuredTaskScope;
import java.util.concurrent.Future;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

class ScopedValueExample {

    // Define scoped values for different types of context
    private static final ScopedValue<String> USER_ID = ScopedValue.newInstance();
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
    private static final ScopedValue<Map<String, String>> SECURITY_CONTEXT = ScopedValue.newInstance();

    private final ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

    // Web request processing with scoped context
    public String handleWebRequest(String userId, String requestId, Map<String, String> securityContext) {
        return ScopedValue.where(USER_ID, userId)
            .where(REQUEST_ID, requestId)
            .where(SECURITY_CONTEXT, securityContext)
            .call(() -> processRequestWithContext());
    }

    private String processRequestWithContext() {
        String userId = USER_ID.get();
        String requestId = REQUEST_ID.get();

        System.out.println("Processing request " + requestId + " for user " + userId);

        // Process in parallel while maintaining context
        return ScopedValue.where(USER_ID, userId)
            .where(REQUEST_ID, requestId)
            .where(SECURITY_CONTEXT, SECURITY_CONTEXT.get())
            .call(() -> {
                try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {

                    // All forked tasks inherit the scoped values
                    Future<String> businessLogic = scope.fork(this::executeBusinessLogic);
                    Future<String> auditLog = scope.fork(this::createAuditLog);
                    Future<String> notification = scope.fork(this::sendNotification);

                    scope.join();
                    scope.throwIfFailed();

                    return combineResults(
                        businessLogic.resultNow(),
                        auditLog.resultNow(),
                        notification.resultNow()
                    );

                } catch (Exception e) {
                    throw new RuntimeException("Request processing failed", e);
                }
            });
    }

    private String executeBusinessLogic() {
        String userId = USER_ID.get();
        String requestId = REQUEST_ID.get();

        // Simulate business logic
        try { Thread.sleep(100); } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        return "Business logic completed for user " + userId + " (request: " + requestId + ")";
    }

    private String createAuditLog() {
        String userId = USER_ID.get();
        String requestId = REQUEST_ID.get();
        Map<String, String> securityContext = SECURITY_CONTEXT.get();

        String securityInfo = securityContext.getOrDefault("role", "unknown");
        return "Audit logged for user " + userId + " with role " + securityInfo + " (request: " + requestId + ")";
    }

    private String sendNotification() {
        String userId = USER_ID.get();
        String requestId = REQUEST_ID.get();

        return "Notification sent to user " + userId + " (request: " + requestId + ")";
    }

    // Nested scoped value usage
    public void processWithNestedScope(String userId, List<String> tasks) {
        ScopedValue.where(USER_ID, userId)
            .run(() -> {
                for (String task : tasks) {
                    ScopedValue.where(REQUEST_ID, "task-" + task)
                        .run(() -> processTask(task));
                }
            });
    }

    private void processTask(String task) {
        String userId = USER_ID.get();
        String requestId = REQUEST_ID.get();

        System.out.println("Processing task " + task + " for user " + userId + " (request: " + requestId + ")");
    }

    private String combineResults(String businessResult, String auditResult, String notificationResult) {
        return String.format("Results: [%s] [%s] [%s]", businessResult, auditResult, notificationResult);
    }

    public void shutdown() {
        virtualExecutor.shutdown();
    }
}
```

**Bad example:**

```java
// AVOID: ThreadLocal with virtual threads
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.Map;
import java.util.HashMap;

class BadScopedValueUsage {

    // BAD: Using ThreadLocal with virtual threads
    private static final ThreadLocal<String> USER_CONTEXT = new ThreadLocal<>();

    public void processWithThreadLocal(String userId) {
        USER_CONTEXT.set(userId);

        ExecutorService virtualExecutor = Executors.newVirtualThreadPerTaskExecutor();

        virtualExecutor.submit(() -> {
            // BAD: ThreadLocal values don't propagate to virtual threads properly
            String currentUserId = USER_CONTEXT.get(); // Likely null
            System.out.println("Processing for user: " + currentUserId);
        });

        // BAD: Not cleaning up ThreadLocal
        // USER_CONTEXT.remove(); // Missing cleanup
    }

    // BAD: Manual context propagation
    public void processWithManualPropagation(String userId, String requestId) {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // BAD: Manually passing context to each task
        executor.submit(() -> {
            processTask(userId, requestId, "task1");
        });

        executor.submit(() -> {
            processTask(userId, requestId, "task2");
        });

        // BAD: Error-prone and verbose
    }

    private void processTask(String userId, String requestId, String task) {
        System.out.println("Processing " + task + " for user " + userId + " (request: " + requestId + ")");
    }

    // BAD: ThreadLocal memory leak
    private static final ThreadLocal<Map<String, String>> CONTEXT =
        ThreadLocal.withInitial(HashMap::new);

    public void leakyContextManagement() {
        CONTEXT.get().put("userId", "123");
        CONTEXT.get().put("requestId", "456");

        // Process something
        processData();

        // BAD: Forgetting to clean up can cause memory leaks
        // CONTEXT.remove(); // Missing cleanup
    }

    private void processData() {
        System.out.println("Processing data...");
    }
}
```

### Example 11: Cooperative Cancellation and Interruption

Title: Propagate interruption, bound waits, and cleanup safely
Description: Design tasks to be cancellable and responsive to interruption. Use timeouts on blocking calls, avoid swallowing `InterruptedException`, and ensure resources are cleaned up after cancellation.

**Good example:**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

class CancellationService {
    private final BlockingQueue<String> queue = new ArrayBlockingQueue<>(100);
    private final ExecutorService executor = Executors.newFixedThreadPool(2);
    private final ReentrantLock lock = new ReentrantLock();

    public Future<?> startWorker() {
        return executor.submit(() -> {
            while (!Thread.currentThread().isInterrupted()) {
                try {
                    String item = queue.poll(500, TimeUnit.MILLISECONDS); // timed wait
                    if (item != null) process(item);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt(); // restore and exit loop
                }
            }
            cleanup();
        });
    }

    public boolean tryUpdateStateSafely() throws InterruptedException {
        if (lock.tryLock(200, TimeUnit.MILLISECONDS)) { // time-bounded acquisition
            try {
                // update state
                return true;
            } finally {
                lock.unlock();
            }
        }
        return false;
    }

    private void process(String s) { /* ... */ }
    private void cleanup() { /* close resources */ }

    public void shutdown() {
        executor.shutdownNow(); // interrupt workers
    }
}
```

**Bad example:**

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;

class BadCancellationService {
    private final BlockingQueue<String> queue = new LinkedBlockingQueue<>(); // unbounded
    private final ExecutorService executor = Executors.newSingleThreadExecutor();

    public void startWorker() {
        executor.submit(() -> {
            for (;;) {
                try {
                    String item = queue.take(); // indefinite block
                    process(item);
                } catch (InterruptedException ignored) { // SWALLOWED
                    // continues running forever
                }
            }
        });
    }

    private void process(String s) { /* ... */ }
}
```

### Example 12: Overload Protection and Backpressure

Title: Bound queues, reject sanely, and limit concurrency
Description: Prevent unbounded growth and cascading failures using bounded queues, appropriate rejection policies, and bulkheads (semaphores) for scarce resources.

**Good example:**

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Callable;
import java.util.concurrent.Future;
import java.util.concurrent.RejectedExecutionException;
import java.util.concurrent.Semaphore;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

class BackpressureExample {
    private final Semaphore dbBulkhead = new Semaphore(20); // cap concurrent DB calls
    private final ThreadPoolExecutor executor = new ThreadPoolExecutor(
        4, 8, 60, TimeUnit.SECONDS,
        new ArrayBlockingQueue<>(200), // bounded
        new ThreadPoolExecutor.CallerRunsPolicy() // graceful shedding
    );

    public Future<String> handleRequest(Callable<String> task) {
        return executor.submit(() -> {
            if (!dbBulkhead.tryAcquire(500, TimeUnit.MILLISECONDS)) {
                throw new RejectedExecutionException("Overloaded");
            }
            try {
                return task.call();
            } finally {
                dbBulkhead.release();
            }
        });
    }
}
```

**Bad example:**

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class BadBackpressureExample {
    // Unbounded queue behind fixed pool can accumulate unbounded work
    private final ExecutorService pool = Executors.newFixedThreadPool(8); // LinkedBlockingQueue (unbounded)

    public void flood(Runnable task) {
        for (int i = 0; i < 1_000_000; i++) {
            pool.submit(task); // no bounds, no rejection
        }
    }
}
```

### Example 13: ForkJoin and ManagedBlocker for Blocking Operations

Title: Cooperate with the common pool when blocking
Description: Avoid blocking the ForkJoin common pool. If blocking is unavoidable, use `ForkJoinPool.ManagedBlocker` or a dedicated executor.

**Good example:**

```java
import java.util.concurrent.ForkJoinPool;

class ManagedBlockerExample {
    static String blockingIO() throws InterruptedException {
        Thread.sleep(200); // simulate blocking
        return "ok";
    }

    static String callBlockingSafely() throws InterruptedException {
        ForkJoinPool.managedBlock(new ForkJoinPool.ManagedBlocker() {
            volatile boolean done;
            @Override public boolean block() throws InterruptedException {
                if (!done) {
                    blockingIO();
                    done = true;
                }
                return true;
            }
            @Override public boolean isReleasable() { return done; }
        });
        return "ok";
    }
}
```

**Bad example:**

```java
import java.util.List;

class BadParallelBlocking {
    public List<String> doWork(List<String> inputs) {
        return inputs.parallelStream() // uses common pool
            .map(x -> {
                try {
                    Thread.sleep(200); // BLOCKS common pool worker
                    return x + "!";
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            })
            .toList();
    }
}
```

### Example 14: Avoid Pinning with Virtual Threads

Title: Keep blocking out of intrinsic locks
Description: With virtual threads, `synchronized` around blocking calls can pin to a carrier thread. Prefer cooperative locks or move blocking outside critical sections.

**Good example:**

```java
import java.util.concurrent.locks.ReentrantLock;

class PinningSafeExample {
    private final ReentrantLock lock = new ReentrantLock();
    private int shared = 0;

    public void doWork() {
        // Keep critical section minimal, non-blocking
        lock.lock();
        try {
            shared++;
        } finally {
            lock.unlock();
        }
        // Perform blocking outside lock (ok with virtual threads)
        try { Thread.sleep(100); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
}
```

**Bad example:**

```java
class PinningBadExample {
    private int shared;
    public synchronized void doWork() {
        shared++;
        try {
            Thread.sleep(100); // pins virtual thread to carrier thread
        } catch (InterruptedException e) {
            // swallowed
        }
    }
}
```


### Example 15: Phased Execution with Phaser

Title: Coordinate tasks in phases with dynamic party registration
Description: Use Phaser for coordinating tasks that proceed in phases, allowing dynamic registration and deregistration of parties.

**Good example:**

```java
import java.util.concurrent.Phaser;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class PhaserExample {
    private final Phaser phaser = new Phaser(1); // main thread

    public void runPhasedTasks(int taskCount) {
        ExecutorService executor = Executors.newFixedThreadPool(taskCount);

        for (int i = 0; i < taskCount; i++) {
            phaser.register(); // dynamic registration
            executor.submit(() -> {
                try {
                    // Phase 0 work
                    Thread.sleep(100);
                    phaser.arriveAndAwaitAdvance(); // wait for all

                    // Phase 1 work
                    Thread.sleep(200);
                    phaser.arriveAndDeregister(); // done
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        phaser.arriveAndAwaitAdvance(); // advance phase 0
        phaser.arriveAndAwaitAdvance(); // advance phase 1
        phaser.arriveAndDeregister(); // main done

        executor.shutdown();
    }
}
```

**Bad example:**

```java
class BadPhaser {
    public void misusePhaser() {
        Phaser phaser = new Phaser(3);
        // BAD: forgetting to arrive/deregister
        // leads to deadlock
        phaser.awaitAdvance(0); // hangs forever
    }
}
```

### Example 16: Synchronization with CyclicBarrier

Title: Wait for threads to reach common barrier points
Description: Use CyclicBarrier for synchronizing threads at barrier points, reusable across multiple cycles.

**Good example:**

```java
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class BarrierExample {
    public void coordinate(int parties) throws Exception {
        CyclicBarrier barrier = new CyclicBarrier(parties, () -> System.out.println("All arrived!"));

        ExecutorService executor = Executors.newFixedThreadPool(parties);
        for (int i = 0; i < parties; i++) {
            executor.submit(() -> {
                try {
                    // Work
                    Thread.sleep(100);
                    barrier.await(); // sync point 1

                    // More work
                    Thread.sleep(200);
                    barrier.await(); // sync point 2 (reusable)
                } catch (Exception e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        executor.shutdown();
    }
}
```

**Bad example:**

```java
class BadBarrier {
    public void misuse() throws Exception {
        CyclicBarrier barrier = new CyclicBarrier(3);
        barrier.await(); // BAD: not enough parties, hangs
    }
}
```

### Example 17: Data Exchange with Exchanger

Title: Safely exchange data between two threads
Description: Use Exchanger for point-to-point data exchange between exactly two threads.

**Good example:**

```java
import java.util.concurrent.Exchanger;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class ExchangerExample {
    public void exchangeData() {
        Exchanger<String> exchanger = new Exchanger<>();
        ExecutorService executor = Executors.newFixedThreadPool(2);

        executor.submit(() -> {
            try {
                String data = "From Thread 1";
                String received = exchanger.exchange(data);
                System.out.println("Thread 1 received: " + received);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        executor.submit(() -> {
            try {
                String data = "From Thread 2";
                String received = exchanger.exchange(data);
                System.out.println("Thread 2 received: " + received);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });

        executor.shutdown();
    }
}
```

**Bad example:**

```java
class BadExchanger {
    public void misuse() throws InterruptedException {
        Exchanger<String> exchanger = new Exchanger<>();
        exchanger.exchange("data"); // BAD: no partner, hangs forever
    }
}
```

## Output Format

- **ANALYZE** Java code to identify specific concurrency issues and categorize them by impact (CRITICAL, PERFORMANCE, DEADLOCK_RISK, SCALABILITY, THREAD_SAFETY) and concurrency area (thread safety, synchronization, thread pools, async operations, modern concurrency)
- **CATEGORIZE** concurrency improvements found: Thread Safety Issues (race conditions vs atomic operations, unsafe collections vs concurrent collections, shared mutable state vs immutable objects), Thread Pool Management (improper sizing vs optimal configuration, resource leaks vs proper lifecycle management), Synchronization Problems (deadlock risks vs lock-free algorithms, excessive contention vs efficient synchronization), Performance Issues (blocking operations vs non-blocking alternatives, memory consistency problems vs volatile/final usage), and Modern Concurrency Gaps (platform threads vs virtual threads, callback hell vs CompletableFuture composition, missing structured concurrency vs scoped concurrency patterns)
- **APPLY** concurrency best practices directly by implementing the most appropriate improvements for each identified issue: Replace unsafe collections with concurrent alternatives, implement proper synchronization using atomic classes and concurrent utilities, configure thread pools with appropriate sizing and lifecycle management, refactor blocking operations to non-blocking alternatives using CompletableFuture, eliminate race conditions through immutability and proper synchronization, and adopt modern concurrency features like virtual threads and structured concurrency where beneficial
- **IMPLEMENT** comprehensive concurrency refactoring using proven patterns: Establish thread-safe data structures using concurrent collections and atomic classes, implement efficient synchronization with locks, semaphores, and barriers, configure optimal thread pool management with proper ExecutorService usage, apply asynchronous programming patterns with CompletableFuture composition, integrate modern concurrency features (virtual threads, structured concurrency, scoped values), and implement proper error handling and resource management in concurrent contexts
- **REFACTOR** code systematically following the concurrency improvement roadmap: First eliminate critical thread safety issues through atomic operations and concurrent collections, then optimize synchronization mechanisms to reduce contention and deadlock risks, configure proper thread pool management and lifecycle, refactor blocking operations to asynchronous alternatives, integrate modern concurrency features for improved scalability, and implement comprehensive testing strategies for concurrent code validation
- **EXPLAIN** the applied concurrency improvements and their benefits: Thread safety enhancements through proper synchronization and atomic operations, performance improvements via optimized thread pool management and non-blocking operations, scalability gains from modern concurrency features like virtual threads, deadlock prevention through lock-free algorithms and proper synchronization patterns, and maintainability improvements from structured concurrency and clear async composition
- **VALIDATE** that all applied concurrency refactoring compiles successfully, maintains thread safety guarantees, eliminates race conditions and deadlock risks, preserves or improves performance characteristics, and achieves the intended concurrency improvements through comprehensive testing and verification
- **CANCELLATION/INTERRUPTION**: Identify blocking calls and long-running tasks; ensure interruption is propagated/restored, add timeouts (`orTimeout`, `completeOnTimeout`, timed `poll/take/tryLock`), and verify `Future.cancel(true)` paths release resources.
- **BACKPRESSURE/OVERLOAD**: Detect unbounded producers and queues; introduce bounded queues, rejection policies, semaphores/bulkheads, and when streaming, prefer `Flow`/Reactive Streams to enforce backpressure.
- **FORKJOIN/PARALLEL STREAMS USAGE**: Flag blocking operations in common pool, migrate to dedicated executors or `ManagedBlocker`, verify tasks are CPU-bound and side-effect-free, and gate `parallelStream()` usage behind measurements.
- **PINNING WITH VIRTUAL THREADS**: Inspect `synchronized` blocks around blocking I/O; replace with cooperative locks, shrink critical sections, and recommend JFR pinning analysis.
- **COORDINATION PRIMITIVES**: Identify opportunities for Phaser (phased tasks), CyclicBarrier (reusable barriers), and Exchanger (pairwise exchange); ensure proper usage with interruption handling and resource cleanup.

## Safeguards

- **BLOCKING SAFETY CHECK**: ALWAYS run `./mvnw compile` before ANY recommendations
- **CRITICAL VALIDATION**: Execute `./mvnw clean verify` to ensure all tests pass
- **MANDATORY VERIFICATION**: Confirm all existing functionality remains intact after concurrency improvements
- **ROLLBACK REQUIREMENT**: Ensure all changes can be easily reverted if issues arise
- **INCREMENTAL SAFETY**: Apply concurrency improvements incrementally, validating after each modification
- **THREAD SAFETY VALIDATION**: Verify thread-safe components work correctly under concurrent access
- **DEADLOCK PREVENTION**: Check for potential deadlock scenarios in synchronized code
- **RESOURCE LEAK PROTECTION**: Ensure proper cleanup of thread pools, executors, and other concurrent resources
- **MEMORY CONSISTENCY CHECK**: Validate proper synchronization and memory visibility semantics
- **PERFORMANCE REGRESSION GUARD**: Monitor for performance degradation after concurrency changes
- **INTERRUPTION/CANCELLATION DISCIPLINE**: Never swallow interruptions; propagate or restore interrupt flags. Ensure timeouts on blocking operations and verify cancellation paths free resources safely.
- **VIRTUAL-THREAD PINNING GUARD**: Audit for intrinsic locks around blocking calls; prefer lock implementations compatible with parking. Use JFR to detect pinning.
- **OVERLOAD/BACKPRESSURE PROTECTION**: Avoid unbounded queues; enforce bounded capacity, sane rejection policies, and rate/concurrency limits to prevent cascading failures.
- **TIMEOUTS/RETRIES/IDEMPOTENCY**: Bound external calls with timeouts, use bounded-jittered retries only for idempotent operations, and validate no duplicate side effects occur.
- **COORDINATION SAFETY**: For Phaser/Barrier/Exchanger, validate party counts, handle interruptions, and prevent hangs from mismatched arrivals.