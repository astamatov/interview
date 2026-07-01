# System Design Case Study - Search Autocomplete

## Problem Statement

Design an autocomplete service similar to:

- Google Search
- Amazon Search
- YouTube Search
- LinkedIn Search

The system should suggest search terms as the user types.

Example:

```text
User types:

"do"

↓

Suggestions:

docker
dotnet
domain driven design
download manager
```

Suggestions should appear within a few milliseconds.

---

# 1. Functional Requirements

The system should:

- Return suggestions while typing
- Rank suggestions by relevance
- Support millions of search terms
- Learn from user searches
- Handle spelling variations
- Support multiple languages

---

# 2. Non-Functional Requirements

The system should provide:

- Very low latency (<100 ms)
- High availability
- High read throughput
- Horizontal scalability

Autocomplete is an extremely read-heavy system.

---

# 3. High-Level Architecture

```text
              Client
                 │
                 ▼
          API Gateway
                 │
                 ▼
      Autocomplete Service
                 │
        ┌────────┴────────┐
        ▼                 ▼
      Redis             Trie
        │                 │
        └────────┬────────┘
                 ▼
           Search Database
```

---

# 4. Why Not Use SQL LIKE?

Bad approach:

```sql
WHERE Name LIKE 'do%'
```

With millions of rows this becomes expensive.

Autocomplete requires specialized indexing structures.

---

# 5. Trie

## Interview Answer

A Trie (Prefix Tree) stores words by character.

Example:

```text
root
 ├── d
 │   └── o
 │       ├── c
 │       │    └── k...
 │       └── t
 │            └── n...
```

Searching for:

```
do
```

takes approximately O(length of prefix).

This is much faster than scanning every word.

---

# 6. Ranking Suggestions

Suggestions should not be alphabetical only.

Possible ranking factors:

- Search frequency
- Recent popularity
- User location
- Personal search history
- Trending searches

Example:

```
dotnet

100,000 searches

↓

Higher Rank
```

---

# 7. Caching

Frequently requested prefixes should be cached.

Example:

```text
"d"

↓

Redis

↓

Suggestions
```

Popular prefixes account for a large percentage of traffic.

---

# 8. Updating the Index

New searches continuously arrive.

Flow:

```text
Search

↓

Queue

↓

Index Worker

↓

Update Trie
```

Updating asynchronously keeps search latency low.

---

# 9. Debouncing

Clients should not send requests for every keystroke.

Instead:

```text
D

DO

DOT

↓

Wait 200 ms

↓

Send Request
```

Debouncing reduces unnecessary traffic.

---

# 10. Top-K Suggestions

The Trie can store the most popular suggestions at each node.

Example:

Prefix:

```
do
```

Node contains:

```text
docker
dotnet
domain
download
```

This avoids traversing the entire subtree on every request.

---

# 11. Distributed Scaling

One Trie cannot always fit in memory.

Possible approaches:

- Partition by first letter
- Partition by language
- Hash-based partitioning

Example:

```text
A-F

↓

Server 1

---------------

G-M

↓

Server 2

---------------

N-Z

↓

Server 3
```

---

# 12. Common Interview Questions

### Why use a Trie?

## Answer

Because Trie lookup depends on the prefix length rather than the total number of stored words.

This makes prefix searches extremely efficient.

---

### Why cache autocomplete?

Popular prefixes such as:

```
a

t

s
```

are requested constantly.

Caching greatly reduces latency and backend load.

---

### Why update asynchronously?

Autocomplete does not require immediate consistency.

Background updates improve throughput and responsiveness.

---

### Why debounce requests?

Without debouncing:

```
"D"

↓

"DO"

↓

"DOT"

↓

"DOTN"

↓

"DOTNE"

↓

"DOTNET"
```

would generate six backend requests.

Debouncing often reduces this to one.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Trie | Fast prefix search | Memory usage |
| Redis | Low latency | Cache invalidation |
| Top-K Cache | Faster suggestions | More memory |
| Async Updates | Better throughput | Eventual consistency |

---

# Staff Engineer Insight

A common mistake is optimizing only search speed.

Autocomplete quality matters just as much.

Users expect suggestions that are:

- Relevant
- Popular
- Personalized
- Fresh

Another common mistake is assuming autocomplete and full-text search are the same problem.

Autocomplete optimizes prefix matching.

Full-text search optimizes relevance across complete documents.

They often use different data structures and storage engines.

---

# Summary

- Trie is the standard structure for autocomplete.
- Prefix search complexity depends on prefix length.
- Cache popular prefixes in Redis.
- Rank suggestions using popularity and relevance.
- Update indexes asynchronously.
- Use debouncing to reduce request volume.
- Scale by partitioning the autocomplete index.
- Treat autocomplete as a low-latency, read-heavy service.
