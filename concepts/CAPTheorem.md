# CAP Theorem

## What is CAP Theorem?

The **CAP Theorem**, also known as **Brewer's Theorem**, is a fundamental principle in distributed systems that states that a distributed database can only provide **two** out of the following three guarantees simultaneously:

1. **Consistency** (C)
2. **Availability** (A)
3. **Partition Tolerance** (P)

Due to the inherent challenges of distributed systems, you cannot achieve all three guarantees at the same time. This forces system architects to make trade-offs depending on the system’s requirements.

---

## Breakdown of CAP Theorem Components

### 1. **Consistency (C)**

- **Definition**: Every read receives the most recent write or an error. In other words, **all nodes see the same data at the same time**.

- **Analogy**: Imagine a library where all copies of a book are always kept updated. If a new chapter is added to one copy, every other copy immediately reflects this change. If a reader opens any book, they will see the latest version, but if the system can't update a copy in real-time, it will not show anything until consistency is restored.

- **Real-World Example**: A traditional **SQL database** (such as PostgreSQL or MySQL) where all reads and writes are done on a single server or replicated immediately to ensure consistency.

### 2. **Availability (A)**

- **Definition**: Every request receives a response, even if it’s not the most recent data. In an available system, nodes do not wait for each other and will return the best answer they can provide (potentially stale data).

- **Analogy**: Imagine the same library, but this time the priority is on getting a book into every reader's hands. If the new chapter hasn’t arrived yet, a reader might get an older version of the book, but they always get a response—either the latest available version or a previous one.

- **Real-World Example**: **DNS (Domain Name System)** ensures availability by serving potentially stale information when a DNS server is temporarily unavailable. Users still get responses even though the latest data may not be present.

### 3. **Partition Tolerance (P)**

- **Definition**: The system continues to function despite network partitions. A network partition occurs when there is a failure in communication between nodes, such as network latency or failures that prevent nodes from syncing.

- **Analogy**: Picture the library with a large power outage in half of the building, but the other half continues to serve readers. The library system will still provide books to users in the working part of the building, but the copies in the downed section may not reflect the latest updates.

- **Real-World Example**: **NoSQL databases** like **Cassandra** or **DynamoDB** are designed to be partition-tolerant and continue operating even when parts of the network are down or unreachable.

---

## Understanding the Trade-Offs: CAP Triangle

According to the CAP Theorem, you can only pick **two** out of the three guarantees at the same time:

### 1. **CP Systems (Consistency + Partition Tolerance)**
- **Description**: CP systems prioritize data consistency across all nodes while ensuring the system continues to function during network partitions. However, this comes at the cost of **availability**—the system might become unavailable to serve some requests if the system cannot guarantee consistency during a partition.

- **When to Use**: When **strong consistency** is crucial, such as in financial systems where the most recent data is always needed (e.g., a bank account balance).

> **Example**: **HBase** is a CP system, providing strong consistency even during partitions, but may sacrifice availability.

### 2. **AP Systems (Availability + Partition Tolerance)**
- **Description**: AP systems ensure that the system remains available and functional even during network partitions, but at the cost of **consistency**. This means some nodes might serve stale or outdated data, but users always receive a response.

- **When to Use**: When **availability** is more important than strict consistency, such as in social media platforms where delivering messages quickly is prioritized over strict ordering.

> **Example**: **Cassandra** and **DynamoDB** are AP systems that ensure availability and partition tolerance but allow eventual consistency.

### 3. **CA Systems (Consistency + Availability)**
- **Description**: CA systems ensure that all requests receive a consistent response and that the system is highly available as long as there are no network partitions. However, if a partition occurs, the system might fail to function.

- **When to Use**: CA systems are more theoretical because they cannot handle network partitions effectively. In practice, distributed systems always need to account for partitions, meaning that CA systems tend to degrade into CP or AP systems when partitions occur.

> **Example**: A **single-node SQL database** (e.g., MySQL running on a single server) provides consistency and availability but will fail if the network between the application and the database is partitioned.

---

## Key Considerations for CAP Theorem in System Design

### 1. **Eventual Consistency**
- In an **AP system**, eventual consistency means that after some time, all replicas of the data will converge to the same state, but there may be temporary inconsistencies.
- **Use Case**: Systems like social networks where it’s acceptable for users to see slightly outdated data, as long as the data eventually becomes consistent.

> **Example**: In a **shopping cart system**, it's acceptable for a user to see a slightly outdated cart as long as the system eventually synchronizes all items correctly.

### 2. **Strong Consistency vs. Weak Consistency**
- **Strong Consistency**: Guarantees that any read always returns the most recent write. Typically used in CP systems.
- **Weak/Eventually Consistency**: Data may be inconsistent for a period but will eventually become consistent across nodes. Used in AP systems.

> **Example**: Strong consistency is critical in **banking systems**, where all account updates must be reflected instantly. In contrast, weak consistency is sufficient for **cached web content**.

### 3. **Consistency Levels**
- Some systems offer tunable consistency, where you can choose how much consistency or availability you want.
- **Tunable Consistency**: You can configure whether the system prioritizes consistency (e.g., waiting for a majority of nodes to agree) or availability (returning the fastest result from any node).

> **Example**: **Cassandra** allows you to tune the consistency level for each query, providing flexibility based on your needs.

---

## Real-World Examples of CAP Trade-offs

1. **MongoDB**:
    - MongoDB provides eventual consistency in replica sets and can be configured to prioritize availability or consistency depending on the system design.
    - It is typically designed to be an **AP system** (Availability + Partition Tolerance), sacrificing strong consistency during partitions for the sake of availability.

2. **Google’s Spanner**:
    - Spanner is a globally distributed database that achieves **CP** (Consistency + Partition Tolerance) with the help of Google's infrastructure and specialized hardware (TrueTime API) to provide global consistency across distributed nodes.

3. **Amazon DynamoDB**:
    - DynamoDB is an example of an **AP** system that ensures high availability and partition tolerance. It sacrifices immediate consistency in favor of availability and uses eventual consistency to reconcile differences between nodes.

---

## CAP Theorem and the Real World

In practice, every distributed system will experience network partitions, so it’s crucial to choose between prioritizing **consistency** or **availability**. The choice depends on the application's requirements:

- **Consistency-critical applications** (e.g., banking, e-commerce transactions) need **CP** systems to ensure data accuracy.
- **Availability-critical applications** (e.g., social networks, video streaming) benefit from **AP** systems to remain responsive even during network disruptions.

---

## Important Terms

- **Consistency**: Ensures all nodes see the same data at the same time.
- **Availability**: Guarantees that every request receives a response, even if the data is not up-to-date.
- **Partition Tolerance**: Ensures the system continues to operate despite network partitions or failures.
- **Eventual Consistency**: A form of consistency where all replicas eventually converge to the same state, but may serve outdated data for a period.
- **Strong Consistency**: Guarantees that all reads reflect the most recent writes.

---

## Conclusion

The CAP Theorem is a core principle in distributed system design, highlighting the trade-offs between consistency, availability, and partition tolerance. While you can’t achieve all three guarantees simultaneously, understanding the use cases for each combination (CP, AP, CA) helps in designing systems that meet the specific requirements of availability and consistency in real-world applications.

---

## Further Reading

- [CAP Theorem - Wikipedia](https://en.wikipedia.org/wiki/CAP_theorem)
- [Consistency Models in Distributed Systems](https://www.cs.cornell.edu/courses/cs5412/2019sp/slides/06distributedconsistency.pdf)
- [Brewer’s CAP Theorem](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)
