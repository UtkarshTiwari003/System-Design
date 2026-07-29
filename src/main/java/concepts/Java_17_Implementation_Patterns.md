# Java 17 Implementation Patterns

This file is designed for fast lookup during live contests.

## Quick Index

- [1. Advanced Custom Sorting](#1-advanced-custom-sorting) — sort 2D arrays and custom objects with comparators.
- [2. PriorityQueue (Heap) Customizations](#2-priorityqueue-heap-customizations) — min-heap, max-heap, and heapify notes.
- [3. Lambda & Functional Style Idioms](#3-lambda--functional-style-idioms) — `computeIfAbsent`, `getOrDefault`, and `merge` patterns.
- [4. Stream API Cookbook](#4-stream-api-cookbook) — grouping, filtering, and flattening examples.
- [5. High-Efficiency Collection Conversions](#5-high-efficiency-collection-conversions) — fast conversions between collections and arrays.
- [6. Bitwise Manipulation & Math Foundations](#6-bitwise-manipulation--math-foundations) — modular arithmetic, exponentiation, and bit counting.
- [7. Boxing and Unboxing Overhead](#7-boxing-and-unboxing-overhead) — memory and garbage-collection trade-offs.
- [8. Competitive Fast I/O Footnote](#8-competitive-fast-io-footnote) — buffered input boilerplate for large test cases.

---

## 1. Advanced Custom Sorting

### Sorting a primitive 2D array

```java
import java.util.Arrays;

public class CustomSortDemo {
    public static void main(String[] args) {
        int[][] intervals = {
            {5, 1},
            {2, 2},
            {3, 1},
            {2, 3}
        };

        Arrays.sort(intervals, (a, b) -> {
            if (a[1] != b[1]) {
                return Integer.compare(a[1], b[1]);
            }
            return Integer.compare(b[0], a[0]);
        });

        for (int[] row : intervals) {
            System.out.println(Arrays.toString(row));
        }
    }
}
```

**Logic:** The second element is sorted ascending. When tied, the first element is sorted descending.

### Sorting custom objects

```java
import java.util.Arrays;
import java.util.List;

class Task {
    int id;
    int priority;

    Task(int id, int priority) {
        this.id = id;
        this.priority = priority;
    }

    @Override
    public String toString() {
        return "Task{" + "id=" + id + ", priority=" + priority + "}";
    }
}

public class ObjectSortDemo {
    public static void main(String[] args) {
        List<Task> tasks = Arrays.asList(
            new Task(2, 5),
            new Task(1, 5),
            new Task(3, 2)
        );

        tasks.sort(
            java.util.Comparator.comparingInt((Task t) -> t.priority)
                .thenComparingInt(t -> -t.id)
        );

        System.out.println(tasks);
    }
}
```

---

## 2. PriorityQueue (Heap) Customizations

### Min-heap and max-heap

```java
import java.util.PriorityQueue;
import java.util.Queue;

public class HeapDemo {
    public static void main(String[] args) {
        Queue<Integer> minHeap = new PriorityQueue<>();
        minHeap.add(5);
        minHeap.add(1);
        minHeap.add(4);

        System.out.println(minHeap.peek());

        Queue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
        maxHeap.add(5);
        maxHeap.add(1);
        maxHeap.add(4);

        System.out.println(maxHeap.peek());
    }
}
```

```text
Min-Heap          Max-Heap
    1                 5
   / \               / \
  4   5             4   1
```

### Custom objects

```java
import java.util.PriorityQueue;
import java.util.Queue;

class Node {
    int value;
    int index;

    Node(int value, int index) {
        this.value = value;
        this.index = index;
    }
}

public class HeapObjectDemo {
    public static void main(String[] args) {
        Queue<Node> minHeap = new PriorityQueue<>((a, b) -> Integer.compare(a.value, b.value));
        minHeap.add(new Node(10, 1));
        minHeap.add(new Node(3, 2));
        minHeap.add(new Node(7, 3));

        System.out.println(minHeap.peek().value);
    }
}
```

**Note:** Building a heap with the constructor-based heapify is **O(N)**. Repeated `add()` calls are **O(N log N)** in total.

---

## 3. Lambda & Functional Style Idioms

| Idiom | Use Case | Example |
|---|---|---|
| `computeIfAbsent` | Create a value only when missing | `map.computeIfAbsent(key, k -> new ArrayList<>());` |
| `getOrDefault` | Read a value safely with a fallback | `map.getOrDefault(key, 0);` |
| `merge` | Combine counts or totals | `map.merge(key, 1, Integer::sum);` |

```java
import java.util.*;

public class IdiomDemo {
    public static void main(String[] args) {
        Map<String, List<Integer>> map = new HashMap<>();
        map.computeIfAbsent("a", k -> new ArrayList<>()).add(1);

        Map<String, Integer> freq = new HashMap<>();
        freq.put("x", 2);
        int value = freq.getOrDefault("y", 0);
        System.out.println(value);

        Map<String, Integer> counts = new HashMap<>();
        counts.merge("x", 1, Integer::sum);
        System.out.println(counts.get("x"));
    }
}
```

---

## 4. Stream API Cookbook

### Grouping elements by frequency

```java
import java.util.*;
import java.util.stream.Collectors;

public class StreamFrequencyDemo {
    public static void main(String[] args) {
        List<Integer> nums = Arrays.asList(1, 2, 2, 3, 3, 3);

        Map<Integer, Long> freq = nums.stream()
            .collect(Collectors.groupingBy(n -> n, Collectors.counting()));

        System.out.println(freq);
    }
}
```

### Filtering and collecting to maps

```java
import java.util.*;
import java.util.stream.Collectors;

public class StreamMapDemo {
    public static void main(String[] args) {
        List<String> words = Arrays.asList("apple", "banana", "apple", "car");

        Map<String, Long> lengths = words.stream()
            .filter(w -> w.length() > 4)
            .collect(Collectors.groupingBy(w -> w, Collectors.counting()));

        System.out.println(lengths);
    }
}
```

### Flattening nested structures

```java
import java.util.*;
import java.util.stream.Collectors;

public class StreamFlattenDemo {
    public static void main(String[] args) {
        List<List<Integer>> nested = Arrays.asList(
            Arrays.asList(1, 2),
            Arrays.asList(3, 4)
        );

        List<Integer> flat = nested.stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());

        System.out.println(flat);
    }
}
```

### Readability vs. Performance
- **Use streams** for clarity and concise transformations.
- **Use primitive loops** when speed is critical and the logic is simple.
- **Avoid streams** in very tight loops if they hurt runtime limits.

---

## 5. High-Efficiency Collection Conversions

| Goal | Modern Java 17 Method | Notes |
|---|---|---|
| List to primitive array | `list.stream().mapToInt(Integer::intValue).toArray()` | Creates a new array. |
| Primitive array to List | `Arrays.stream(array).boxed().toList()` | Creates a new immutable list. |
| Array to Stream | `Arrays.stream(array)` | Works for arrays. |
| List to Set | `new HashSet<>(list)` | Creates a new mutable set. |
| Array to Map | `Arrays.stream(array).collect(Collectors.toMap(...))` | Creates a new mutable map. |

```java
import java.util.*;
import java.util.stream.Collectors;

public class ConversionDemo {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3);
        int[] arr = list.stream().mapToInt(Integer::intValue).toArray();

        List<Integer> fromArray = Arrays.stream(arr).boxed().toList();
        Set<Integer> set = new HashSet<>(list);

        Map<Integer, String> map = Arrays.stream(new Integer[]{1, 2, 3})
            .collect(Collectors.toMap(x -> x, x -> "v" + x));

        System.out.println(Arrays.toString(arr));
        System.out.println(fromArray);
        System.out.println(set);
        System.out.println(map);
    }
}
```

**View vs. New Structure:** `List.of(...)`, `Set.of(...)`, and `Map.of(...)` create immutable structures. `new ArrayList<>(list)` and `new HashSet<>(set)` create new mutable containers.

---

## 6. Bitwise Manipulation & Math Foundations

### Safe modular arithmetic

```java
public class ModDemo {
    public static int mod(int a, int b) {
        int r = a % b;
        return r < 0 ? r + b : r;
    }

    public static void main(String[] args) {
        System.out.println(mod(-7, 3));
    }
}
```

### Binary exponentiation

```java
public class PowDemo {
    public static long pow(long base, long exp) {
        long result = 1;
        long current = base;

        while (exp > 0) {
            if ((exp & 1L) == 1L) {
                result *= current;
            }
            current *= current;
            exp >>= 1;
        }

        return result;
    }

    public static void main(String[] args) {
        System.out.println(pow(2, 10));
    }
}
```

### Bit counting

```java
public class BitCountDemo {
    public static void main(String[] args) {
        int value = 13;
        System.out.println(Integer.bitCount(value));

        int manual = 0;
        int x = value;
        while (x != 0) {
            manual += x & 1;
            x >>= 1;
        }
        System.out.println(manual);
    }
}
```

---

## 7. Boxing and Unboxing Overhead

**Rule:** Prefer primitives over wrappers in tight loops.

- `int` is a primitive and uses less memory.
- `Integer` is an object and creates heap pressure.
- Frequent boxing and unboxing can trigger extra garbage collection.

```java
public class BoxingDemo {
    public static void main(String[] args) {
        int sum = 0;
        for (int i = 0; i < 1_000_000; i++) {
            sum += i;
        }
        System.out.println(sum);
    }
}
```

**Best practice:** Use `int[]`, `long[]`, and `double[]` when the data volume is large.

---

## 8. Competitive Fast I/O Footnote

```java
import java.io.*;
import java.util.StringTokenizer;

public class FastIO {
    private static final BufferedReader INPUT = new BufferedReader(new InputStreamReader(System.in));
    private static StringTokenizer tokenizer;

    private static String next() throws IOException {
        while (tokenizer == null || !tokenizer.hasMoreTokens()) {
            tokenizer = new StringTokenizer(INPUT.readLine());
        }
        return tokenizer.nextToken();
    }

    public static void main(String[] args) throws Exception {
        int n = Integer.parseInt(next());
        int[] arr = new int[n];

        for (int i = 0; i < n; i++) {
            arr[i] = Integer.parseInt(next());
        }

        System.out.println(arr[0]);
    }
}
```

**Tip:** This pattern is reliable for avoiding TLE in large input cases.

---

## 9. Records, Constants, String/Math Helpers

### String vs. StringBuilder vs. StringBuffer

| Type | Mutable | Thread-safe | Best use |
|---|---|---|---|
| `String` | No | Yes | Immutable constants and map keys |
| `StringBuilder` | Yes | No | Fast single-threaded text building |
| `StringBuffer` | Yes | Yes | Safe multi-threaded text building |

### Interconversion examples

```java
public class StringConversionDemo {
    public static void main(String[] args) {
        String s = "hello";

        StringBuilder sb = new StringBuilder(s);
        sb.append(" world");
        String fromBuilder = sb.toString();

        StringBuffer buffer = new StringBuffer("Java");
        buffer.append(" 17");
        String fromBuffer = buffer.toString();

        String fromChars = String.valueOf(new char[]{'a', 'b', 'c'});

        System.out.println(fromBuilder);
        System.out.println(fromBuffer);
        System.out.println(fromChars);
    }
}
```

### Record usage

```java
public record Point(int x, int y) {
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be non-negative");
        }
    }
}

public class RecordDemo {
    public static void main(String[] args) {
        Point p = new Point(3, 4);
        System.out.println(p.x() + ", " + p.y());
    }
}
```

### Common constants and helpers

```java
public class UtilityDemo {
    public static void main(String[] args) {
        int maxInt = Integer.MAX_VALUE;
        int minInt = Integer.MIN_VALUE;
        double pi = Math.PI;
        double e = Math.E;

        String text = "  Java 17  ";
        String upper = text.trim().toUpperCase();
        String joined = String.join("-", "a", "b", "c");
        String repeated = "ha".repeat(3);

        System.out.println(maxInt);
        System.out.println(minInt);
        System.out.println(pi);
        System.out.println(e);
        System.out.println(upper);
        System.out.println(joined);
        System.out.println(repeated);
    }
}
```

### String and Math shortcuts worth remembering

```java
public class StringMathDemo {
    public static void main(String[] args) {
        String s = "abcde";
        char c = s.charAt(2);
        String sub = s.substring(1, 4);
        boolean ok = s.startsWith("ab") && s.endsWith("de");

        int absVal = Math.abs(-10);
        int maxVal = Math.max(4, 9);
        int minVal = Math.min(4, 9);
        double root = Math.sqrt(25.0);
        double power = Math.pow(2, 10);
        double rounded = Math.round(3.7);

        System.out.println(c);
        System.out.println(sub);
        System.out.println(ok);
        System.out.println(absVal);
        System.out.println(maxVal);
        System.out.println(minVal);
        System.out.println(root);
        System.out.println(power);
        System.out.println(rounded);
    }
}
```

**Note:** Use `record` for immutable data carriers. Use `StringBuilder` when you need many concatenations inside loops.
