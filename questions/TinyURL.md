# System Design: TinyURL

## Problem Statement

Design a scalable URL shortening service, like TinyURL or Bitly, that allows users to convert long URLs into short, unique URLs. When the short URL is accessed, it redirects to the original long URL.

### Requirements

#### Functional Requirements:
1. **Shorten URL**: The service should provide users with a short alias for a given long URL.
2. **Redirect to Original URL**: When a user visits the short URL, the service should redirect them to the original long URL.
3. **Custom Alias (Optional)**: Users may request a custom short alias instead of the system-generated one.
4. **Expiration (Optional)**: Short URLs can have an expiration date, after which they no longer redirect.

#### Non-Functional Requirements:
1. **Scalability**: The system should handle a large number of URL shortening and redirection requests.
2. **High Availability**: The service should be available 24/7 without downtime.
3. **Low Latency**: URL redirection should happen quickly with minimal delay.
4. **Analytics (Optional)**: Provide analytics for the number of times the short URL is accessed.

---

## Components of the System

The TinyURL system consists of the following components:

1. **API Gateway**: Exposes APIs for creating short URLs and redirects users when they visit short URLs.
2. **URL Shortening Service**: Handles the business logic for shortening URLs and expanding short URLs to their original forms.
3. **Database**: Stores the mappings between long URLs and their short URLs, along with other metadata (e.g., expiration dates).
4. **Cache (Optional)**: Used to store frequently accessed short-to-long URL mappings to reduce database load.
5. **Analytics Module (Optional)**: Tracks and stores access analytics for the shortened URLs.

---

## High-Level Design

1. **User Interface (UI)**: Users submit long URLs and receive shortened URLs. They can also enter short URLs in their browser, which are redirected to the long URLs.
2. **API Layer**: Exposes endpoints to shorten URLs (`POST /shorten`) and expand shortened URLs (`GET /{shortUrl}`).
3. **Service Layer**:
    - **URL Shortener**: Converts long URLs to short URLs using a unique identifier.
    - **Redirect Service**: Looks up the original URL for a given short URL and redirects the user.
4. **Storage Layer**:
    - **Primary Storage (Database)**: Stores the long-to-short URL mappings, custom aliases, and metadata.
    - **Cache (Optional)**: Reduces the load on the database by storing frequently accessed URL mappings.

---

## Detailed Design

### 1. **Short URL Generation**

#### Approach 1: Base62 Encoding
- Each URL is assigned a unique ID (auto-incrementing or generated via a hash).
- The unique ID is converted into a **Base62** string (consisting of uppercase letters, lowercase letters, and digits). Base62 encoding is used because it creates short, alphanumeric URLs.

For example:
- URL ID: 125 -> Base62: `cb`
- URL ID: 100000 -> Base62: `q0k`

This results in short, unique URLs that can be generated efficiently.

#### Approach 2: Hashing
- Another approach is to hash the long URL (using algorithms like MD5 or SHA256) and take a fixed number of characters from the hash to generate the short URL.
- However, hash collisions need to be handled, making this approach slightly more complex.

#### Approach 3: Custom Alias
- Allow users to provide a custom alias. For example, they can create `tinyurl.com/mycustomalias`.

### 2. **Database Schema**

| Field          | Type          | Description                                 |
|----------------|---------------|---------------------------------------------|
| `short_url`    | String (PK)   | The shortened URL (or alias).               |
| `long_url`     | Text          | The original long URL.                      |
| `created_at`   | Timestamp     | Timestamp when the URL was shortened.       |
| `expiration`   | Timestamp     | (Optional) Expiration date for the short URL.|
| `click_count`  | Integer       | (Optional) Number of times the short URL was accessed.|
| `user_id`      | String        | (Optional) ID of the user who created the URL.|

### 3. **Redirection**

When a user accesses a short URL, the system performs a **GET** request on the short URL, looks it up in the database, retrieves the original URL, and redirects the user to it.

### 4. **Expiration (Optional)**

Add an optional expiration date to the short URL. After the expiration date, the short URL will no longer redirect, and users will receive a 404 error.

### 5. **Analytics (Optional)**

- Track how many times the short URL has been accessed.
- Provide statistics on access (e.g., daily, weekly).
- Record the IP address, geographic location, and timestamp of access.

### 6. **Caching (Optional)**

To reduce load on the database, frequently accessed URL mappings can be stored in a **distributed cache** (like **Redis** or **Memcached**). Cache the mappings from short URLs to long URLs to speed up redirections.

---

## Key Challenges

### 1. **Unique Short URL Generation**
- We need to ensure that the generated short URL is unique. This can be handled using an auto-incrementing ID (with Base62 encoding) or by ensuring that hash collisions are resolved if using a hash-based approach.

### 2. **Database Bottlenecks**
- As the service scales, the database might become a bottleneck. To mitigate this:
    - Use database **sharding** based on URL ID or hash.
    - Implement **read replicas** to handle read-heavy workloads (since redirection requests are much more frequent than URL creation).

### 3. **Scaling the Service**
- Horizontal scaling is required to handle a large number of read and write requests:
    - **APIs** and **databases** should be replicated across multiple servers.
    - Implement **load balancing** to distribute traffic evenly.
    - Use **CDNs** to serve static content and reduce the load on application servers.

### 4. **Cache Invalidation**
- Implement a proper cache invalidation strategy to ensure that cached mappings are updated when URLs expire or are modified.

---

## Data Flow

### 1. **Shortening a URL** (`POST /shorten`)
- The user submits a long URL via the UI or API.
- The system checks if the long URL already exists in the database (to avoid creating duplicate short URLs).
- If the long URL is new, the system generates a unique short URL using Base62 encoding or hashing.
- The long URL, short URL, and other metadata (e.g., creation time, expiration) are stored in the database.
- The system returns the shortened URL to the user.

### 2. **Redirecting a Short URL** (`GET /{shortUrl}`)
- A user visits the shortened URL.
- The system checks the cache for the corresponding long URL:
    - If found, it redirects the user to the original long URL.
    - If not found, the system queries the database for the long URL.
    - The long URL is returned, and the user is redirected.
    - (Optional) Cache the short-to-long URL mapping for future requests.
- The system updates the click count and other analytics if required.

---

## Scaling the System

### 1. **Database Sharding**
To handle the large number of URL mappings, shard the database by URL ID or hash of the URL. This ensures that data is distributed evenly across multiple nodes, reducing the load on any single database server.

### 2. **Caching**
Use a distributed caching layer (e.g., Redis) to store frequently accessed short-to-long URL mappings. This reduces the load on the database and speeds up redirections.

### 3. **Load Balancing**
Use load balancers to distribute API requests across multiple application servers. This ensures that the system can handle a large number of requests concurrently.

### 4. **Content Delivery Network (CDN)**
Serve static assets (such as the web interface) via a CDN to reduce the load on the servers and improve performance for users in different geographic regions.

---

## Security Considerations

### 1. **Short URL Abuse**
Short URLs can be used to mask malicious websites. To prevent this, the system can:
- Implement a URL blacklist to block known malicious URLs.
- Scan submitted URLs using third-party security APIs (e.g., Google Safe Browsing API) before creating short URLs.

### 2. **Rate Limiting**
To prevent abuse (e.g., spamming the service to create a large number of short URLs), implement rate limiting on the API and user accounts.

---

## Conclusion

Designing a URL shortening service like TinyURL requires careful consideration of scalability, unique short URL generation, and efficient redirection. By implementing features like caching, database sharding, and load balancing, the system can handle a large volume of requests while maintaining low latency and high availability. Optional features like analytics and expiration further enhance the service for advanced use cases.

---

## Further Reading

- [TinyURL Architecture](https://example-link.com/tinyurl-architecture)
- [How to Build a URL Shortener](https://example-link.com/build-url-shortener)
- [Caching Best Practices](https://example-link.com/caching-best-practices)
