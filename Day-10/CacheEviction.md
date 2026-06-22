# Cache Eviction Policies

When the cache becomes full and a new item needs to be inserted, the cache must decide which existing item should be removed.

This process is called:

```text
Cache Eviction
```

The strategy used to choose the item to remove is called:

```text
Cache Eviction Policy
```

---

# Why Do We Need Eviction Policies?

Cache is usually stored in:

```text
RAM
```

RAM is:

```text
Fast

Expensive

Limited
```

Eventually the cache reaches its maximum capacity.

Example:

```text
Cache Capacity = 1000 Items

Current Items = 1000
```

A new item arrives.

Question:

```text
Which Item Should Be Removed?
```

This is solved using cache eviction policies.

---

# FIFO (First In First Out)

## Rule

Remove the oldest item in the cache.

---

## Example

Current Cache:

```text
A B C D E
```

New Item:

```text
F
```

Remove:

```text
A
```

Result:

```text
B C D E F
```

---

## Advantages

```text
Simple

Easy To Implement
```

---

## Disadvantages

```text
Does Not Consider Usage Patterns

May Remove Frequently Accessed Data
```

---

# LRU (Least Recently Used)

## Rule

Remove the item that has not been accessed for the longest period of time.

---

## Example

Cache:

```text
A B C D
```

Access Pattern:

```text
A

B

A

D
```

Recent Usage Order:

```text
D

A

B
```

Least Recently Used:

```text
C
```

New Item:

```text
E
```

Remove:

```text
C
```

Result:

```text
A B D E
```

---

## Assumption

```text
Recently Accessed Data

↓

Likely To Be Accessed Again
```

---

## Real World Examples

```text
Instagram Feed

Product Pages

User Sessions

News Feed
```

---

## Advantages

```text
Works Well For Most Applications

Adapts To User Behaviour
```

---

## Disadvantages

```text
Popular Old Data May Be Removed
```

---

# LFU (Least Frequently Used)

## Rule

Remove the item that has been accessed the fewest number of times.

---

## Example

Cache Statistics:

```text
A = 100 Hits

B = 50 Hits

C = 2 Hits

D = 1 Hit
```

New Item Arrives.

Remove:

```text
D
```

because it has the lowest frequency.

---

## Assumption

```text
Frequently Accessed Data

↓

Important Data
```

---

## Real World Examples

```text
YouTube Trending Videos

Leaderboards

Popular Products

Hot News Articles
```

---

## Advantages

```text
Keeps Popular Content Longer

Good For Repeated Access Patterns
```

---

## Disadvantages

Old Popular Data May Stay Forever.

Example:

```text
World Cup Final

1 Million Views Last Month

10 Views Today
```

LFU may still keep it because of its historical frequency.

---

# Random Eviction

## Rule

Choose a random item and remove it.

---

## Example

Cache:

```text
A B C D E
```

Randomly Remove:

```text
C
```

Result:

```text
A B D E F
```

---

## Advantages

```text
Very Simple

Low Overhead
```

---

## Disadvantages

```text
May Remove Important Data

Unpredictable
```

---

# LRU vs LFU

| Feature        | LRU                     | LFU              |
| -------------- | ----------------------- | ---------------- |
| Tracks         | Recent Usage            | Usage Frequency  |
| Question Asked | Who Used Recently?      | Who Used Most?   |
| Best For       | User Behaviour Patterns | Popular Content  |
| Example        | Instagram Feed          | YouTube Trending |

---

## Example

Suppose:

```text
A = Accessed 100 Times Last Month

B = Accessed 5 Times Today
```

### LRU

Keeps:

```text
B
```

because it was accessed recently.

---

### LFU

Keeps:

```text
A
```

because it has a higher frequency count.

---

# Redis Eviction Policies

Redis supports multiple eviction policies.

Commonly used:

```text
allkeys-lru

allkeys-lfu

volatile-lru

volatile-lfu
```

---

## Most Common Interview Answer

If the interviewer asks:

### Cache Is Full. What Will You Do?

Answer:

I would use a cache eviction policy such as LRU or LFU.

LRU removes the least recently used item and works well for most applications because recently accessed data is likely to be accessed again.

LFU removes the least frequently used item and is useful when popularity is more important than recency.

---

# Interview Summary

```text
FIFO
↓
Remove Oldest Item

LRU
↓
Remove Least Recently Used Item

LFU
↓
Remove Least Frequently Used Item

Random
↓
Remove Random Item

Most Common In Industry
↓
LRU

Best For Trending Content
↓
LFU
```
