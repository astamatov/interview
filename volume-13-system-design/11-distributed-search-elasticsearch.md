# System Design Case Study - Distributed Search (Elasticsearch)

## Problem Statement

Design a distributed search system similar to those used by:

- Amazon
- LinkedIn
- GitHub
- Stack Overflow

The system should support:

- Full-text search
- Fast filtering
- Sorting
- Pagination
- Relevance ranking
- High availability
- Horizontal scalability

Unlike autocomplete, this system searches complete documents rather than prefixes.

---

# 1. Functional Requirements

The system should support:

- Full-text search
- Filtering
- Sorting
- Highlighting
- Faceted search
- Near real-time indexing

Example:

```
Search:

"distributed cache"

↓

Relevant documents
```

---

# 2. Non-Functional Requirements

The system should provide:

- Low latency (<200 ms)
- High availability
- Horizontal scalability
- Fault tolerance
- High read throughput

---

# 3. High-Level Architecture

```text
               Client
                  │
                  ▼
            API Gateway
                  │
                  ▼
          Search Service
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
     Elasticsearch      Redis
        │
        ▼
 Indexing Pipeline
        │
        ▼
 Primary Database
```

The relational database remains the source of truth.

Elasticsearch is an optimized search index.

---

# 4. Why Not Use SQL?

Bad approach:

```sql
WHERE Description LIKE '%distributed cache%'
```

Problems:

- Slow on large datasets
- Poor relevance
- Limited linguistic support

Search engines are specifically optimized for text retrieval.

---

# 5. Inverted Index

## Interview Answer

Traditional databases organize data by rows.

Search engines organize data by words.

Example:

Documents:

```text
Doc1

Distributed Cache

Doc2

Distributed Database

Doc3

Cache Invalidation
```

Inverted Index:

```text
distributed

↓

Doc1

Doc2

---------------

cache

↓

Doc1

Doc3
```

Searching becomes extremely fast.

---

# 6. Indexing Pipeline

Data should not be written directly into Elasticsearch from every application.

Typical flow:

```text
Application

↓

Database

↓

Message Broker

↓

Indexer

↓

Elasticsearch
```

This decouples indexing from application requests.

---

# 7. Near Real-Time Search

Elasticsearch is not immediately consistent.

Documents become searchable after a refresh.

Typical refresh interval:

```
~1 second
```

This is acceptable for most search workloads.

---

# 8. Shards

## Interview Answer

An index is divided into multiple shards.

Example:

```text
Products

↓

Shard 1

Shard 2

Shard 3
```

Each shard stores part of the data.

This enables horizontal scaling.

---

# 9. Replicas

Each shard may have replicas.

Example:

```text
Primary Shard

↓

Replica 1

↓

Replica 2
```

Benefits:

- Higher availability
- Better read throughput
- Fault tolerance

---

# 10. Relevance Scoring

Results are not simply sorted alphabetically.

Ranking considers:

- Keyword frequency
- Field importance
- Term rarity
- Match quality

Modern Elasticsearch versions primarily use the BM25 ranking algorithm.

The highest-scoring documents appear first.

---

# 11. Filters vs Queries

Example:

```text
Query

↓

Find relevant documents

---------------

Filter

↓

Restrict results
```

Filters:

- Country
- Category
- Price

Filters are often cacheable and faster because they do not calculate relevance scores.

---

# 12. Aggregations

Example:

Search:

```
Laptop
```

Aggregations:

```text
Dell (120)

HP (90)

Lenovo (70)
```

These power faceted search experiences.

---

# 13. Common Interview Questions

### Why use Elasticsearch?

## Answer

It is optimized for full-text search, relevance ranking and distributed querying.

---

### Why not query the database directly?

Relational databases are optimized for transactions.

Search engines are optimized for text retrieval.

Each technology serves a different purpose.

---

### Why use shards?

Shards distribute both storage and query processing across multiple nodes.

---

### Why use replicas?

Replicas improve availability and allow read requests to be distributed.

---

### Why use asynchronous indexing?

Search systems usually tolerate eventual consistency.

Asynchronous indexing improves application responsiveness.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Inverted Index | Extremely fast search | Larger index size |
| Shards | Horizontal scaling | More coordination |
| Replicas | Fault tolerance | Additional storage |
| Async Indexing | Faster writes | Eventual consistency |
| Filters | Faster execution | Less flexible than scoring queries |

---

# Staff Engineer Insight

A common mistake is treating Elasticsearch as the primary database.

It is not.

Elasticsearch should be considered a **derived view** of the data.

The primary database remains the source of truth.

If the search index is lost, it should be rebuildable from the database.

Another common mistake is trying to model complex transactional workflows inside Elasticsearch.

Use the right tool for the right job:

- SQL for transactions.
- Elasticsearch for search.
- Redis for caching.

Avoid forcing one technology to solve every problem.

---

# Summary

- Elasticsearch is designed for distributed full-text search.
- It uses an inverted index instead of row-based storage.
- Shards distribute data horizontally.
- Replicas improve availability and read scalability.
- BM25 provides relevance ranking.
- Filters and queries serve different purposes.
- Aggregations power faceted search.
- Keep the primary database as the source of truth.
- Use asynchronous indexing to improve responsiveness.
