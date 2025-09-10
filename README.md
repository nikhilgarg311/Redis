# Redis
(For in detail documentation refer https://www.dragonflydb.io/guides/mastering-redis-cache-from-basic-to-advanced)

## 🚀 Redis Caching: Beginner to Advanced

------------------------------------------------------------------------

## 🔹 1. What is Redis?

-   **Redis** = Remote Dictionary Server\
-   It's an **in-memory key-value data store**.\
-   Super fast (because it runs in memory, not disk).\
-   Supports **strings, lists, sets, hashes, sorted sets, streams,
    bitmaps, HyperLogLog, geospatial indexes**.\
-   Commonly used for:
    -   **Caching**
    -   **Session store**
    -   **Real-time analytics**
    -   **Pub/Sub messaging**

👉 For caching, Redis is extremely popular because: - Low latency
(microseconds). - Supports **TTL (time-to-live)** for automatic
expiration. - Scales horizontally with clustering.

------------------------------------------------------------------------

## 🔹 2. Why Use Redis for Caching?

Caching means **storing frequently accessed data temporarily** to reduce
database load and improve speed.

### Example:

Without cache:

    User → App → Database → Response (slow, DB hit every time)

With Redis cache:

    User → App → Redis Cache → Fast Response
                ↘ DB (only when cache miss)

**Benefits:** - Faster response time - Less load on DB - Handles spikes
in traffic - Reduces cost (fewer DB queries)

------------------------------------------------------------------------

## 🔹 3. Basic Caching Pattern

### Step 1: Store data in Redis

``` bash
SET user:101 '{"id":101, "name":"Alice", "age":25}' EX 3600
```

-   Key: `user:101`
-   Value: JSON string
-   `EX 3600`: expires in 1 hour

### Step 2: Get data from Redis

``` bash
GET user:101
```

### Step 3: On cache miss → fetch from DB → update Redis

------------------------------------------------------------------------

## 🔹 4. Caching Strategies

### ✅ Read-Through Cache

-   App asks cache first.
-   If miss → fetch from DB → populate cache.
-   Transparent to app.

### ✅ Write-Through Cache

-   When writing data, write to both DB and cache.
-   Cache is always updated, but write latency is higher.

### ✅ Write-Behind (Write-Back) Cache

-   App writes to cache only.
-   Cache updates DB asynchronously later.
-   Faster, but risk of data loss if Redis crashes.

### ✅ Cache Aside (Lazy Loading) → **Most Common**

-   App looks in cache.
-   If miss → get from DB and update cache.
-   Cache updated only when needed.

------------------------------------------------------------------------

## 🔹 5. Cache Expiration & Eviction

Redis supports **TTL (Time-To-Live)**:

``` bash
SET page:home "HTML_CONTENT" EX 60  # expires in 60s
```

### Eviction Policies (when memory full):

-   **noeviction** → return error when memory full.
-   **allkeys-lru** → evict least recently used key.
-   **volatile-lru** → evict least recently used among keys with TTL.
-   **allkeys-random** → evict random keys.
-   **volatile-ttl** → evict keys with shortest TTL first.

👉 Choose policy in `redis.conf`:

``` bash
maxmemory-policy allkeys-lru
```

------------------------------------------------------------------------

## 🔹 6. Intermediate Concepts

### 🔸 Connection Pooling

Redis is single-threaded, but it's fast. Use a **connection pool** for
efficient connections.

### 🔸 Serialization

-   Store objects as JSON / MsgPack / Protobuf.
-   Example (Node.js):

``` js
await redis.set("user:101", JSON.stringify(user), "EX", 3600);
```

### 🔸 Cache Invalidation

-   **Explicit delete** when data changes:

``` bash
DEL user:101
```

-   Or use **pub/sub** to notify services about updates.

------------------------------------------------------------------------

## 🔹 7. Advanced Concepts

### 🔸 Distributed Caching

-   Multiple app servers → single Redis instance (or cluster).
-   **Redis Cluster** supports **sharding** automatically.

### 🔸 Cache Stampede Problem

-   When cache expires → many requests hit DB at once.
-   Solutions:
    -   **Mutex Locking**: only one process fetches from DB.
    -   **Staggered Expiration**: add random buffer to TTL.
    -   **Cache Aside with Double-Checked Locking**.

### 🔸 Hot Keys Problem

-   Some keys are accessed too frequently → single-node bottleneck.
-   Solutions:
    -   Shard data.
    -   Replicate Redis.
    -   Use CDN for static hot data.

### 🔸 Near-Cache

-   Keep a small in-memory cache at the **application level** (e.g.,
    Guava/Node.js LRU cache) to avoid even Redis call.

### 🔸 Multi-Level Caching

-   L1 Cache → in-app memory (fastest, but small).
-   L2 Cache → Redis (shared, scalable).
-   L3 Cache → Database.

### 🔸 Persistence in Redis

-   **RDB (snapshotting)** → periodic backups.
-   **AOF (Append Only File)** → logs every write.
-   Ensures Redis cache survives restart (but many times cache is
    treated as ephemeral).

------------------------------------------------------------------------

## 🔹 8. Example: Node.js + Redis Caching

``` js
import express from "express";
import { createClient } from "redis";
import fetch from "node-fetch";

const app = express();
const redisClient = createClient();
await redisClient.connect();

app.get("/user/:id", async (req, res) => {
  const { id } = req.params;

  // 1. Check cache
  const cached = await redisClient.get(`user:${id}`);
  if (cached) {
    return res.json(JSON.parse(cached)); // Cache hit
  }

  // 2. Fetch from DB (simulate API)
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`);
  const user = await response.json();

  // 3. Store in cache with 1h expiry
  await redisClient.setEx(`user:${id}`, 3600, JSON.stringify(user));

  res.json(user);
});

app.listen(3000, () => console.log("Server running"));
```

------------------------------------------------------------------------

## 🔹 9. Monitoring Redis Cache

-   Use **Redis CLI**:

    ``` bash
    redis-cli monitor
    info memory
    info stats
    ```

-   Tools:

    -   RedisInsight (GUI from Redis Labs)
    -   Grafana + Prometheus

------------------------------------------------------------------------

## 🔹 10. When NOT to Use Redis for Caching

-   Extremely large datasets that don't fit in memory.
-   Strong consistency requirements (Redis eventual consistency in
    clustering).
-   When persistence is critical (better to use DB).

------------------------------------------------------------------------

# 🎯 Summary

-   **Beginner**: Understand key-value, GET/SET, TTL.\
-   **Intermediate**: Learn strategies (cache-aside, write-through),
    eviction, serialization.\
-   **Advanced**: Handle cache stampede, hot keys, clustering,
    persistence, multi-level caching.
