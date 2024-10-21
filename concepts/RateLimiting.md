# Rate Limiting

## What is Rate Limiting?

**Rate limiting** is a technique used to control the amount of traffic sent or received by a system within a specific time frame. It is typically used to prevent **overloading** servers or services, ensure **fair usage** of resources, and protect against **abuse** or **denial of service (DoS) attacks**.

Rate limiting is crucial for maintaining the stability and performance of your system, especially in high-traffic environments like public APIs, web applications, and microservices architectures.

---

## Why is Rate Limiting Important?

Without rate limiting, a system can become overwhelmed with too many requests in a short period, leading to performance degradation, increased response times, or even system crashes. Rate limiting ensures that no single user or client consumes too many resources, protecting the system from being overwhelmed and ensuring **fair usage** for all users.

---

## How Rate Limiting Works

Rate limiting is usually implemented by tracking the number of requests made by a client (identified by their IP address, API key, or user ID) within a **time window**. If the number of requests exceeds a specified limit, the system will **throttle** or **block** further requests until the time window resets.

For example:
- A rate limit of 100 requests per minute means that if a client sends more than 100 requests in a minute, the system will start rejecting requests or slowing down responses.

---

## Types of Rate Limiting Algorithms

There are several algorithms used to implement rate limiting. Each has different properties and use cases:

### 1. **Token Bucket**

- **Description**: The **token bucket** algorithm allows a certain number of tokens (representing requests) to be added to a "bucket" at regular intervals. Each request consumes one token. If the bucket is empty, the request is denied or throttled until more tokens are added.
  
- **Pros**:
  - Allows **bursts** of traffic while still maintaining an average request rate.
  
- **Cons**:
  - Managing tokens adds some complexity.

- **Use Case**: Ideal for services where short bursts of traffic are allowed, such as public APIs.

> **Analogy**: Imagine a bucket that fills up with tokens at a steady rate. You can take out tokens (send requests) as long as the bucket has tokens, but once it's empty, you must wait for it to refill.

### 2. **Leaky Bucket**

- **Description**: Similar to the token bucket, but the **leaky bucket** algorithm processes requests at a **fixed rate**. Excess requests are stored in a queue. If the queue is full, additional requests are discarded.
  
- **Pros**:
  - Ensures that traffic is processed at a steady, fixed rate, preventing sudden spikes.
  
- **Cons**:
  - Does not handle bursty traffic as well as the token bucket algorithm.

- **Use Case**: Suitable for services where a constant flow of requests is required, such as video streaming or content delivery networks (CDNs).

> **Analogy**: Picture a bucket with a small hole at the bottom. Water (requests) flows out at a fixed rate, and if too much water enters too quickly, the bucket overflows, causing some requests to be dropped.

### 3. **Fixed Window Counter**

- **Description**: The **fixed window counter** algorithm counts the number of requests made in a **fixed time window** (e.g., 100 requests per minute). If the limit is exceeded during the time window, all additional requests are blocked until the window resets.
  
- **Pros**:
  - Simple to implement and understand.
  
- **Cons**:
  - Can lead to **spikes** at the boundaries of time windows, where many requests are sent just before and after the window resets.

- **Use Case**: Commonly used for **API rate limiting** where each user or API key has a fixed number of requests per time window.

> **Analogy**: Imagine a gate that opens for exactly one minute. During that minute, you can send up to 100 requests. Once the minute is over, the gate closes, and the count resets.

### 4. **Sliding Window Log**

- **Description**: The **sliding window log** algorithm tracks each individual request with a timestamp. Requests are allowed if the number of requests within the **sliding window** (e.g., the last minute) doesn’t exceed the limit.
  
- **Pros**:
  - Provides more accurate rate limiting by avoiding spikes at time window boundaries.
  
- **Cons**:
  - Requires more storage to track timestamps of all requests, which may add overhead.

- **Use Case**: Useful when a smoother rate-limiting approach is required to avoid spikes, such as in critical services with high-traffic variability.

> **Analogy**: Think of it as a rolling time window that constantly moves forward. It checks how many requests have been made in the past 60 seconds, regardless of the exact time boundary.

### 5. **Sliding Window Counter**

- **Description**: Similar to the fixed window counter but divides the time window into smaller segments (e.g., seconds within a minute). Requests are limited based on the sum of requests from the current and previous segments to smooth out sudden bursts.
  
- **Pros**:
  - Reduces spikes at the boundaries of time windows.
  
- **Cons**:
  - Slightly more complex to implement than the fixed window counter.

- **Use Case**: Useful in systems where smoothing out traffic spikes is important, such as online services during peak hours.

---

## Implementing Rate Limiting in APIs

When rate limiting APIs, there are a few important design considerations:

### 1. **Client Identification**
   - Identify clients using **API keys**, **IP addresses**, or **user IDs** to ensure rate limits apply correctly. This ensures that each client gets a fair share of system resources.

### 2. **Response Handling**
   - When a client exceeds the rate limit, return a **429 Too Many Requests** HTTP status code. This informs the client that they need to wait before making more requests.

   > **Example**: 
   ```http
   HTTP/1.1 429 Too Many Requests
   Retry-After: 60
   ```

### 3. **Retry-After Header**
- The `Retry-After` header tells the client when they can send the next request, helping to manage retries more efficiently.

### 4. **Rate Limit Headers**
- Include headers in API responses to inform the client of their remaining request quota and when the rate limit resets.
> Example:
   ```
   X-RateLimit-Limit: 100
   X-RateLimit-Remaining: 10
   X-RateLimit-Reset: 1629033600
   ```

---

## Rate Limiting Strategies

### 1. **Global Rate Limiting**
- **Description**: The rate limit is applied to all requests, regardless of the client. This is often used in simple systems where every client has the same limit.

> **Example**: An API allows no more than 1,000 requests per second across all users.

### 2. **Per-User or Per-API Key Rate Limiting**
- **Description**: Each user or API key has its own rate limit. This ensures that heavy users or abusers don’t affect the experience of others.

> **Example**: A user is allowed to make up to 100 requests per minute with their API key. Other users have their own separate limits.

### 3. **Geographic Rate Limiting**
- **Description**: Rate limits are applied based on geographic location (e.g., country, region) to ensure fair usage across different areas and reduce the impact of distributed attacks.

> **Example**: API requests from the US are limited to 500 requests per second, while requests from Europe are limited to 300 requests per second.

### 4. **Tiered Rate Limiting**
- **Description**: Different rate limits are applied to different users based on their **subscription tier** or **role** (e.g., free vs. premium users).

> **Example**: Free users can make 50 requests per minute, while premium users can make 200 requests per minute.

---

## Real-World Examples of Rate Limiting

### 1. **Twitter API Rate Limits**
- Twitter applies per-user and per-application rate limits to ensure the platform remains stable and usable for all developers. For example, users can only retrieve up to 900 tweets every 15 minutes via the API.

### 2. **Cloudflare Rate Limiting**
- Cloudflare provides rate-limiting services to prevent DDoS (Distributed Denial of Service) attacks by limiting requests based on IP address. This protects web applications from being overwhelmed by excessive traffic.

### 3. **GitHub API Rate Limits**
- GitHub limits users to 5,000 API requests per hour for authenticated requests and 60 requests per hour for unauthenticated requests. If the limit is exceeded, users receive a 403 Forbidden status code.

---

## Important Terms

- **Throttle**: Temporarily delaying or limiting requests when the rate limit is exceeded.
- **429 Too Many Requests**: An HTTP status code indicating the client has sent too many requests within a given timeframe.
- **Retry-After Header**: An HTTP header that tells the client how long they should wait before making another request.
- **Burst Traffic**: A short period where the request rate temporarily exceeds the normal load.

---

## Conclusion

Rate limiting is a critical technique in system design to prevent abuse, ensure fair resource

distribution, and protect services from excessive traffic. Understanding the various rate limiting algorithms and when to apply them is key to building resilient, scalable systems that maintain performance and availability under load.

---

## Further Reading

- [How to Implement Rate Limiting](https://blog.cloudflare.com/rate-limiting-how-to/)
- [API Rate Limiting Best Practices](https://konghq.com/blog/api-rate-limiting-best-practices/)
- [GitHub API Rate Limits](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting)