# System Design Case Study - File Storage System (Dropbox / Google Drive)

## Problem Statement

Design a cloud file storage service similar to:

- Dropbox
- Google Drive
- OneDrive

The system should allow users to:

- Upload files
- Download files
- Synchronize files across devices
- Share files
- Store multiple versions
- Support very large files

---

# 1. Functional Requirements

The system should support:

- File upload
- File download
- Delete file
- Rename file
- Folder hierarchy
- File sharing
- Version history
- Resume interrupted uploads

---

# 2. Non-Functional Requirements

The system should provide:

- High availability
- Durability
- Horizontal scalability
- Low latency downloads
- Secure storage

Files must never be lost.

---

# 3. High-Level Architecture

```
            Client

               │

               ▼

        API Gateway

               │

     Metadata Service

               │

         SQL Database

               │

──────── File Upload ────────

               ▼

       Blob Storage

               │

               ▼

             CDN
```

Metadata and file contents should be stored separately.

---

# 4. Metadata Storage

Database stores:

```
FileId

OwnerId

FileName

Size

ContentType

StoragePath

Version

CreatedAt

ModifiedAt
```

Actual file bytes belong in object storage.

---

# 5. Blob Storage

Files should be stored in:

- Azure Blob Storage
- Amazon S3
- Google Cloud Storage

Object storage provides:

- High durability
- Scalability
- Low operational overhead

Databases should not store large binary files.

---

# Interview Question

### Why not store files in SQL Server?

## Answer

Databases are optimized for structured data.

Large binary files:

- Increase backup size
- Reduce database performance
- Increase storage costs

Object storage is designed specifically for this workload.

---

# 6. Upload Flow

```
Client

↓

API

↓

Metadata Validation

↓

Generate Upload URL

↓

Blob Storage

↓

Upload File

↓

Save Metadata
```

The application should avoid proxying large files whenever possible.

---

# 7. Pre-Signed URLs / SAS Tokens

Instead of:

```
Client

↓

Application

↓

Blob Storage
```

Use:

```
Client

↓

Blob Storage
```

The API generates a temporary secure upload URL.

Benefits:

- Lower application load
- Better scalability
- Faster uploads

---

# 8. Chunked Upload

Large files should be split into chunks.

```
File

↓

Chunk 1

Chunk 2

Chunk 3

Chunk 4
```

Advantages:

- Resume uploads
- Retry only failed chunks
- Parallel uploads

---

# 9. Resumable Uploads

If upload fails:

```
Uploaded:

Chunk 1

Chunk 2

Missing:

Chunk 3

Chunk 4
```

Resume from the first missing chunk instead of restarting.

---

# 10. File Download

Flow:

```
Client

↓

API

↓

Metadata

↓

Temporary Download URL

↓

Blob Storage

↓

Download
```

Again, the application avoids streaming the file itself.

---

# 11. CDN

Frequently downloaded files should be served through a CDN.

```
Blob Storage

↓

CDN

↓

User
```

Benefits:

- Lower latency
- Reduced storage load
- Global distribution

---

# 12. File Versioning

Every update creates a new version.

Example:

```
Document

↓

v1

↓

v2

↓

v3
```

Users can restore previous versions.

---

# 13. Deduplication

Multiple users may upload identical files.

Calculate:

```
SHA-256 Hash

↓

Already Exists?

↓

Reuse Blob
```

Store only one physical copy.

Benefits:

- Lower storage cost
- Faster uploads

---

# 14. Virus Scanning

Uploads should not become immediately available.

```
Upload

↓

Temporary Storage

↓

Virus Scan

↓

Approved

↓

Visible
```

Scanning should happen asynchronously.

---

# 15. Common Interview Questions

### Why chunk uploads?

## Answer

Large uploads become more reliable.

Only failed chunks need retransmission.

---

### Why use Blob Storage?

Because object storage is optimized for storing large files with high durability.

---

### Why generate temporary upload URLs?

The application avoids becoming a bottleneck for large file transfers.

---

### Why store metadata separately?

Metadata queries are frequent.

File downloads are relatively infrequent.

Separating them improves scalability.

---

# Trade-offs

| Decision | Benefit | Trade-off |
|----------|---------|-----------|
| Blob Storage | Durable and scalable | External dependency |
| Chunk Upload | Reliable large uploads | More implementation complexity |
| CDN | Faster downloads | Cache invalidation |
| Deduplication | Lower storage cost | Hash computation |
| Versioning | Easy recovery | Increased storage usage |

---

# Staff Engineer Insight

A common mistake is routing every upload through the application server.

For a 5 GB upload, the application would unnecessarily consume:

- Network bandwidth
- CPU
- Memory

Generating temporary upload URLs allows clients to communicate directly with object storage.

Another common mistake is assuming uploads finish successfully.

Real systems must handle:

- Interrupted connections
- Duplicate uploads
- Partial uploads
- Client retries
- Storage failures

Design for failure from the beginning.

---

# Summary

- Store metadata and file contents separately.
- Use object storage for large files.
- Generate temporary upload/download URLs.
- Support chunked and resumable uploads.
- Use a CDN for downloads.
- Implement file versioning.
- Deduplicate identical files using hashes.
- Scan uploads before making them available.
- Design uploads to tolerate interruptions and retries.
