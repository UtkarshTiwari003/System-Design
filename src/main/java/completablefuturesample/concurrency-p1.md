# Concurrency Concepts

## **Semaphores**
```java
Semaphore semaphore = new Semaphore(3);

public void accessDatabase() throws InterruptedException {
    semaphore.acquire();     // acquire permit
    try {
        // critical section (DB access)
    } finally {
        semaphore.release(); // release permit
    }
}
```
**What it solves?**
Suppose 100 threads attempt DB access simultaneously. The database supports only 3 connections. Without control, system overload occurs. Semaphore enforces bounded concurrency.

**Internal mechanism (conceptual view):**
Semaphore is built on AbstractQueuedSynchronizer (AQS).
Threads unable to acquire are parked (blocked) in a FIFO queue.
When a permit is released, one waiting thread is unparked.

Important properties:-
- It does NOT protect data (unless permits = 1).
- It controls access quantity.
- It can be fair or non-fair.
- It does not require the releasing thread to be the acquiring thread.

Use cases:
- Rate limiting
- Connection pools
- Throttling APIs
- Bounded parallelism in CPU-bound tasks

---

## **Bounded Blocking Queue**
```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(3);

public void produce(String item) throws InterruptedException {
    queue.put(item);   // blocks if full
}

public String consume() throws InterruptedException {
    return queue.take(); // blocks if empty
}
```
**What it solves?**

Consider the following system:
- 100 producer threads generate tasks  
- Only 5 consumer threads process them  
- Each task consumes memory  

If the queue is **unbounded**, producers continue adding tasks even when consumers cannot keep up.  
Tasks accumulate in memory → heap growth → eventual `OutOfMemoryError`.

If the queue is **bounded** (e.g., capacity = 50):

- Once 50 tasks are waiting,
- Any further `put()` operation blocks,
- Producers are forced to wait,
- The system applies **natural backpressure**.

This stabilizes throughput by aligning production rate with consumption capacity.

---

**Internal mechanism (conceptual view):**

Taking `ArrayBlockingQueue` as an example:
Internally it uses:
- A fixed-size array buffer  
- One `ReentrantLock`  
- Two `Condition` objects:
  - `notFull`
  - `notEmpty`

The lock guarantees mutual exclusion.  
The condition variables coordinate producers and consumers.

---