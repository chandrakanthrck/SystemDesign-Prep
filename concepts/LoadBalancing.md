# Load Balancing

## What is Load Balancing?

Load balancing is the process of **distributing network traffic** or application requests across multiple servers or resources to ensure no single server is overwhelmed. By spreading the workload, load balancing helps optimize resource use, maximize throughput, minimize response times, and ensure high availability.

### Why is Load Balancing Important?

Without load balancing, one server may become a bottleneck, causing delays or even system crashes, especially during traffic spikes. Load balancers distribute the load so that no single server takes on too much, improving the system's **fault tolerance** and **scalability**.

---

## How Load Balancing Works

A **load balancer** acts as a middleman between the client and the servers. When a request comes in, the load balancer forwards the request to one of the available servers based on certain algorithms or rules. After the server processes the request, the response is sent back to the client, often through the load balancer.

---

## Types of Load Balancers

### 1. **Layer 4 Load Balancer** (Transport Layer)

- Operates at the **transport layer** (TCP/UDP).
- Routes traffic based on **IP addresses** and **ports**.
- Doesn’t look at the actual content of the request (i.e., application-level details).

> **Example**: NGINX can function as a Layer 4 load balancer, distributing requests based on IP addresses.

### 2. **Layer 7 Load Balancer** (Application Layer)

- Operates at the **application layer** (HTTP/HTTPS).
- Routes traffic based on more complex rules such as **URL paths**, **HTTP headers**, or **cookies**.
- Provides greater flexibility, as it can route requests based on the actual content.

> **Example**: AWS Application Load Balancer (ALB) can route traffic based on specific URLs or headers (e.g., directing `/images` requests to one set of servers and `/videos` to another).

---

## Load Balancing Algorithms

Load balancers use different algorithms to decide how to distribute incoming traffic. Some common algorithms include:

### 1. **Round Robin**
- Requests are distributed to each server in turn, one by one.
- Simple and easy to implement but does not account for server load.

> **Analogy**: Imagine passing out papers in a classroom. You hand one paper to each student in order, going around the room until everyone has one.

### 2. **Least Connections**
- Directs traffic to the server with the fewest active connections.
- More effective in scenarios where some requests take longer to process than others.

> **Analogy**: If you have multiple checkout lines at a grocery store, this method is like directing a new shopper to the line with the fewest people waiting.

### 3. **IP Hash**
- The client’s IP address is used to determine which server will handle the request. The same client IP is routed to the same server each time.
- Good for maintaining **session persistence** (sticky sessions), where each client’s requests are routed to the same server.

> **Use Case**: Useful in scenarios where session persistence is critical, such as online banking.

### 4. **Weighted Round Robin**
- Similar to round robin, but servers are assigned different weights based on their capacity. Servers with higher weights get more requests.

> **Use Case**: If one server is more powerful (has more CPU or RAM), it can handle more requests, so it's assigned a higher weight.

### 5. **Least Response Time**
- Routes traffic to the server that has the **least response time** or **fastest connection**.

> **Use Case**: Effective for applications where low latency is crucial.

---

## Types of Load Balancing Architecture

### 1. **DNS Load Balancing**
- DNS servers return multiple IP addresses for a single domain name, distributing traffic across multiple servers.
- **Pros**: Simple to implement.
- **Cons**: No insight into server health (e.g., if one server is down, the DNS server might still send traffic to it).

> **Example**: When you type `example.com`, DNS load balancing can return different IPs depending on the user's location.

### 2. **Client-Side Load Balancing**
- The load balancing logic is built into the client (e.g., app or browser). The client makes decisions on which server to send the request to based on information from a service registry.
- **Tools**: Netflix Ribbon.

> **Use Case**: Microservices architecture where the client knows the available service instances and routes requests accordingly.

### 3. **Server-Side Load Balancing**
- The load balancer sits between the client and the servers, distributing traffic to backend servers.
- Most common type of load balancing.
- **Tools**: NGINX, HAProxy, AWS ELB, F5.

---

## Key Considerations for Load Balancing

### 1. **Health Checks**
- Load balancers should continuously check the health of backend servers to ensure that requests are only sent to servers that are functioning properly.
- **Active Health Checks**: The load balancer actively pings or sends requests to the server to check its health.
- **Passive Health Checks**: The load balancer detects issues based on failed or slow requests.

> **Example**: If one server crashes, the load balancer should automatically route requests to other available servers.

### 2. **Session Persistence (Sticky Sessions)**
- In some cases, you want all requests from the same client to go to the same server. This is especially important when using stateful sessions.
- Implemented using cookies, IP hashing, or session IDs.

> **Example**: An e-commerce site might need session persistence to ensure a user’s shopping cart data stays with the same server during a session.

### 3. **SSL Termination**
- Offloading SSL encryption/decryption to the load balancer can reduce the computational load on backend servers.
- Some load balancers can handle SSL/TLS traffic, decrypt it, and forward unencrypted traffic to backend servers.

> **Example**: AWS Application Load Balancer (ALB) can terminate SSL connections and forward plain HTTP traffic to backend servers.

### 4. **Global Load Balancing**
- Used for distributing traffic across multiple **geographic regions**. This improves performance by routing traffic to servers closer to the client.
- **Tools**: AWS Route 53, Cloudflare.

> **Use Case**: If your application has users across the globe, global load balancing ensures that users in Asia are routed to servers in Asia for lower latency.

---

## Tools and Technologies for Load Balancing

1. **NGINX**: Popular open-source web server that can act as a reverse proxy and load balancer.
2. **HAProxy**: High-performance open-source load balancer.
3. **AWS Elastic Load Balancer (ELB)**: Managed load balancer provided by AWS for both Layer 4 and Layer 7 traffic.
4. **F5 Networks**: Hardware-based load balancer for high-performance environments.
5. **Kubernetes Ingress**: Load balancing for Kubernetes clusters.

---

## Real-World Example of Load Balancing

### Example: Load Balancing a Social Media Application

- **Scenario**: A social media platform with millions of users experiences peak traffic during certain times of the day (e.g., mornings and evenings). The platform needs to handle requests to view profiles, post updates, and load photos.

- **Solution**:
    - Use **Layer 7 Load Balancing** to direct photo-related requests to servers optimized for large media files.
    - Implement **Round Robin** to distribute user profile requests evenly across multiple servers.
    - **Global Load Balancing** routes users to the nearest data center to reduce latency.
    - Use **Health Checks** to ensure that only healthy servers receive traffic. If a server goes down, requests are automatically redirected.

---

## Important Terms

- **Sticky Sessions**: Ensures requests from the same client are always routed to the same server.
- **SSL Termination**: Offloading SSL decryption from the backend servers to the load balancer.
- **Health Checks**: Regular checks by the load balancer to ensure backend servers are operational.
- **Reverse Proxy**: A server that routes client requests to other servers, often used in conjunction with load balancers.

## Conclusion

Load balancing is a critical aspect of system design, ensuring that applications remain highly available and responsive under varying traffic conditions. By understanding different load balancing strategies, algorithms, and architectures, you can design systems that handle traffic efficiently and provide a seamless user experience.

---

## Further Reading

- [Load Balancing Algorithms](https://en.wikipedia.org/wiki/Load_balancing_(computing)#Algorithms)
- [NGINX Load Balancing](https://www.nginx.com/resources/glossary/load-balancing/)
- [AWS Elastic Load Balancer](https://aws.amazon.com/
