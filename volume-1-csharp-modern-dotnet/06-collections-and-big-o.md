# Collections and Big O

## Overview

Choosing the right collection is one of the easiest ways to improve performance. Senior engineers are expected not only to know the available collections but also **their internal implementation, time complexity, memory trade-offs and common use cases.**

---

# 1. List<T>

## Interview Answer

`List<T>` is the most commonly used collection in .NET. Internally it is implemented as a **dynamic array**.

When the internal array becomes full, List allocates a larger array (usually doubling its capacity) and copies all existing elements into it.

Because of this, appending elements is usually O(1), while resizing is O(n).

---

## Internal Structure

```
Capacity = 4

+----+----+----+----+
| A  | B  | C  | D  |
+----+----+----+----+

Add(E)

↓

Capacity = 8

+----+----+----+----+----+----+----+----+
| A  | B  | C  | D  | E  |    |    |    |
+----+----+----+----+----+----+----+----+
```

---

## Time Complexity

| Operation | Complexity |
|-----------|------------|
| Index access | O(1) |
| Add (end) | O(1) amortized |
| Insert middle | O(n) |
| Remove middle | O(n) |
| Search | O(n) |

---

## Senior Tip

If you already know approximately how many items will be stored, initialize the capacity.

```csharp
var users = new List<User>(10000);
```

This avoids multiple reallocations.

---

# 2. Dictionary<TKey,TValue>

## Interview Answer

Dictionary is implemented as a **hash table**.

Instead of searching every element, it computes a hash code from the key and uses that hash to determine where the value should be stored.

Because lookup is based on hashing rather than sequential searching, average lookup complexity is O(1).

---

## Internal Structure

```
Buckets

0
1
2
3
4
5

↓

Entries

Hash
Key
Value
Next
```

The bucket points to an entry.

---

# 3. How does Dictionary lookup work?

Example:

```csharp
dictionary["John"]
```

Steps:

1. Compute hash code.
2. Convert hash into bucket index.
3. Go directly to that bucket.
4. Compare keys.
5. Return value.

Unlike a List, the dictionary does **not** scan every element.

---

# 4. What are Collisions?

Sometimes different keys generate the same bucket.

Example:

```
Hash("John") → Bucket 3

Hash("Mike") → Bucket 3
```

Now both values share one bucket.

Dictionary resolves this by keeping entries linked together.

Lookup becomes:

```
Bucket

↓

John

↓

Mike

↓

Peter
```

This is called a collision chain.

---

# 5. Why is Dictionary O(1)?

Average lookup:

```
Hash

↓

Bucket

↓

Entry
```

Only a few comparisons are required.

Worst case:

If every key ends up in the same bucket:

```
Bucket

↓

A

↓

B

↓

C

↓

D

↓

E
```

Lookup becomes O(n).

Fortunately, good hash functions make this extremely rare.

---

# 6. Resize

When Dictionary becomes too full, it allocates a larger bucket array and rehashes all existing entries.

This operation is O(n).

However, because resizing happens infrequently, average insert performance remains O(1).

---

# 7. HashSet<T>

## Interview Answer

HashSet is internally very similar to Dictionary.

The difference is that it stores only keys.

No values.

Perfect for checking uniqueness.

Example:

```csharp
var ids = new HashSet<int>();

ids.Add(1);
ids.Add(2);
ids.Add(2);
```

Result:

```
1
2
```

Duplicate values are ignored.

---

## Time Complexity

| Operation | Complexity |
|-----------|------------|
| Add | O(1) |
| Contains | O(1) |
| Remove | O(1) |

---

# 8. Queue<T>

Queue implements **FIFO**.

First In

First Out

Example:

```
A

↓

B

↓

C
```

Removal order:

```
A

↓

B

↓

C
```

Common uses:

- Background jobs
- Print queues
- Message processing

---

# 9. Stack<T>

Stack implements **LIFO**.

Last In

First Out

```
Push A

Push B

Push C
```

Pop order:

```
C

↓

B

↓

A
```

Common uses:

- Undo
- DFS algorithms
- Expression evaluation

---

# 10. LinkedList<T>

Unlike List,

LinkedList does not store items in one contiguous array.

Each node stores references to the next and previous nodes.

```
A

↔

B

↔

C

↔

D
```

Advantages:

- Fast insertion/removal when you already have the node.

Disadvantages:

- Slow indexing.
- Poor CPU cache locality.

---

# 11. Big O Summary

| Collection | Lookup | Insert | Remove |
|------------|--------|--------|---------|
| List | O(1) index / O(n) search | O(1) end | O(n) |
| Dictionary | O(1) | O(1) | O(1) |
| HashSet | O(1) | O(1) | O(1) |
| Queue | O(1) | O(1) | O(1) |
| Stack | O(1) | O(1) | O(1) |
| LinkedList | O(n) | O(1)* | O(1)* |

\* when the node is already known.

---

# 12. Common Interview Questions

### Why is Dictionary faster than List?

Dictionary computes a hash and jumps directly to the bucket.

List performs sequential searching.

---

### When would you use HashSet?

Whenever uniqueness matters or fast membership checking is required.

---

### Why isn't LinkedList used more often?

Although insertion is O(1), poor memory locality makes it slower than List for many real-world workloads.

---

### Can Dictionary guarantee O(1)?

No.

Average case is O(1).

Worst case is O(n) due to collisions.

---

# Senior Tips

- Default to `List<T>` unless another collection clearly fits better.
- Use `Dictionary<TKey,TValue>` for lookups.
- Use `HashSet<T>` for uniqueness.
- Don't choose collections based only on Big O; consider memory usage and CPU cache locality.
- A good `GetHashCode()` implementation is critical for Dictionary performance.

---

# Summary

- `List<T>` is a dynamic array.
- `Dictionary<TKey,TValue>` is a hash table.
- `HashSet<T>` stores unique values.
- `Queue<T>` is FIFO.
- `Stack<T>` is LIFO.
- `LinkedList<T>` provides fast insert/remove when the node is known.
- Understand **why** average dictionary lookup is O(1) and why the worst case is O(n).
