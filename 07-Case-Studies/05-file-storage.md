# 📁 File Storage System (Google Drive)

## 1. Problem Statement

Design a cloud file storage system like **Google Drive** or **Dropbox** that allows users to upload, store, and sync files across devices.

---

## 2. Clarifying Questions (Ask First)

* What file types? (Documents, images, videos, any)
* Maximum file size? (10MB, 100MB, 5GB)
* What operations? (Upload, download, delete, share, versioning)
* Real-time sync? (Changes sync across devices immediately)
* Collaboration? (Multiple users edit same file)
* Storage quota? (Free tier limits)

---

## 3. Requirements

### Functional

* Upload files
* Download files
* Delete files
* List files (with pagination)
* Search files
* Share files/folders
* Version history
* Real-time sync (optional)

### Non-Functional

* High availability (99.99%)
* Durable storage (files never lost)
* Low latency downloads (<100ms to start)
* Scalable to petabytes
* Cost-effective

---

## 4. Scale Assumptions

* 1B users
* 100M daily active users
* 10B files stored
* Average file size: 10MB
* Total storage: 10B × 10MB = 100PB
* Upload: 1M files/day = ~100GB/sec peak
* Download: 10:1 read/write ratio = 10M downloads/day
* Peak QPS: 10K uploads, 100K downloads

---

## 5. High-Level Architecture

```
Client → Load Balancer → API Gateway
                            │
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
    Metadata Service   Upload Service   Download Service
            │               │               │
            ↓               ↓               ↓
    Metadata DB      Object Storage    CDN (Hot Files)
    (SQL/NoSQL)      (S3/GCS)         (CloudFront)
```

---

## 6. Core Components

* **API Gateway**: Routes requests, handles auth
* **Metadata Service**: Manages file metadata (name, size, owner, permissions)
* **Upload Service**: Handles file uploads (chunked for large files)
* **Download Service**: Handles file downloads
* **Object Storage**: Stores actual file data (S3, GCS, Azure Blob)
* **CDN**: Caches popular files for fast downloads
* **Metadata Database**: Stores file metadata (SQL for relationships, or NoSQL for scale)
* **Search Service**: Indexes file names/content for search
* **Version Service**: Manages file versions
* **Sync Service**: Handles real-time sync (WebSocket/SSE)

---

## 7. Data Flow

**Upload Flow (Small File <100MB)**
1. Client → POST /api/files/upload {file, metadata}
2. API Gateway → Authenticate user
3. Upload Service → Validate file (size, type, quota)
4. Upload Service → Generate unique file_id
5. Upload Service → Upload to object storage (S3) in chunks
6. Upload Service → Store metadata in DB (file_id, user_id, name, size, path, created_at)
7. Upload Service → Return file_id to client
8. Background → Index file for search (if needed)

**Upload Flow (Large File >100MB)**
1. Client → POST /api/files/upload {metadata} → Get upload_url
2. Client → Upload file in chunks (e.g., 5MB chunks) to upload_url
3. Upload Service → Store chunks in object storage
4. Client → POST /api/files/complete {file_id, chunk_ids}
5. Upload Service → Combine chunks (or use multipart upload)
6. Upload Service → Store metadata in DB
7. Upload Service → Return success

**Download Flow**
1. Client → GET /api/files/{file_id}/download
2. API Gateway → Authenticate, check permissions
3. Download Service → Query metadata DB for file path
4. Download Service → Check CDN cache
5. If cache hit → Return CDN URL (redirect)
6. If cache miss → Generate signed URL for object storage
7. Client → Downloads directly from object storage or CDN
8. Background → Cache popular files in CDN

**List Files Flow**
1. Client → GET /api/files?folder_id=xyz&page=1
2. Metadata Service → Query DB (SELECT * FROM files WHERE folder_id = xyz ORDER BY created_at LIMIT 50)
3. Metadata Service → Return paginated results
4. Client → Renders file list

**Search Flow**
1. Client → GET /api/files/search?q=document
2. Search Service → Query search index (Elasticsearch)
3. Search Service → Return matching files (with relevance score)
4. Client → Displays results

**Sync Flow (Real-time)**
1. User A → Uploads file on Device 1
2. Upload Service → Stores file, publishes event
3. Sync Service → Detects change, pushes to Device 2 via WebSocket
4. Device 2 → Receives update, syncs file
5. User A → Sees same file on both devices

---

## 8. Bottlenecks

* **Large File Uploads**: 5GB file uploads timeout
  * Solution: Chunked uploads, resumable uploads, background processing
* **Metadata Database**: 10B files, complex queries
  * Solution: Shard by user_id, use read replicas, cache hot metadata
* **Object Storage Bandwidth**: 100GB/sec uploads
  * Solution: Distribute across regions, use multipart uploads
* **CDN Cache**: Popular files not cached
  * Solution: Pre-warm CDN for popular files, longer TTL
* **Search Indexing**: Slow for 10B files
  * Solution: Async indexing, incremental updates, shard search index
* **Version Storage**: Storing all versions is expensive
  * Solution: Keep last N versions, archive old versions to cold storage
* **Sync Conflicts**: Multiple users edit same file
  * Solution: Last-write-wins, or operational transformation for real-time collaboration

---

## 9. Trade-offs

| Decision | Choice | Why | Trade-off |
|----------|--------|-----|-----------|
| **Object Storage** | S3/GCS (managed) | Scalable, durable, cost-effective | Vendor lock-in, but acceptable |
| **Metadata DB** | SQL (PostgreSQL) with read replicas | Complex queries (folders, sharing), ACID | May need sharding at very large scale |
| **File Chunking** | 5MB chunks for large files | Resumable, handles network issues | More API calls, but necessary |
| **CDN Strategy** | Cache popular files, direct for others | Reduces bandwidth costs | May serve stale files (rare updates) |
| **Versioning** | Keep last 10 versions, archive rest | Balances storage cost and functionality | Old versions in cold storage (slower access) |
| **Search** | Elasticsearch for full-text | Fast, scalable search | Additional service to maintain |

**Key Trade-off:**
> "I use object storage (S3) for file data because it's scalable to petabytes and cost-effective. The trade-off is eventual consistency for metadata updates, but file data itself is immutable once uploaded, so this is acceptable. For metadata, I use SQL for complex queries (folders, sharing), with read replicas for scale."

---

## 10. How to Explain This in an Interview

> "I'll design a file storage system with separate services for metadata and file data, using object storage for scalability.
>
> For architecture, I separate metadata (file names, permissions, folder structure) from file data (actual bytes). Metadata goes in a SQL database for complex queries like folder hierarchies and sharing. File data goes in object storage (S3) for scalability to petabytes.
>
> For uploads, I use chunked uploads for files >100MB. The client uploads in 5MB chunks, and the server combines them. This handles network issues and allows resumable uploads. For small files, I upload directly.
>
> For downloads, I use a CDN to cache popular files. When a user requests a file, I check the CDN first. If it's cached, I redirect to the CDN URL. If not, I generate a signed URL for direct download from object storage and cache it in the CDN for future requests.
>
> For metadata, I shard the database by user_id to handle 1B users. I use read replicas for list/search queries. I cache hot metadata (recently accessed files) in Redis to reduce database load.
>
> For search, I use Elasticsearch to index file names and content. When a file is uploaded, I asynchronously index it. Search queries go to Elasticsearch, which returns file_ids, then I fetch metadata from the database.
>
> For versioning, I store each version as a separate object in storage with a version_id. I keep the last 10 versions in hot storage and archive older versions to cold storage (cheaper, slower access).
>
> The key trade-off is separating metadata and file data: metadata in SQL for queries, file data in object storage for scale. This allows independent scaling of each component."

**Key Phrases:**
* "Separate metadata from file data for independent scaling"
* "Object storage for petabytes of file data"
* "Chunked uploads for large files"
* "CDN for popular files, direct storage for others"

---

## 11. Common Mistakes

* **Storing files in database**: "We store file bytes in DB"
  * Fix: Use object storage for files, DB only for metadata
* **No chunking for large files**: "Upload 5GB file in one request"
  * Fix: Chunk large files, support resumable uploads
* **Synchronous indexing**: "We index files during upload"
  * Fix: Async indexing to avoid blocking uploads
* **No CDN**: "All downloads from object storage"
  * Fix: Use CDN for popular files to reduce bandwidth costs
* **Single metadata database**: "One DB for 10B files"
  * Fix: Shard by user_id, use read replicas
* **No versioning strategy**: "We store all versions forever"
  * Fix: Keep last N versions, archive old ones
* **Not handling sync conflicts**: "Multiple users edit, data lost"
  * Fix: Last-write-wins or operational transformation
