# Database Indexing

## What is Database Indexing?

**Database indexing** is a data structure technique used to improve the speed of data retrieval operations on a database table. An index allows a database engine to find rows more quickly and efficiently by minimizing the amount of data it needs to scan.

Think of an index like the index of a book: instead of flipping through every page to find a topic, you go to the index, find the relevant page numbers, and jump directly to the desired content.

---

## Why is Indexing Important?

As database tables grow larger, querying data without an index can become slow and inefficient because the database must perform a **full table scan** (scanning each row one by one) to find the required data.

Indexing improves **query performance** by allowing the database to search and retrieve data faster, especially for **large datasets**. However, there are trade-offs, such as increased storage space and slower write operations (e.g., inserts, updates, deletes) because indexes need to be maintained.

---

## How Indexing Works

An index stores a **sorted copy** of specific columns from a table, along with pointers to the corresponding rows in the original table. When a query is executed, the database uses the index to find matching rows without having to scan the entire table.

### Example:
Consider a table `Users` with 1 million records. If you query for users with a specific name (`SELECT * FROM Users WHERE name = 'John';`), without an index on the `name` column, the database would have to scan every row. With an index on the `name` column, the database can directly look up the location of rows where `name = 'John'`.

---

## Types of Database Indexes

### 1. **Primary Index**

- **Description**: A **primary index** is automatically created on the primary key of a table. It ensures that each record in the table is unique and provides the fastest way to access records based on the primary key.

- **Use Case**: Used for accessing rows by their unique identifiers, like `user_id`, `order_id`, or any primary key column.

> **Example**: In a `Users` table with a `user_id` as the primary key, the primary index ensures that each `user_id` is unique and can be retrieved efficiently.

---

### 2. **Secondary Index**

- **Description**: A **secondary index** is created on columns that are not the primary key. It helps speed up queries based on non-primary key columns.

- **Use Case**: Useful for queries that filter or sort by non-primary key fields, like searching for users by `email` or `name`.

> **Example**: If you often query users by `email`, you can create a secondary index on the `email` column to make these queries faster.

---

### 3. **Composite Index (Multi-Column Index)**

- **Description**: A **composite index** is an index on multiple columns. It can speed up queries that involve multiple conditions (e.g., `WHERE` clauses with multiple columns).

- **Use Case**: Useful for queries that filter by multiple columns, like finding orders by `customer_id` and `order_date`.

> **Example**: If you frequently query a table for users by both `last_name` and `first_name`, creating a composite index on both columns can speed up these queries.

---

### 4. **Unique Index**

- **Description**: A **unique index** enforces the uniqueness of the indexed column(s), meaning no two rows can have the same value for the indexed column(s).

- **Use Case**: Commonly used for columns that must contain unique values, such as `email` or `username`.

> **Example**: If `email` must be unique for each user, a unique index ensures that no two users can have the same `email` address.

---

### 5. **Full-Text Index**

- **Description**: A **full-text index** is designed for performing **text-based searches** within large bodies of text (e.g., articles, blog posts). It supports efficient search for words or phrases.

- **Use Case**: Used in applications like search engines or blog platforms where users search for specific text within large documents or articles.

> **Example**: A blog platform might use a full-text index to quickly search for blog posts containing the word "performance" in the title or content.

---

### 6. **Clustered vs. Non-Clustered Index**

- **Clustered Index**:
    - **Description**: A **clustered index** determines the **physical order** of data in the table. There can only be one clustered index per table, as the rows can only be sorted in one way.
    - **Use Case**: Used when data needs to be retrieved in a specific order, such as by `timestamp` or `ID`.

  > **Example**: In a `Logs` table where you often query logs by `timestamp`, using a clustered index on the `timestamp` column ensures the logs are stored and retrieved in chronological order.

- **Non-Clustered Index**:
    - **Description**: A **non-clustered index** does not affect the physical order of data in the table. It creates a separate data structure (usually a B-tree) to store the indexed data.
    - **Use Case**: Used when you need to speed up queries on columns that are not the primary key or clustered index.

  > **Example**: In a `Users` table, you might have a clustered index on `user_id` (the primary key) and a non-clustered index on `email` for fast lookups.

---

## Indexing Trade-Offs

While indexes greatly improve read performance, they come with trade-offs that need to be considered:

### 1. **Increased Storage**
- Each index requires additional disk space to store the indexed data, which can lead to higher storage costs, especially for large datasets.

### 2. **Slower Write Performance**
- Indexes need to be updated every time a write operation (insert, update, delete) is performed. This can slow down write-heavy operations since the database must update both the table and the associated indexes.

### 3. **Index Maintenance**
- Over time, as data is added, updated, and deleted, indexes can become fragmented, which can degrade performance. Regular index maintenance (e.g., rebuilding or reorganizing indexes) is necessary to ensure optimal performance.

---

## Best Practices for Indexing

1. **Index Only Where Necessary**: Don't over-index. While indexes improve read performance, they slow down writes and consume storage. Focus on indexing columns that are frequently queried.

2. **Use Composite Indexes for Multiple Columns**: If queries often filter by multiple columns, use composite indexes to speed up these queries. However, be mindful of the order of columns in the index; the index is most effective when the columns match the query's filtering order.

3. **Analyze Query Patterns**: Use query performance tools (e.g., `EXPLAIN` in MySQL, `EXPLAIN ANALYZE` in PostgreSQL) to understand how queries are using indexes and where improvements can be made.

4. **Avoid Indexing Highly Volatile Columns**: Columns that are frequently updated, like `last_login` or `order_status`, may not benefit from indexing due to the high overhead of updating the index.

5. **Monitor Index Health**: Regularly monitor index usage and health. Rebuild or reorganize fragmented indexes to maintain performance.

---

## Real-World Examples of Indexing

1. **E-commerce Search**:
    - In an e-commerce platform, indexing product names and categories can significantly speed up search queries, improving the user experience for customers looking for specific items.

2. **Social Media Platforms**:
    - Social media platforms use indexing to quickly retrieve user data, posts, and messages. Indexes on user IDs and post timestamps ensure that large volumes of data can be queried efficiently.

3. **Banking Systems**:
    - In banking systems, indexing is used to quickly retrieve transaction history for accounts. Indexes on `account_number`, `transaction_date`, and `transaction_id` speed up querying large transaction datasets.

---

## Important Terms

- **Index**: A data structure that improves the speed of data retrieval.
- **Full Table Scan**: When a database scans every row of a table to find matching data, often slow for large datasets.
- **Primary Key**: A unique identifier for a row in a database table.
- **Composite Index**: An index that includes multiple columns.
- **Clustered Index**: An index that determines the physical order of data in a table.
- **Non-Clustered Index**: An index that does not affect the physical order of the table’s data, storing pointers to the data instead.

---

## Conclusion

Indexing is a critical technique in database optimization, allowing for fast and efficient querying of data. By understanding the different types of indexes and their trade-offs, system designers can build databases that are both performant and scalable. However, it's essential to strike a balance between read performance and the overhead of maintaining indexes during write operations.

---

## Further Reading

- [Database Indexing - Wikipedia](https://en.wikipedia.org/wiki/Database_index)
- [MySQL Indexing Best Practices](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html)
- [PostgreSQL Indexing Documentation](https://www.postgresql.org/docs/current/indexes.html)
