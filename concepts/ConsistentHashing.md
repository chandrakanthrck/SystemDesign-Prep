# Consistent Hashing

## What is Consistent Hashing?

**Consistent hashing** is a key distribution technique used to distribute requests (or keys) across multiple servers (or nodes) in a distributed system. It ensures that when servers are added or removed, only a minimal number of keys are reassigned, reducing disruption and rebalancing overhead.

Unlike traditional hashing techniques, where adding or removing a server causes significant reshuffling of data, consistent hashing helps maintain **balance** and **efficiency** when scaling a distributed system.

---

## Why is Consistent Hashing Important?

In distributed systems, especially those involving **distributed databases**, **caches**, or **load balancing**, it's crucial to evenly distribute data or traffic across multiple servers. However, when servers are added or removed (due to scaling, failures, or upgrades), it can cause **rebalancing**, where a large number of data items or requests need to be reassigned.

Consistent hashing minimizes this rebalancing, ensuring that only a few keys need to be remapped, which leads to:

- **Improved scalability**: Easier to add or remove nodes without major disruption.
- **Fault tolerance**: System can gracefully handle node failures.
- **Load distribution**: Balances traffic or data across all nodes.

---

## How Does Consistent Hashing Work?

1. **Hashing Servers**: Each server (or node) is assigned a position on a **hash ring** using a hash function (e.g., SHA-256 or MD5). The hash ring represents a circular space (0 to maximum hash value).

2. **Hashing Keys**: Each key (or request) is also hashed and placed on the same hash ring.

3. **Mapping Keys to Servers**: A key is mapped to the first server encountered **clockwise** on the ring. This server is responsible for handling the key.

4. **Adding/Removing Servers**:
    - When a server is added, only the keys between the new server and its preceding server are remapped.
    - When a server is removed, its keys are reassigned to the next server in the ring.

   This minimizes the number of keys that need to be reassigned when servers are added or removed.

---

## Example of Consistent Hashing

### Step-by-Step Example:

- Imagine a system with 4 servers: S1, S2, S3, and S4.
- These servers are hashed and placed on a circular hash ring.
- Similarly, data items or requests (K1, K2, K3, etc.) are hashed and placed on the same ring.

### Before Adding a New Server:

1. **Mapping Keys**: Each key is assigned to the first server it encounters moving clockwise on the ring.
    - K1 → S2
    - K2 → S3
    - K3 → S1

### After Adding a New Server (S5):

1. A new server, S5, is added and hashed onto the ring.
2. Only the keys between S5 and its preceding server (S4) are reassigned to S5.
    - K3, previously handled by S1, might now be handled by S5, but K1 and K2 remain on their respective servers.

---

## Virtual Nodes in Consistent Hashing

One challenge with consistent hashing is that servers may be unevenly distributed on the ring, leading to imbalanced traffic or data distribution.

### Solution: **Virtual Nodes** (or "VNodes")

- **Virtual Nodes**: Each physical server is assigned multiple **virtual nodes**, and these virtual nodes are hashed separately on the ring.
- **Why Use Virtual Nodes?**: They ensure better distribution of keys across servers by increasing the number of positions on the ring that each server occupies.

### Benefits of Virtual Nodes:
- **Better Load Balancing**: More uniform distribution of keys across all servers.
- **Easier Scaling**: When new servers are added, fewer keys need to be reassigned, and load distribution is smoother.

> **Example**: Instead of hashing just S1, S2, S3, and S4, we hash S1 as S1a, S1b, S1c, and so on. This creates more entry points on the hash ring, leading to better distribution.

---

## Advantages of Consistent Hashing

1. **Minimized Rebalancing**: Only a small portion of the data needs to be redistributed when servers are added or removed.

2. **Scalability**: It’s easier to scale the system horizontally by adding new nodes, without significant disruptions to the existing key distribution.

3. **Fault Tolerance**: When a node fails, only its associated data is reassigned to nearby nodes, ensuring the system continues to function without major reconfiguration.

4. **Even Load Distribution**: By using virtual nodes, consistent hashing ensures even distribution of data or traffic across all nodes.

---

## Trade-Offs in Consistent Hashing

1. **Complexity**: Implementing consistent hashing and managing virtual nodes adds complexity to system design.

2. **Hotspots**: If a server has too few virtual nodes or the hash function isn’t balanced well, certain servers may become hotspots, receiving too much traffic.

3. **Latency**: With consistent hashing, some requests may need to be forwarded to distant servers, potentially increasing the latency in large distributed systems.

---

## Use Cases for Consistent Hashing

### 1. **Distributed Caching**
- Consistent hashing is often used in systems like **Memcached** or **Redis** to distribute cached data across multiple servers. When new servers are added or removed, only a small subset of cached items needs to be redistributed.

### 2. **Distributed Databases**
- Distributed databases like **Cassandra** and **DynamoDB** use consistent hashing to distribute data across nodes in a cluster. This ensures that data is evenly distributed and minimizes disruption during scaling or node failures.

### 3. **Load Balancers**
- Load balancers can use consistent hashing to route requests to the same server consistently, improving cache hit rates and ensuring that session-specific data is directed to the correct server.

---

## Analogy for Beginners

Imagine a **round table** with several people seated around it (these people represent servers). Each person has a bowl in front of them. You also have a pile of candy (representing data). To distribute the candy:

1. You start from your current position (representing a key).
2. You hand the candy to the next person clockwise, and they store it in their bowl.
3. If a new person (server) joins the table, only the candy between them and the person just before them is redistributed to their bowl.

This ensures that most of the candy remains in place, even if someone joins or leaves.

---

## Real-World Examples of Consistent Hashing

### 1. **Amazon DynamoDB**:
- DynamoDB uses consistent hashing to distribute data across multiple servers, ensuring scalability and fault tolerance. When a server is added or removed, only a small portion of the data is redistributed, improving efficiency.

### 2. **Memcached**:
- Memcached is a distributed memory object caching system that uses consistent hashing to manage cache items across multiple cache servers. This ensures that cached data is evenly distributed, and when cache servers are added or removed, only a portion of the cached data needs to be reassigned.

### 3. **Content Delivery Networks (CDNs)**:
- CDNs use consistent hashing to route user requests to the closest edge server. When a new edge server is added, only requests from a portion of users are rerouted to the new server, minimizing disruption.

---

## Best Practices for Consistent Hashing

1. **Use Virtual Nodes**: Always use virtual nodes to ensure a more balanced distribution of keys across all nodes. This helps to avoid uneven traffic or data loads.

2. **Monitor Key Distribution**: Regularly monitor the key distribution across nodes to ensure no server becomes a hotspot. Adjust the number of virtual nodes if necessary.

3. **Optimize for Latency**: In systems where minimizing latency is critical, ensure that the hash function takes the proximity of servers into account to avoid forwarding requests to distant servers.

4. **Test with Scaling**: Consistent hashing should be thoroughly tested with scaling events (adding or removing nodes) to ensure minimal key redistribution and system disruption.

---

## Important Terms

- **Hash Ring**: A circular data structure used to map both servers and keys in consistent hashing.
- **Virtual Nodes (VNodes)**: Multiple logical representations of a single physical server on the hash ring to improve distribution.
- **Rebalancing**: The process of redistributing keys across nodes when the system is scaled up or down.
- **Hotspot**: A server that receives disproportionate traffic or data, often due to poor hash function balance.

---

## Conclusion

Consistent hashing is a powerful technique used to manage data distribution in scalable, distributed systems. By minimizing data redistribution during scaling events and ensuring even load distribution through virtual nodes, consistent hashing enables distributed systems to grow efficiently while maintaining performance and fault tolerance.

---

## Further Reading

- [Introduction to Consistent Hashing and Scaling Distributed Systems](https://medium.com/system-design-blog/consistent-hashing-a-guide-to-scalable-system-design-47b6e1f710fa)
- [Amazon DynamoDB Architecture](https://aws.amazon.com/dynamodb/)
- [Consistent Hashing in Cassandra](https://cassandra.apache.org/doc/latest/architecture/dynamo.html)
