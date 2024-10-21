# Database Sharding

## What is Database Sharding?

**Sharding** is a database architecture pattern where a large dataset is split into smaller, more manageable pieces called **shards**, which are distributed across multiple databases or servers. Each shard contains a subset of the total data and is treated as a separate, independent database.

### Why is Sharding Important?

As the amount of data grows, a single database can become a bottleneck, affecting performance and limiting scalability. Sharding helps to distribute the workload across multiple databases (or servers), ensuring that no single instance is overloaded, thereby improving performance and scalability.

---

## How Sharding Works

In a sharded system, each shard contains a portion of the data. The data is typically divided based on a **sharding key**, such as user ID or geographic region. The application then determines which shard to query based on the value of this key.

For example:
- If you have a database of customer data and you shard it by **Customer ID**, one shard might store customer records with IDs from 1 to 1,000, while another shard stores records with IDs from 1,001 to 2,000.

---

## Types of Sharding

### 1. **Horizontal Sharding**

In horizontal sharding, rows of a database table are distributed across multiple shards. Each shard contains a unique subset of the rows, but all shards have the same table schema.

- **Analogy**: Imagine you're splitting a large file drawer into multiple smaller drawers. Each drawer contains a different set of files (rows), but all drawers are labeled with the same categories.

- **Example**: A large customer database is split such that Shard 1 contains customer records with IDs from 1 to 1,000, and Shard 2 contains records with IDs from 1,001 to 2,000.

- **Use Case**: Horizontal sharding is often used when the volume of data is too large for a single database to handle, and query performance is slowing down due to the size of the dataset.

### 2. **Vertical Sharding**

In vertical sharding, columns of a table are split across multiple shards. Each shard contains a subset of the columns. For example, one shard might store customer names and emails, while another shard stores customer addresses and order histories.

- **Analogy**: Imagine splitting a spreadsheet where one shard contains the first few columns and another shard contains the remaining columns.

- **Example**: A user profile table might be split into two vertical shards:
    - Shard 1: Basic user information (user_id, name, email).
    - Shard 2: Detailed profile information (address, preferences, activity).

- **Use Case**: Vertical sharding is useful when certain columns are accessed or updated more frequently than others, and separating them can improve performance.

---

## Sharding Strategies

### 1. **Range-Based Sharding**

In **range-based sharding**, data is partitioned into ranges based on the value of the sharding key. For example, records with `user_id` from 1 to 1,000 go into one shard, while records with `user_id` from 1,001 to 2,000 go into another shard.

- **Pros**:
    - Easy to implement and query.
    - Data is ordered logically, which can be helpful for range queries.

- **Cons**:
    - Can lead to **hotspots**, where one shard gets more traffic than others. For example, if all recent users are in a specific range, that shard may become a bottleneck.

  > **Analogy**: Imagine a library where books are arranged by genre. If one genre (e.g., "fiction") becomes much more popular than others, the fiction section could become crowded, while other sections remain underutilized.

### 2. **Hash-Based Sharding**

In **hash-based sharding**, a hash function is applied to the sharding key to determine which shard the data should be stored in. This ensures that the data is spread more evenly across the shards.

- **Pros**:
    - Provides a more even distribution of data, reducing the likelihood of hotspots.
    - Simple to implement and ensures balanced loads across shards.

- **Cons**:
    - Range queries can be difficult or impossible, since data is not stored in any logical order.

  > **Analogy**: Imagine distributing data randomly across different servers. No one server becomes too loaded, but finding a specific range of data (e.g., all users between IDs 1 and 1000) is more complex.

### 3. **Geographic (Location-Based) Sharding**

In **geographic sharding**, data is divided based on geographic location. For example, users from North America might be stored in one shard, while users from Europe are stored in another.

- **Pros**:
    - Reduces latency for users in different geographic regions by keeping their data close to them.
    - Helps in compliance with data regulations like GDPR, which require data to be stored in certain regions.

- **Cons**:
    - Uneven distribution of data across regions. One region might have significantly more users or data than another.

  > **Analogy**: Imagine organizing libraries by city. Each city has its own local library to store and manage data relevant to its residents.

### 4. **Directory-Based Sharding**

In **directory-based sharding**, a centralized directory is used to keep track of which shard contains which data. When a query is made, the directory is consulted to determine which shard the data resides in.

- **Pros**:
    - Provides flexibility in deciding where to place data.
    - Makes it easier to move data between shards as the directory can be updated accordingly.

- **Cons**:
    - Directory lookup can introduce a performance overhead.
    - If the directory goes down, the entire system can fail.

  > **Analogy**: Imagine a librarian keeping a record of which shelf each book is located on. You consult the librarian before looking for a book.

---

## Key Considerations for Sharding

### 1. **Sharding Key Selection**
- Choosing the right **sharding key** is critical for effective sharding. The sharding key determines how data is distributed across shards.
- A poor choice of sharding key can lead to imbalanced shards (e.g., some shards storing much more data than others).

> **Tip**: Choose a sharding key that distributes data evenly. For example, avoid using fields that can be sequential (e.g., timestamps), as this can lead to hotspot issues.

### 2. **Rebalancing Shards**
- Over time, some shards may grow faster than others, causing them to become unbalanced. Rebalancing is the process of redistributing data across shards to maintain balance.
- Rebalancing can be challenging and requires careful planning to avoid downtime.

> **Example**: If one shard becomes too large, data from that shard may need to be split into two smaller shards to distribute the load evenly.

### 3. **Cross-Shard Queries**
- Queries that need to fetch data from multiple shards can be more complex and slower.
- One solution is to use **scatter-gather** queries, where the query is sent to all relevant shards, and the results are aggregated.

> **Example**: If a user has data stored in multiple shards (e.g., order history in one shard and payment details in another), a cross-shard query is needed to gather all the relevant information.

### 4. **Data Consistency**
- Maintaining consistency across shards can be difficult, especially in systems that require strong consistency.
- Sharding often leads to **eventual consistency**, where updates may not be immediately visible across all shards.

### 5. **Resharding**
- As the system grows, you may need to **reshard** the data by splitting or merging shards.
- Resharding can be complex and time-consuming, especially if the system is already in production.

> **Tip**: Plan for resharding early in the design process to minimize disruptions.

---

## Tools and Technologies for Sharding

1. **MongoDB**
    - Supports horizontal sharding out of the box. Data is sharded across multiple servers based on a user-defined sharding key.

2. **Cassandra**
    - A NoSQL database that distributes data across nodes using a consistent hashing strategy. Ideal for systems that need horizontal scaling.

3. **MySQL with ProxySQL**
    - MySQL does not natively support sharding, but sharding can be implemented using **ProxySQL** or other middleware solutions that handle query routing and shard management.

4. **HBase**
    - A distributed NoSQL database that automatically shards data into regions and stores them across multiple servers.

---

## Real-World Example of Sharding

### Example: Sharding a User Database for a Social Media Platform

- **Scenario**: A social media platform with millions of users is growing rapidly. Each user has profile data, posts, and followers. As the user base grows, a single database struggles to keep up with the demand.

- **Solution**: The platform decides to shard the user database by **User ID**. Users with IDs from 1 to 10,000 are stored in one shard, and users with IDs from 10,001 to 20,000 are stored in another shard. Each shard contains all of the data for the users in that range.

- **Challenges**: Over time, certain shards become larger than others, as some users are more active. The platform needs to implement **resharding** to split the busiest shards into smaller pieces to maintain balanced performance.

---

## Important Terms

- **Shard**: A horizontal partition of a database.
- **Sharding Key**: The key used to determine which shard a piece of data belongs to.
- **Hotspot**: A situation where one shard receives disproportionately more traffic than others.
- **Cross-Shard Query**: A query that requires fetching data from multiple shards.
- **Resharding**: The process of redistributing data across shards to maintain balance.

## Conclusion

Database sharding is a powerful technique for scaling databases as data grows. By distributing data across multiple shards, sharding allows systems to handle large volumes of data efficiently. However, choosing the right sharding strategy, managing cross-shard queries, and planning for resharding are crucial challenges to consider when implementing sharding in a system.

---

## Further Reading

- [Sharding in MongoDB](https://docs.mongodb.com/manual/sharding/)
- [Horizontal Sharding in MySQL](https://severalnines.com/database-blog/guide-database-sharding-pros-and-cons)
- [Sharding in Cassandra](https://cassandra.apache.org/doc/latest/architecture/sharding.html)
