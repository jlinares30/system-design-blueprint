# ◇ Caching Strategies

A cache is a high-speed data storage layer that stores a subset of transient data, serving future requests faster than querying the primary database.

---

## ▪ Multi-Tier Caching Architecture

1. **Client-Side Cache (Browser):** HTTP headers (`Cache-Control`, `ETag`) instruct the browser to cache resources locally.
2. **CDN (Content Delivery Network):** Geographically distributed network of proxy servers caching static assets (images, CSS, JS, HTML) and dynamic responses close to end users.
3. **API Gateway / Reverse Proxy Cache:** Caches API responses at the edge (e.g., NGINX caching, Varnish).
4. **Application Cache (In-Memory):** In-memory data structures residing directly within the application process.
5. **Distributed Cache:** Shared, external in-memory key-value databases accessed by multiple application servers (e.g., Redis, Memcached).

---

## ▪ Read and Write Patterns

### 1. Cache-Aside (Lazy Loading)
The application queries the cache first. On a *cache hit*, it returns the data. On a *cache miss*, it queries the database, updates the cache with the retrieved data, and returns the response.
*   **Pros:** Only requested data is cached. Database failures are tolerated since the app fallback-queries the database directly.
*   **Cons:** Cache miss penalty on the first query. Potential stale data if database updates do not invalidate the cache.

### 2. Write-Through
The application writes data to the cache first, and the cache synchronously writes to the database before completing the request.
*   **Pros:** Data in the cache is never stale.
*   **Cons:** High write latency because of two sequential write operations.

### 3. Write-Behind (Write-Back)
The application writes data directly to the cache, which acknowledges the write immediately. The cache then asynchronously flushes updates to the database in the background.
*   **Pros:** Extremely high write throughput.
*   **Cons:** Risk of data loss if the cache node crashes before updates are flushed to the database.

---

## ▪ Eviction Policies

When the cache reaches memory limits, eviction policies determine which items to delete:

*   **LRU (Least Recently Used):** Evicts the item that has not been accessed for the longest time. Highly effective for general workloads.
*   **LFU (Least Frequently Used):** Evicts the item with the lowest request count.
*   **FIFO (First In, First Out):** Evicts the oldest item regardless of access frequency.

---

## ▪ Key Architectural Considerations

*   **Cache Penetration:** Requests targeting non-existent keys bypass the cache and hit the database.
    *   *Mitigation:* Use **Bloom Filters** to check if keys exist before querying, or cache empty/null values with a short TTL.
*   **Cache Avalanche:** Multiple keys expire simultaneously, causing a massive surge of concurrent queries to flood the database.
    *   *Mitigation:* Add random variation (*jitter*) to expiration times (TTL).
*   **Cache Stampede (Dogpiling):** A highly popular key expires, causing thousands of concurrent requests to attempt to compute and write the value back to the cache simultaneously.
    *   *Mitigation:* Use mutual exclusion locks (mutex) to allow only one thread to recalculate the value, or pre-generate values in the background before expiry.
