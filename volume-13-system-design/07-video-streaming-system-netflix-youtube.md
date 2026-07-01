# System Design Case Study - Video Streaming System (Netflix / YouTube)

## Problem Statement

Design a video streaming platform similar to:

- YouTube
- Netflix
- Vimeo

The system should allow users to:

- Upload videos
- Watch videos
- Stream different quality levels
- Resume playback
- Generate thumbnails
- Serve content globally

---

# 1. Functional Requirements

The system should support:

- Video upload
- Video playback
- Multiple resolutions
- Adaptive streaming
- Video metadata
- Watch history
- Continue watching
- Search and discovery

---

# 2. Non-Functional Requirements

The system should provide:

- High availability
- Low startup latency
- Global distribution
- High durability
- Massive read scalability

Video playback traffic is usually much larger than upload traffic.

---

# 3. High-Level Architecture

```text
Client
  │
  ▼
API Gateway
  │
  ▼
Video Metadata Service
  │
  ▼
SQL / NoSQL Database

Upload Path:
Client
  │
  ▼
Blob Storage
  │
  ▼
Message Queue
  │
  ▼
Transcoding Workers
  │
  ▼
Processed Video Storage
  │
  ▼
CDN

Playback Path:
Client
  │
  ▼
CDN
  │
  ▼
Video Segments
```

---

# 4. Video Upload Flow

```text
User uploads video
  ↓
Video stored in raw storage
  ↓
Message published
  ↓
Transcoding workers process video
  ↓
Multiple resolutions generated
  ↓
Metadata updated
  ↓
Video becomes available
```

Uploads should not block while transcoding runs.

Transcoding is a background process.

---

# 5. Why Use Object Storage?

Videos are large binary objects.

They should be stored in:

- Azure Blob Storage
- Amazon S3
- Google Cloud Storage

Databases should store only metadata.

Examples:

```text
VideoId
OwnerId
Title
Description
Duration
StoragePath
Status
CreatedAt
```

The actual video bytes belong in object storage.

---

# 6. Transcoding

## Interview Answer

Transcoding converts uploaded videos into multiple formats and resolutions.

Example:

```text
Original video

↓

1080p

720p

480p

360p
```

This allows users on different devices and networks to stream the most appropriate version.

---

# 7. Why Transcoding is Asynchronous

Transcoding can take seconds, minutes or even hours.

It should never happen inside the upload request.

Correct design:

```text
Upload
  ↓
Queue message
  ↓
Return response
  ↓
Process in background
```

This keeps the API responsive.

---

# 8. Adaptive Bitrate Streaming

## Interview Answer

Adaptive Bitrate Streaming allows the player to switch video quality based on network conditions.

Example:

```text
Good network → 1080p

Medium network → 720p

Poor network → 360p
```

Common protocols:

- HLS
- MPEG-DASH

The video is split into small segments.

The player downloads the best segment quality it can support.

---

# 9. Video Segments

Instead of downloading one huge file:

```text
movie.mp4
```

the system stores many small chunks:

```text
segment_001.ts
segment_002.ts
segment_003.ts
```

Benefits:

- Faster startup
- Easier seeking
- Adaptive quality switching
- Better CDN caching

---

# 10. CDN

A CDN is critical for video streaming.

```text
Origin Storage
  ↓
CDN Edge Location
  ↓
User
```

Benefits:

- Lower latency
- Reduced origin load
- Global distribution
- Better playback experience

Most video traffic should be served from CDN edge locations.

---

# 11. Playback Flow

```text
Client requests video
  ↓
API returns manifest URL
  ↓
Player downloads manifest
  ↓
Player requests video segments from CDN
  ↓
Playback starts
```

The application API should not stream video bytes directly.

---

# 12. Manifest File

The manifest describes available video qualities.

Example:

```text
1080p
720p
480p
360p
```

The player uses the manifest to choose the correct stream.

---

# 13. Watch History

Track:

```text
UserId
VideoId
Position
UpdatedAt
```

This enables:

- Continue watching
- Recommendations
- Analytics

Watch progress can be written asynchronously or batched to reduce database load.

---

# 14. Thumbnails

Thumbnail generation should also run in the background.

Flow:

```text
Video uploaded
  ↓
Worker extracts frames
  ↓
Thumbnails stored
  ↓
Metadata updated
```

Thumbnails are served through CDN.

---

# 15. Recommendation System

At a high level, recommendations may use:

- Watch history
- Likes
- Search history
- Similar users
- Trending videos

For system design interviews, a basic recommendation pipeline is enough unless the interviewer asks for ML details.

---

# 16. Common Interview Questions

### Why not stream directly from the application server?

## Answer

Streaming video through the application server would consume huge bandwidth and make the API layer the bottleneck.

CDN and object storage are designed for this workload.

---

### Why split videos into segments?

## Answer

Segments allow adaptive streaming, faster startup and better CDN caching.

---

### Why use a queue for transcoding?

## Answer

Transcoding is slow and CPU-intensive.

Queues decouple upload from processing and allow workers to scale independently.

---

### Why generate multiple resolutions?

## Answer

Different users have different devices and network speeds.

Multiple resolutions improve playback quality and reduce buffering.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Object Storage | Durable large-file storage | External dependency |
| CDN | Low-latency global playback | Cache invalidation |
| Transcoding | Device/network compatibility | High compute cost |
| Segmentation | Adaptive streaming | More files to manage |
| Async Processing | Responsive uploads | Eventual availability |

---

# Staff Engineer Insight

A common mistake is designing video streaming as simple file download.

Real streaming platforms optimize for:

- Startup time
- Buffering
- Network adaptation
- Global delivery
- CDN hit ratio
- Background processing

Another common mistake is making the upload request responsible for too much work.

Upload should store the raw video and enqueue processing.

Everything else should happen asynchronously.

This makes the system much more scalable and resilient.

---

# Summary

- Store raw and processed videos in object storage.
- Store metadata separately in a database.
- Use background workers for transcoding and thumbnails.
- Generate multiple resolutions.
- Use HLS or DASH for adaptive bitrate streaming.
- Split videos into segments.
- Serve video through CDN.
- Track watch history for continue-watching and recommendations.
- Keep the API layer out of the video byte-serving path.
