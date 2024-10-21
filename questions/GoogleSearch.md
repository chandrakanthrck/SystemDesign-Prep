# System Design: Google Search

## Problem Statement

Design a search engine like Google that indexes billions of web pages and efficiently returns relevant results for user queries within milliseconds. The system should scale to handle millions of search queries per second and provide accurate, ranked results.

## Requirements

### Functional Requirements:
1. **Web Crawling**: Continuously crawl the web to gather new and updated web pages.
2. **Indexing**: Build an efficient index of the crawled web pages for fast retrieval.
3. **Querying**: Users can enter search queries, and the system returns relevant, ranked results.
4. **Ranking**: Rank search results by relevance, quality, and popularity.
5. **Autocomplete**: Provide search query suggestions as the user types.
6. **Spell Correction**: Correct misspelled search queries.
7. **Ad Placement**: Display relevant ads along with search results (optional).

### Non-Functional Requirements:
1. **Low Latency**: Search results should be returned in milliseconds.
2. **Scalability**: The system must handle billions of web pages and millions of queries per second.
3. **Freshness**: Ensure that new pages and updates are indexed regularly.
4. **Fault Tolerance**: The system should handle hardware failures gracefully.
5. **High Availability**: The system should be available 24/7 with minimal downtime.
6. **Consistency**: Ensure that the index remains consistent even as new pages are crawled.

## Components

1. **Web Crawler**:
    - Crawls the web, following links to discover new pages.
    - Stores raw HTML content in a temporary storage system for further processing.
    - Key challenge: Avoid overloading websites and handle politeness policies (robots.txt).

2. **Document Processor**:
    - Extracts relevant information such as text, metadata, and links from raw HTML.
    - Parses the HTML into structured content (keywords, metadata).
    - Builds a graph of web pages (links between pages are stored to aid in ranking).

3. **Indexer**:
    - Builds an inverted index, mapping keywords to a list of documents (URLs) containing those keywords.
    - Optimized for efficient retrieval of documents during search queries.
    - Indexes metadata such as page rank, freshness, and relevance.
    - Uses sharding and partitioning to distribute the index across multiple servers.

4. **Query Processor**:
    - Parses the user query to identify key terms and phrases.
    - Searches the inverted index for documents matching the query terms.
    - Retrieves a list of relevant documents from the index.

5. **Ranker**:
    - Ranks the list of retrieved documents based on various factors like relevance, page rank, and user engagement.
    - Relevance ranking is calculated using algorithms like TF-IDF, page rank, and machine learning models.
    - May also consider the freshness of content and user personalization.

6. **Auto-Suggest and Spell Checker**:
    - Provides real-time suggestions and spell corrections as the user types the query.
    - Auto-suggest is based on popular queries and past user behavior.
    - Spell checking uses a dictionary-based approach combined with query correction techniques.

7. **Ad System (Optional)**:
    - Displays relevant ads along with search results.
    - Ads are retrieved and ranked based on their relevance to the user’s search query and bidding information.

## High-Level Architecture

1. **Web Crawling and Indexing**:
    - Distributed web crawlers gather web pages and store raw HTML.
    - Document processors extract content and links from raw HTML.
    - Indexers build the inverted index from processed documents, optimized for retrieval.

2. **Query Execution**:
    - User submits a search query.
    - The query processor retrieves relevant documents from the index.
    - The ranker orders the documents by relevance, quality, and other factors.
    - The results are displayed to the user, optionally including relevant ads.

3. **Sharding and Partitioning**:
    - The index is too large to store on a single machine, so it is partitioned across multiple servers.
    - Sharding is done based on document IDs or keywords to distribute load evenly.
    - A load balancer directs incoming search requests to the appropriate shard.

4. **Caching**:
    - Popular queries are cached to reduce load on the system and improve response time.
    - Search results are stored in an in-memory cache (e.g., Redis) for faster retrieval.

## Data Flow

1. **Crawling**:
    - Crawlers continuously fetch new or updated pages from the web and store the HTML content.

2. **Indexing**:
    - The document processor extracts content and sends it to the indexer to update the inverted index.

3. **Query Processing**:
    - User submits a query via the UI.
    - The query is parsed, and relevant documents are retrieved from the index.
    - The ranker reorders the results, and the final list is returned to the user.

4. **Ad Placement**:
    - For commercial queries, relevant ads are selected from the ad system and displayed alongside organic search results.

## Key Challenges

1. **Scalability**:
    - The system must scale to handle billions of web pages and millions of queries per second.
    - Use horizontal scaling for the crawler, indexer, and query processor.

2. **Data Freshness**:
    - Crawlers need to keep the index updated with new or updated content.
    - Prioritize crawling popular or frequently updated websites.

3. **Ranking Algorithms**:
    - Rank search results using algorithms like PageRank, TF-IDF, and machine learning models for better relevance.
    - Personalize rankings based on user history and behavior.

4. **Handling Fault Tolerance**:
    - Use replication and redundancy to ensure availability even in case of server failures.

## Advanced Features

1. **Personalized Search Results**:
    - Tailor search results based on user preferences, search history, and location.

2. **Real-Time Indexing**:
    - Implement real-time indexing for certain categories like news to ensure that fresh content is always available.

3. **Mobile Optimization**:
    - Optimize search results and performance for mobile devices, as a large number of searches come from mobile.

## Conclusion

Designing a system like Google Search involves efficient crawling, indexing, and ranking of billions of web pages while ensuring low-latency query responses. The system must scale horizontally to handle the load and must employ various techniques for caching, load balancing, and fault tolerance.