# Java 17 DSA Cheat Sheet

This sheet is built for fast review during interviews and contests.

## Quick Index

- [ArrayList](#arraylist) — dynamic array with fast random access.
- [LinkedList](#linkedlist) — node-based list with fast front/back inserts.
- [Stack](#stack) — LIFO structure for DFS and parsing.
- [Queue](#queue) — FIFO structure for BFS and scheduling.
- [Deque (ArrayDeque)](#deque-arraydeque) — two-ended queue for sliding windows and deques.
- [HashMap](#hashmap) — key-value lookup with average O(1) access.
- [HashSet](#hashset) — unique-element storage with fast membership checks.
- [PriorityQueue](#priorityqueue) — heap-based queue for top-k and shortest path problems.
- [HashMap Deep Dive: Internals & Mechanics](#hashmap-deep-dive-internals--mechanics) — internal bucket logic, collisions, and resizing.

---

## ArrayList

### 1. Visual Structure & Analogy
**Analogy:** Think of an ArrayList as a dynamic bookshelf. It stores items in contiguous slots.

```text
[0] [1] [2] [3] [4] ...
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Add at end | `list.add(value)` | `true` | No failure for normal adds. |
| Add at index | `list.add(index, value)` | `void` | Throws `IndexOutOfBoundsException` if index is invalid. |
| Get | `list.get(index)` | `E` | Throws `IndexOutOfBoundsException` if index is invalid. |
| Set | `list.set(index, value)` | Previous element | Throws `IndexOutOfBoundsException` if index is invalid. |
| Remove by index | `list.remove(index)` | Removed element | Throws `IndexOutOfBoundsException` if index is invalid. |
| Remove by value | `list.remove(value)` | `true`/`false` | Removes first matching element. |
| Contains | `list.contains(value)` | `true`/`false` | Works by equality. |

### 3. Traversals & Iteration

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ArrayListDemo {
    public static void main(String[] args) {
        List<Integer> nums = new ArrayList<>(List.of(1, 2, 3, 4, 5));

        for (int i = 0; i < nums.size(); i++) {
            System.out.print(nums.get(i) + " ");
        }
        System.out.println();

        Iterator<Integer> it = nums.iterator();
        while (it.hasNext()) {
            int x = it.next();
            if (x % 2 == 0) {
                it.remove();
            }
        }

        System.out.println(nums);
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| ArrayList | O(n) | O(n) | O(n) | O(1) amortized | O(n) | O(n) | O(n) | O(1) | O(1) |

### 5. Common Use Cases
- Sliding window problems.
- Fast random access in dynamic arrays.
- Frequent read-heavy workloads.

**JVM reference note:** The JVM stores object references in a backing array. Indexing is direct, but resizing copies those references to a new array.

---

## LinkedList

### 1. Visual Structure & Analogy
**Analogy:** Think of a LinkedList as a train. Each node points to the next node.

```text
[Head] -> [A] <-> [B] <-> [C] -> [null]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Add first | `list.addFirst(value)` | `void` | Works even for empty list. |
| Add last | `list.addLast(value)` | `void` | Works even for empty list. |
| Get first | `list.getFirst()` | First element | Throws `NoSuchElementException` if empty. |
| Get last | `list.getLast()` | Last element | Throws `NoSuchElementException` if empty. |
| Remove first | `list.removeFirst()` | Removed element | Throws `NoSuchElementException` if empty. |
| Remove last | `list.removeLast()` | Removed element | Throws `NoSuchElementException` if empty. |
| Contains | `list.contains(value)` | `true`/`false` | Works by equality. |

### 3. Traversals & Iteration

```java
import java.util.LinkedList;
import java.util.List;

public class LinkedListDemo {
    public static void main(String[] args) {
        List<Integer> nums = new LinkedList<>(List.of(10, 20, 30));

        for (int value : nums) {
            System.out.print(value + " ");
        }
        System.out.println();
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| LinkedList | O(n) | O(n) | O(n) | O(1) | O(1) | O(1) | O(1) | O(n) | O(n) |

### 5. Common Use Cases
- Frequent insertions and deletions at the front or back.
- Implementing undo/redo stacks.
- Browser history or playlist-like structures.

**JVM reference note:** Each node is a separate object. The JVM follows `next` and `prev` references to move between nodes.

---

## Stack

### 1. Visual Structure & Analogy
**Analogy:** Think of a Stack as a stack of plates. The last placed plate is removed first.

```text
[top] -> [C] -> [B] -> [A]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Push | `stack.push(value)` | `void` | No failure for normal use. |
| Pop | `stack.pop()` | Removed element | Throws `EmptyStackException` if empty. |
| Peek | `stack.peek()` | Top element | Throws `EmptyStackException` if empty. |
| Is empty | `stack.isEmpty()` | `true`/`false` | Simple check. |

### 3. Traversals & Iteration

```java
import java.util.Stack;

public class StackDemo {
    public static void main(String[] args) {
        Stack<Integer> stack = new Stack<>();
        stack.push(1);
        stack.push(2);
        stack.push(3);

        while (!stack.isEmpty()) {
            System.out.println(stack.pop());
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Stack | O(n) | O(n) | O(n) | O(1) | O(1) | O(1) | O(1) | O(1) | O(1) |

### 5. Common Use Cases
- Balanced parentheses validation.
- DFS traversal in graphs and trees.
- Undo operations.

**JVM reference note:** Stack operations push and pop object references from a backing structure. The runtime uses those references directly.

---

## Queue

### 1. Visual Structure & Analogy
**Analogy:** Think of a Queue as a line of people. The first person leaves first.

```text
[front] -> [A] -> [B] -> [C] -> [rear]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Enqueue | `queue.add(value)` | `true`/exception | Throws `IllegalStateException` when full for bounded queues. |
| Dequeue | `queue.remove()` | Removed element | Throws `NoSuchElementException` if empty. |
| Peek | `queue.element()` | Front element | Throws `NoSuchElementException` if empty. |
| Poll | `queue.poll()` | `null` or removed element | Returns `null` if empty. |

### 3. Traversals & Iteration

```java
import java.util.LinkedList;
import java.util.Queue;

public class QueueDemo {
    public static void main(String[] args) {
        Queue<Integer> queue = new LinkedList<>();
        queue.add(1);
        queue.add(2);
        queue.add(3);

        while (!queue.isEmpty()) {
            System.out.println(queue.remove());
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Queue | O(n) | O(n) | O(n) | O(1) | O(1) | O(1) | O(1) | O(1) | O(1) |

### 5. Common Use Cases
- BFS traversal in graphs.
- Task scheduling and job queues.
- Producer-consumer buffering.

**JVM reference note:** The queue interface is resolved to a concrete implementation at runtime. The JVM calls the implementation’s methods through object references.

---

## Deque (ArrayDeque)

### 1. Visual Structure & Analogy
**Analogy:** Think of a Deque as a two-ended queue. You can add or remove at both sides.

```text
[front] <-> [A] <-> [B] <-> [C] <-> [rear]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Add first | `deque.addFirst(value)` | `void` | Throws `IllegalStateException` if full for bounded variants. |
| Add last | `deque.addLast(value)` | `void` | Throws `IllegalStateException` if full for bounded variants. |
| Remove first | `deque.removeFirst()` | Removed element | Throws `NoSuchElementException` if empty. |
| Remove last | `deque.removeLast()` | Removed element | Throws `NoSuchElementException` if empty. |
| Peek first | `deque.peekFirst()` | `null` or element | Returns `null` if empty. |
| Peek last | `deque.peekLast()` | `null` or element | Returns `null` if empty. |

### 3. Traversals & Iteration

```java
import java.util.ArrayDeque;
import java.util.Deque;

public class DequeDemo {
    public static void main(String[] args) {
        Deque<Integer> deque = new ArrayDeque<>();
        deque.addLast(1);
        deque.addLast(2);
        deque.addFirst(0);

        while (!deque.isEmpty()) {
            System.out.println(deque.removeFirst());
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Deque (ArrayDeque) | O(n) | O(n) | O(n) | O(1) | O(1) | O(1) | O(1) | O(1) | O(1) |

### 5. Common Use Cases
- Sliding window with two ends.
- Monotonic queue problems.
- Fast stack/queue hybrid use cases.

**JVM reference note:** `ArrayDeque` stores references in a circular array and moves a head/tail index around that array.

---

## HashMap

### 1. Visual Structure & Analogy
**Analogy:** Think of a HashMap as a labeled locker room. Keys map to values through a hash index.

```text
Bucket Array -> [ ] [ ] [ ] [ ]
                  |     |     |
                  v     v     v
                [Node] [Node] [Node]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Put | `map.put(key, value)` | Previous value or `null` | Replaces old value if key already exists. |
| Get | `map.get(key)` | `null` or value | Returns `null` if key is absent. |
| Contains key | `map.containsKey(key)` | `true`/`false` | Simple presence check. |
| Contains value | `map.containsValue(value)` | `true`/`false` | Linear scan in many cases. |
| Remove | `map.remove(key)` | Removed value or `null` | Returns `null` if key not present. |
| Size | `map.size()` | Integer | Simple count. |

### 3. Traversals & Iteration

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapDemo {
    public static void main(String[] args) {
        Map<String, Integer> freq = new HashMap<>();
        freq.put("a", 1);
        freq.put("b", 2);
        freq.put("c", 3);

        for (Map.Entry<String, Integer> entry : freq.entrySet()) {
            System.out.println(entry.getKey() + " -> " + entry.getValue());
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| HashMap | O(n) | O(1) | O(n) | O(1) | O(n) | O(1) | O(n) | O(1) | O(n) |

### 5. Common Use Cases
- Frequency counting.
- Caching and memoization.
- Mapping from key to value in graphs and strings.

**JVM reference note:** Buckets hold references to `Node` objects. The JVM follows those references during lookup and collision traversal.

### HashMap Deep Dive: Internals & Mechanics

```text
Bucket Array
[0] [1] [2] [3] [4] [5] [6] [7]
 |   |   |   |   |   |   |   |
 v   v   v   v   v   v   v   v
[Node]--->[Node]---->null   [Node]--->[Node]--->[Node]
        (chain)                        (tree)
```

#### How `put()` works
1. **Compute** the hash with `hashCode()`.
2. **Find** the bucket index with `(n - 1) & hash`.
3. **Check** the bucket.
   - If empty, create a new node.
   - If a node already exists, compare keys with `equals()`.
4. **Replace** the value when keys are equal.
5. **Link** a new node when keys differ and a collision occurs.
6. **Treeify** the bucket if the collision chain becomes long enough.
7. **Resize** when the load factor threshold is crossed.

#### How `get()` works
1. **Compute** the hash with `hashCode()`.
2. **Find** the bucket index.
3. **Walk** the chain or tree in that bucket.
4. **Return** the value when `equals()` matches the key.
5. **Return** `null` when no matching key exists.

#### Collision Resolution
**Collision Resolution:** HashMap uses separate chaining. Each bucket stores either a linked list or a red-black tree.

#### Load Factor and Resizing
**Load Factor:** The default load factor is `0.75`.

**Threshold Rule:** A resize happens when `size > capacity * loadFactor`.

**Capacity Doubling:** When resizing, capacity becomes about `2 * oldCapacity`.

#### Treeification and Untreeification
**Treeification:** A bucket is converted to a tree when the bucket length is at least `8` and the total capacity is at least `64`.

**Untreeification:** A bucket is converted back to a linked list when its length becomes `6` or less.

---

## HashSet

### 1. Visual Structure & Analogy
**Analogy:** Think of a HashSet as a bag of unique items. It stores each item once.

```text
[hash] -> [A] -> [B] -> [C]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Add | `set.add(value)` | `true`/`false` | Returns `false` if the value already exists. |
| Contains | `set.contains(value)` | `true`/`false` | Simple membership test. |
| Remove | `set.remove(value)` | `true`/`false` | Returns `false` if the value is absent. |
| Size | `set.size()` | Integer | Simple count. |

### 3. Traversals & Iteration

```java
import java.util.HashSet;
import java.util.Set;

public class HashSetDemo {
    public static void main(String[] args) {
        Set<Integer> set = new HashSet<>();
        set.add(1);
        set.add(2);
        set.add(2);

        for (Integer value : set) {
            System.out.println(value);
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| HashSet | O(n) | O(1) | O(n) | O(1) | O(n) | O(1) | O(n) | O(1) | O(n) |

### 5. Common Use Cases
- Removing duplicates from a collection.
- Tracking visited nodes in graph problems.
- Fast membership testing.

**JVM reference note:** `HashSet` is backed by a `HashMap`. Each element is stored as a key reference, and the value is a dummy placeholder.

---

## PriorityQueue

### 1. Visual Structure & Analogy
**Analogy:** Think of a PriorityQueue as a heap-based waiting line. The highest priority element sits at the front.

```text
        [1]
      /     \
   [3]      [2]
  /   \    /   \
 [6] [5] [4] [7]
```

### 2. Common Operations

| Operation | Java 17 Method Call | Default/Failure Return Value | Edge Case Behavior |
|---|---|---:|---|
| Add | `queue.add(value)` | `true`/exception | Throws `IllegalStateException` if capacity is exceeded. |
| Peek | `queue.peek()` | `null` or element | Returns `null` if empty. |
| Poll | `queue.poll()` | `null` or removed element | Returns `null` if empty. |
| Remove | `queue.remove()` | Removed element | Throws `NoSuchElementException` if empty. |

### 3. Traversals & Iteration

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class PriorityQueueDemo {
    public static void main(String[] args) {
        Queue<Integer> pq = new PriorityQueue<>();
        pq.add(5);
        pq.add(1);
        pq.add(3);

        while (!pq.isEmpty()) {
            System.out.println(pq.poll());
        }
    }
}
```

### 4. Big-O Complexity

| Structure | Space | Search Avg | Search Worst | Insert Avg | Insert Worst | Delete Avg | Delete Worst | Access Avg | Access Worst |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| PriorityQueue | O(n) | O(n) | O(n) | O(log n) | O(log n) | O(log n) | O(log n) | O(1) | O(1) |

### 5. Common Use Cases
- Kth largest or kth smallest problems.
- Dijkstra’s algorithm.
- Top-K and heap-based selection problems.

**JVM reference note:** The heap is stored as an array of object references. Sift-up and sift-down operations move those references to restore heap order.


---

## Comparator

`Comparator<T>` defines a custom ordering for objects. It is especially useful with `List.sort()`, `Collections.sort()`, and `PriorityQueue`.

### 1. Simple Lambda

For a simple comparison, the lambda can directly compare one field.

```java
import java.util.*;

List<Integer> nums = new ArrayList<>(List.of(5, 2, 9, 1));

// Ascending
nums.sort((a, b) -> a - b);

// Descending
nums.sort((a, b) -> b - a);
```

A safer form for integers is `Integer.compare()` because subtraction can overflow:

```java
nums.sort((a, b) -> Integer.compare(a, b));
nums.sort((a, b) -> Integer.compare(b, a));
```

### 2. Complex Comparator

For objects, the comparison can use multiple fields and tie-breakers.

```java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

List<Person> people = new ArrayList<>(List.of(
    new Person("Alice", 30),
    new Person("Bob", 25),
    new Person("Charlie", 30)
));

people.sort(
    Comparator.comparingInt((Person p) -> p.age)
              .thenComparing(p -> p.name)
);
```

This sorts by:

1. `age` ascending.
2. If ages are equal, `name` ascending.

For descending order:

```java
people.sort(
    Comparator.comparingInt((Person p) -> p.age)
              .reversed()
              .thenComparing(p -> p.name)
);
```

### 3. Comparator with `PriorityQueue`

The same comparator idea can define the priority order of a heap.

```java
// Min-heap by age
PriorityQueue<Person> pq =
    new PriorityQueue<>(Comparator.comparingInt(p -> p.age));

// Max-heap by age
PriorityQueue<Person> maxPq =
    new PriorityQueue<>(
        Comparator.comparingInt((Person p) -> p.age).reversed()
    );
```

**Mental model:** `Comparator` answers one question: **"Should `a` come before `b`?"**

```text
compare(a, b) < 0  -> a comes before b
compare(a, b) == 0 -> a and b are equivalent in ordering
compare(a, b) > 0  -> b comes before a
```
