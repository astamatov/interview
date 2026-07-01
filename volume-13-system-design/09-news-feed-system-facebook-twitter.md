# System Design Case Study - News Feed System (Facebook / X / LinkedIn)

## Problem Statement

Design a News Feed system similar to:

- Facebook
- X (Twitter)
- LinkedIn

The system should allow users to:

- Publish posts
- Follow other users
- View a personalized feed
- Like posts
- Comment on posts
- Share posts

The primary challenge is generating a feed quickly while supporting millions of users.

---

# 1. Functional Requirements

The system should support:

- Create posts
- Delete posts
- Follow/Unfollow users
- Personalized news feed
- Likes
- Comments
- Shares

---

# 2. Non-Functional Requirements

The system should provide:

- Low feed generation latency
- High availability
- Horizontal scalability
- High write throughput
- High read throughput

Read traffic is significantly higher than write traffic.

---

# 3. High-Level Architecture

```text
                 Client
                    │
                    ▼
              API Gateway
                    │
      ┌─────────────┼─────────────┐
      ▼             ▼             ▼
 Post Service  Feed Service  User Service
      │             │             │
      └─────────────┼─────────────┘
                    ▼
             Message Broker
                    ▼
            Feed Workers
                    ▼
          Feed Storage / Cache
```

---

# 4. Database Design

Posts:

```text
PostId
AuthorId
Content
CreatedAt
Visibility
```

Followers:

```text
FollowerId
FollowingId
```

Feed:

```text
UserId
PostId
CreatedAt
```

---

# 5. Feed Generation Strategies

There are two common approaches.

---

## Fan-Out on Write

When a user publishes a post:

```text
Create Post
      ↓
Find Followers
      ↓
Insert Post Into Each Feed
```

Reads become extremely fast.

However, publishing becomes more expensive.

---

## Fan-Out on Read

The feed is built only when the user opens the application.

```text
Open Feed
      ↓
Read Posts
      ↓
Merge
      ↓
Sort
      ↓
Return
```

Writes are cheap.

Reads are more expensive.

---

# Interview Question

### Which strategy is better?

## Answer

Neither is always better.

Most large platforms use a hybrid approach.

Normal users:

```
Fan-Out on Write
```

Celebrities:

```
Fan-Out on Read
```

Otherwise, posting by a celebrity with 50 million followers would generate enormous write traffic.

---

# 6. Feed Ranking

The feed should not simply sort by time.

Possible ranking signals:

- Recency
- Likes
- Comments
- Shares
- User interests
- Relationship strength
- Watch time
- Click history

Modern feeds are ranking systems rather than chronological lists.

---

# 7. Caching

Frequently accessed feeds should be cached.

Example:

```text
User Feed

↓

Redis
```

This avoids rebuilding the same feed repeatedly.

---

# 8. Likes and Comments

Likes should not update the feed synchronously.

Instead:

```text
Like

↓

Queue

↓

Counter Update

↓

Analytics
```

This keeps user interactions responsive.

---

# 9. Timeline Pagination

Instead of returning:

```text
10000 Posts
```

Return:

```text
20 Posts

↓

Cursor

↓

Next 20
```

Cursor-based pagination scales better than OFFSET for continuously changing feeds.

---

# 10. Deleting Posts

Deleting a post requires:

- Removing metadata
- Removing cached feed entries
- Invalidating CDN (if media exists)
- Updating search indexes

Many systems perform these operations asynchronously.

---

# 11. Scaling

```text
Load Balancer

↓

Feed Service

↓

Redis

↓

Database
```

Background workers continuously update cached feeds.

---

# 12. Common Interview Questions

### Why not generate every feed in real time?

## Answer

Building feeds from millions of posts on every request would be too expensive.

Caching and precomputed feeds reduce latency.

---

### Why not always use Fan-Out on Write?

Because users with millions of followers generate enormous write amplification.

---

### Why use cursor pagination?

Cursor pagination performs better on large datasets and avoids inconsistencies caused by OFFSET while new posts are inserted.

---

### Why use Redis?

Redis stores frequently requested feeds and reduces database load.

---

# Trade-offs

| Strategy | Benefit | Trade-off |
|----------|---------|-----------|
| Fan-Out on Write | Fast reads | Expensive writes |
| Fan-Out on Read | Cheap writes | Slower reads |
| Redis Cache | Low latency | Cache invalidation |
| Cursor Pagination | Better scalability | More implementation complexity |

---

# Staff Engineer Insight

One of the biggest mistakes is assuming a News Feed is simply:

```sql
SELECT *
FROM Posts
ORDER BY CreatedAt DESC
```

Real news feeds are recommendation systems.

The challenge is balancing:

- Freshness
- Relevance
- Scalability
- Personalization

Another common mistake is treating all users equally.

Most users have hundreds of followers.

A few users have millions.

Large social platforms optimize separately for these two groups because their traffic patterns are fundamentally different.

---

# Summary

- Separate post creation from feed generation.
- Use Fan-Out on Write for most users.
- Consider Fan-Out on Read for high-fanout accounts.
- Cache feeds using Redis.
- Rank feeds using multiple relevance signals.
- Process likes and comments asynchronously where appropriate.
- Use cursor-based pagination.
- Design the system around read-heavy workloads.
