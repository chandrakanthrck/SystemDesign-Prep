# Caching

## What is Caching?

Caching is the process of storing copies of frequently accessed data in a **temporary storage layer** to serve future requests faster. By reducing the need to repeatedly fetch data from the original source (e.g., database or external API), caching improves system **performance**, reduces **latency**, and lessens the load on back-end resources.

### Why is Caching Important?

Without caching, every request for the same data would result in repeated trips to the database or external services, causing increased load on those systems and slowing down the overall response time for users. Caching helps to **minimize the need for expensive computations** and **expensive data retrieval operations** by reusing the results.

---

## How Caching Works

When a client requests data, the system first checks whether the data is available in the cache:
1. **Cache Hit**: If the data is found in the cache, it is served directly from the cache, which is much faster than fetching from the original source.
2. **Cache Miss**: If the data is not found in the cache, the system retrieves the data from the original source (e.g., database), stores a copy of the data in the cache, and returns the result to the client.

---

## Types of Caching

### 1. **Client-Side Caching**
- **Description**: The cache resides on the client's machine (e.g., browser cache).
- **Use Case**: Caching static resources like HTML, CSS, JavaScript files, and images on the client’s browser to speed up page load times.

> **Analogy**: Think of a browser cache like a "saved memory" of a page you visited recently. If you go back to that page, your browser doesn’t need to reload everything from the server—it just pulls it from the cache on your device.

### 2. **Server-Side Caching**
- **Description**: The cache is maintained on the server-side or in a shared layer between servers.
- **Use Case**: Storing frequently accessed data in a shared server cache to reduce the number of database or external service requests.
- **Types**:
    - **In-memory caching**: Data is stored in memory (RAM) for fast access (e.g., Redis, Memcached).
    - **Disk-based caching**: Data is stored on disk, typically used for larger data sets that are less frequently accessed.

> **Analogy**: Server-side caching is like having a "speed lane" on a busy road for regular commuters, allowing them to skip the traffic and reach their destination faster.

### 3. **Database Query Caching**
- **Description**: Results of frequently run database queries are cached to avoid repeating the same queries.
- **Use Case**: Caching the results of expensive SQL queries that are often repeated, to reduce the load on the database.

---

## Caching Strategies

### 1. **Cache-Aside (Lazy Loading)**
- **Description**: The cache only loads data when it’s requested (on a cache miss). If the cache doesn’t have the data, the system fetches it from the database, stores it in the cache, and then serves it.
- **Use Case**: General-purpose caching where data can expire or change frequently.

> **Example**: If the data is requested and not found in the cache, the application queries the database, caches the result, and returns the data to the user.

### 2. **Write-Through Cache**
- **Description**: Data is written to the cache and the database simultaneously. This ensures the cache always contains fresh data.
- **Use Case**: When you want strong consistency between the cache and the database.

> **Example**: When a user updates their profile, both the cache and the database are updated at the same time to reflect the changes.

### 3. **Write-Back (Write-Behind) Cache**
- **Description**: Data is written to the cache first, and the write to the database is done asynchronously later. This can improve write performance but risks data loss if the cache fails before writing to the database.
- **Use Case**: Systems where data write performance is critical, and eventual consistency is acceptable.

> **Example**: A system where user interactions (like "likes" on a post) are cached first and then persisted to the database in batches later.

### 4. **Read-Through Cache**
- **Description**: The application never directly queries the database. Instead, it interacts only with the cache, which is responsible for fetching the data from the database if necessary.
- **Use Case**: Simplifies application logic by centralizing all data requests to the cache layer.

> **Example**: When a web server always asks the cache for data, and the cache retrieves the data from the database if needed.

---

## Cache Invalidation Strategies

**Cache invalidation** is important to ensure that the cache contains fresh and up-to-date data. Some common invalidation strategies include:

### 1. **Time-to-Live (TTL)**
- **Description**: Data is automatically removed from the cache after a predefined period.
- **Use Case**: Ideal when data is expected to change periodically, but freshness is not critical.

> **Example**: Caching stock prices with a TTL of 5 minutes. After 5 minutes, the cache expires, and new data is fetched from the database.

### 2. **Manual Invalidation**
- **Description**: Developers manually invalidate or update the cache when data changes.
- **Use Case**: When caching dynamic content and changes to the data are unpredictable.

> **Example**: When a user updates their profile, the cache is explicitly cleared to reflect the changes.

### 3. **Event-Driven Invalidation**
- **Description**: The cache is invalidated based on certain events or triggers in the system.
- **Use Case**: When cache invalidation needs to happen in response to specific updates, such as database changes.

> **Example**: When new products are added to an e-commerce site, the cache for the product listing page is invalidated automatically.

---

## Cache Eviction Policies

Cache storage is often limited, so when the cache is full, an **eviction policy** determines which data to remove to make space for new entries. Some common eviction policies include:

### 1. **Least Recently Used (LRU)**
- **Description**: Removes the data that has not been used for the longest time.
- **Use Case**: General-purpose caching where recently accessed data is more likely to be used again.

> **Analogy**: Imagine a fridge where you remove the oldest food item to make space for new groceries.

### 2. **Least Frequently Used (LFU)**
- **Description**: Removes the data that has been accessed the least number of times.
- **Use Case**: Ideal when certain data is consistently requested more often than other data.

> **Analogy**: Think of a bookshelf where you remove books that have been read the fewest times.

### 3. **First-In-First-Out (FIFO)**
- **Description**: Evicts the oldest data based on when it was added to the cache.
- **Use Case**: When it’s important to retain newer data, regardless of how frequently it is accessed.

> **Analogy**: Just like a queue, the oldest item gets evicted first.

---

## Tools for Caching

### 1. **Redis**
- In-memory data structure store used for caching, session management, and more.
- Supports data persistence, allowing cached data to survive a restart.

> **Example**: A system caching frequently accessed user data (e.g., profiles) in Redis to reduce database load.

### 2. **Memcached**
- In-memory key-value store for small chunks of data, such as strings and objects, from results of database calls, API calls, or page rendering.
- No persistence, purely for in-memory caching.

> **Example**: Caching API responses in Memcached for fast retrieval during high traffic.

### 3. **Varnish**
- HTTP reverse proxy that caches the HTTP response from your web server.
- Often used to cache static content such as images, CSS, and JavaScript.

> **Example**: Caching the entire homepage of a news website in Varnish to handle sudden traffic spikes.

---

## Caching Considerations in System Design

### 1. **Data Freshness**
- How fresh should the cached data be? Are you willing to serve slightly stale data to users? If not, you may need aggressive cache invalidation.

### 2. **Cache Size**
- How much data do you plan to cache? Too small a cache means frequent evictions, and too large a cache might require significant memory or disk space.

### 3. **Consistency**
- Do you require strong consistency between the cache and the underlying data source, or is eventual consistency acceptable?

### 4. **Latency**
- Is the cache close enough to the users or the requesting service? Reducing the **round-trip time (RTT)** between the cache and client can improve the performance benefits of caching.

### 5. **Cache Warming**
- Preloading the cache with frequently accessed data to ensure a cache hit for common requests immediately after system startup.

---

## Real-World Example of Caching

### Example: Caching in a Social Media Feed

- **Scenario**: A social media platform generates a personalized feed for each user. Fetching this feed from the database for millions of users can be expensive and slow.

- **Solution**: Implement a **cache-aside strategy** where each user's feed is cached after the first request. For subsequent requests, the cached feed is served directly, reducing database load and speeding up the response time.

- **Eviction Policy**: Use **LRU (Least Recently Used)** to ensure that the most active users’ feeds remain in the cache, while inactive users' data is evicted over time.

---

## Important Terms

- **Cache Hit**: When the requested data is found in the cache.
- **Cache Miss**: When the requested data is not found in the cache and needs to be fetched from the source.
- **TTL (Time-to-Live)**: How long data remains in the cache before it is invalidated.
- **Eviction Policy**: The strategy for removing data from the cache when it is full.

## Conclusion

Caching is a crucial technique in system design for improving performance and scalability. By understanding different caching strategies, invalidation methods, and eviction policies, you can significantly reduce system load and improve response times in large-scale applications.

---

## Further Reading

- [Redis Documentation](https://redis.io/documentation)
- [Memcached Overview](https://memcached.org/about)
- [Varnish Cache](https://varnish-cache.org/)

