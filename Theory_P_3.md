# Summary of Theoretical Concepts. Part 2 (Columnar Databas, )

## 1. Columnar Databases

### Overview

-   **NoSQL Family**: Includes Document, Graph, Key-Value, and Wide Column Stores.
-   **Wide Column Stores**: Examples include Cassandra, HBase, and BigTable.
-   **Use Cases**: OLAP (Online Analytical Processing), Data Mining, and large-scale data repositories.

### Issues with Traditional Databases

-   **Challenges**: Large, unstructured data, random reads/writes, and lack of foreign keys.
-   **Needs**: Incremental scalability, speed, no single point of failure, low cost, and scale-out architecture.

### CAP Theorem

-   **CAP**: Consistency, Availability, Partition-tolerance. Systems can guarantee only two out of three.
-   **Cassandra**: Prioritizes Availability and Partition-tolerance with eventual consistency.

### Column Storage

-   **Row-store vs Column-store**:
    -   **Row-store**: Easy to add/modify records, but may read unnecessary data.
    -   **Column-store**: Efficient for read-intensive operations, only reads relevant data.
-   **Pros**: Data compression, improved bandwidth utilization, better cache locality.
-   **Cons**: Increased disk seek time, higher cost of inserts, and tuple reconstruction costs.

### Compression

-   **Techniques**: Run-length encoding, higher data value locality.
-   **Benefits**: Trades I/O for CPU, reduces storage requirements.

### List of Columnar Databases

-   **Examples**: Cassandra, Vertica, SybaseIQ, C-Store, BigTable, MonetDB, LucidDB.

---

### 1.1 Cassandra

#### 1.1.1 Overview

-   **Origin**: Originally designed at Facebook, now an open-source project under the Apache Foundation.
-   **Data Model**: Combines Google’s BigTable (sparse, distributed data storage) and Amazon’s Dynamo (distributed key-value store).
-   **Use Cases**: Used by companies like Netflix, Spotify, and Adobe for large-scale, distributed data storage.

#### 1.1.2 Data Model

-   **Keyspace**: Equivalent to a database in SQL, typically one per application.
-   **Column Families**: Similar to SQL tables but can be unstructured. Each row in a column family can have different columns.
-   **Columns**: Composed of three parts:
    -   **Name**: Byte array, determines sort order.
    -   **Value**: Byte array, the actual data.
    -   **Timestamp**: Used for conflict resolution (last write wins).
-   **Super Columns**: Group multiple columns under a common name, useful for denormalization.

#### 1.1.3 Query Model

-   **Cassandra vs RDBMS**:
    -   **RDBMS**: Domain-based model (what answers do I have?).
    -   **Cassandra**: Query-based model (what questions do I have?).
-   **Indexing**: Supports secondary indexes and denormalization for query optimization.
-   **Query Operations**:
    -   **get()**: Retrieve a specific column or super column.
    -   **get_slice()**: Retrieve multiple columns in a row.
    -   **multiget_slice()**: Retrieve slices for multiple keys.
    -   **get_range_slices()**: Retrieve columns across a range of keys.

#### 1.1.4 Write and Read Operations

-   **Writes**:
    -   Lock-free and fast.
    -   Writes are logged in a commit log and stored in memory (memtable) before being flushed to disk (SSTable).
    -   **Hinted Handoff**: If a replica is down, the coordinator node stores the write and forwards it when the replica comes back online.
-   **Reads**:
    -   Slower than writes due to consistency checks.
    -   **Read Repair**: If inconsistencies are found between replicas, Cassandra repairs them in the background.
-   **Quorums**:
    -   Configurable consistency levels: ANY, ONE, QUORUM, LOCAL_QUORUM, EACH_QUORUM, ALL.
    -   **Quorum Formula**: \( Q = N/2 + 1 \), where \( N \) is the total number of replicas.

#### 1.1.5 Cluster Management

-   **Gossip Protocol**: Nodes exchange information about cluster state, ensuring fault tolerance and high availability.
-   **Replica Placement**:
    -   **Simple Strategy**: Replicas are placed in a ring within a single datacenter.
    -   **Network Topology Strategy**: Replicas are placed across multiple datacenters and racks for fault tolerance.
-   **Bloom Filters**: Used to quickly check if a key exists in an SSTable, reducing disk I/O.

#### 1.1.6 Properties

-   **Tuneable Consistency**: Allows configuring consistency levels for each operation.
-   **High Availability**: No single point of failure, always-on architecture.
-   **Scalability**: Linear, elastic scalability with decentralized architecture.
-   **Fault Tolerance**: Data is replicated across multiple nodes and datacenters.

#### 1.1.7 Performance

-   **Writes**: Extremely fast (0.12 ms avg) due to lock-free architecture.
-   **Reads**: Fast (15 ms avg), but slower than writes due to consistency checks and read-repair.

#### 1.1.8 Use Cases

-   **Good Fit**: Applications requiring fast writes, large datasets (>GBs), and high availability.
-   **Not Ideal**: Applications requiring complex joins or strong consistency.

---

### 1.2 HBase

#### 1.2.1 Overview

-   **Origin**: Based on Google’s BigTable, open-sourced by Yahoo!, and now part of the Apache Hadoop ecosystem.
-   **Data Model**: A distributed, column-oriented database designed for large-scale data storage.
-   **Use Cases**: Used by companies like Facebook for real-time read/write access to large datasets.

#### 1.2.2 Data Model

-   **Tables**: Split into multiple regions, which are distributed across servers.
-   **Regions**: Each region contains Stores for ColumnFamilies.
    -   **MemStore**: In-memory storage for recent writes.
    -   **StoreFile**: On-disk storage (HFile) for persisted data.
-   **HFile**: The underlying storage format, similar to Google’s SSTable.

#### 1.2.3 Write and Read Operations

-   **Writes**:
    -   **Write-Ahead Log (WAL)**: All writes are first logged to ensure durability.
    -   **MemStore**: Writes are stored in memory before being flushed to disk.
    -   **Compaction**: Merges multiple StoreFiles to reduce read overhead.
-   **Reads**:
    -   Data is read from both MemStore and StoreFiles.
    -   **Bloom Filters**: Used to quickly check if a key exists in an HFile.

#### 1.2.4 Consistency

-   **Strong Consistency**: HBase guarantees strong consistency by using a master-slave architecture.
-   **Write-Ahead Log (WAL)**: Ensures durability and recovery in case of failures.
-   **Log Replay**: During recovery, stale logs are replayed to ensure data consistency.

#### 1.2.5 Cluster Management

-   **HMaster**: Manages metadata and region assignments.
-   **HRegionServer**: Handles read/write requests for regions.
-   **Zookeeper**: Manages cluster coordination and failure detection.

#### 1.2.6 Properties

-   **Strong Consistency**: HBase prioritizes consistency over availability.
-   **Scalability**: Scales horizontally by adding more region servers.
-   **Integration with Hadoop**: Seamlessly integrates with Hadoop for large-scale data processing.

#### 1.2.7 Performance

-   **Writes**: Fast, but slower than Cassandra due to strong consistency guarantees.
-   **Reads**: Efficient for range queries and scans.

#### 1.2.8 Use Cases

-   **Good Fit**: Applications requiring strong consistency, real-time read/write access, and integration with Hadoop.
-   **Not Ideal**: Applications requiring high availability and eventual consistency.

---

### 1.3 Comparison: Cassandra vs HBase

| Feature               | Cassandra                       | HBase                                  |
| --------------------- | ------------------------------- | -------------------------------------- |
| **Consistency Model** | Eventual consistency (tuneable) | Strong consistency                     |
| **Architecture**      | Decentralized, masterless       | Master-slave architecture              |
| **Write Speed**       | Extremely fast (0.12 ms avg)    | Fast, but slower than Cassandra        |
| **Read Speed**        | Fast (15 ms avg)                | Efficient for range queries            |
| **Scalability**       | Linear, elastic scalability     | Scales horizontally                    |
| **Use Cases**         | High availability, fast writes  | Strong consistency, Hadoop integration |

---

### 1.4 Othe Columnar Databases

#### 1.4.1 MonetDB

-   **Features**: SQL and ODMG front-ends, full vertical fragmentation for column storage.
-   **Use Cases**: OLAP and data mining.

### 1.4.2 LucidDB

-   **Storage**: Column-store tables with compressed column values.
-   **Indexing**: B-tree index for mapping row IDs to page IDs.

---

## ELK

### 2.1. **Introduction to Elasticsearch**

-   **Core Features**:

    -   **Near Real-Time Search**: Documents become searchable shortly after being indexed.
    -   **Full-Text Search**: Powered by Apache Lucene, Elasticsearch supports complex search queries.
    -   **Distributed**: Data is distributed across multiple nodes, ensuring scalability and fault tolerance.
    -   **RESTful API**: Interactions with Elasticsearch are performed via RESTful HTTP requests.
    -   **JSON Storage**: Data is stored in JSON format, making it flexible and easy to work with.

-   **ELK Stack**:
    -   **Elasticsearch**: Stores, searches, and analyzes data.
    -   **Logstash**: Aggregates, processes, and ingests data.
    -   **Kibana**: Visualizes and manages data.

---

### 2.2. **Key Concepts**

#### **Indices**

-   **Index**: A collection of documents with a similar structure. Indices are partitioned into **shards** for distributed storage and search.
-   **Inverted Index**: Elasticsearch uses an inverted index to map terms to the documents that contain them, enabling fast full-text search.

#### **Shards and Replicas**

-   **Shards**:
    -   **Primary Shards**: Handle write operations and store the original data.
    -   **Replica Shards**: Copies of primary shards stored on different nodes for fault tolerance and improved read performance.
-   **Replication**: Replicas increase fault tolerance and allow read operations to be distributed across multiple nodes.

#### **Mapping**

-   **Mapping**: Defines the structure of documents in an index, including field types and search behavior.
    -   **Dynamic Mapping**: Elasticsearch automatically detects and adds new fields to the mapping when documents are indexed.
    -   **Explicit Mapping**: Manually defined by the user to control the structure of the index.
-   **Field Types**: Elasticsearch supports various field types, including `text`, `keyword`, `date`, `integer`, `boolean`, `geo_point`, and more.

---

### 2.3. **Interactions with Elasticsearch**

-   **RESTful API**: Elasticsearch provides a RESTful API for interacting with the system.
    -   **HTTP Verbs**:
        -   **GET**: Retrieve documents, indices, or metadata.
        -   **POST**: Create new documents or indices (auto-generates IDs if not provided).
        -   **PUT**: Create or update documents or indices (requires an ID).
        -   **DELETE**: Remove documents or indices.
-   **Endpoints**: Requests can be sent via command line, tools like Postman, or Kibana’s developer tools.

---

### 2.4. **Querying in Elasticsearch**

#### **Query Context vs. Filter Context**

-   **Query Context**: Determines how well a document matches a query. Relevance scores are computed.
-   **Filter Context**: Determines whether a document matches a query. No relevance score is computed, and results are cacheable.

#### **Query Types**

-   **Leaf Query Clauses**: Simple queries that search for specific values in fields.
    -   **Match Query**: Searches for documents that match a provided text, number, date, or boolean value. Supports full-text search.
    -   **Term Query**: Searches for documents containing an exact term in a field. Suitable for structured data like keywords.
    -   **Range Query**: Searches for documents with values within a specified range (e.g., numbers or dates).
-   **Compound Query Clauses**: Combine multiple queries using logical operators.
    -   **Boolean Query**: Combines multiple queries using `must`, `should`, `must_not`, and `filter` clauses.
        -   **must**: All conditions must be true.
        -   **should**: At least one condition should be true.
        -   **must_not**: Conditions must not be true.
        -   **filter**: Conditions must be true, but no relevance score is computed.

---

### 2.5. **Aggregations**

-   **Aggregations**: Perform calculations on data and group documents into buckets.
    -   **Metric Aggregations**: Calculate metrics like sum, average, min, max, etc.
    -   **Bucket Aggregations**: Group documents into buckets based on field values, ranges, or other criteria.
    -   **Pipeline Aggregations**: Use the output of other aggregations as input.
-   **Sub-Aggregations**: Perform nested aggregations within buckets.

---

### 2.6. **Analyzers**

-   **Analyzers**: Process text fields to break them into terms for indexing and searching.
    -   **Standard Analyzer**: Splits text on word boundaries, removes punctuation, and lowercases terms.
    -   **Simple Analyzer**: Splits text on non-letter characters and lowercases terms.
    -   **Whitespace Analyzer**: Splits text on whitespace without lowercasing.
    -   **Language Analyzers**: Language-specific analyzers that handle stemming and stop words.
-   **Custom Analyzers**: Can be created if pre-built analyzers do not meet specific needs.

---

### 2.7. **Data Ingestion and Processing**

-   **Logstash**: A data processing pipeline that ingests, transforms, and sends data to Elasticsearch.
    -   **Input Plugins**: Collect data from various sources (e.g., Beats, databases, files).
    -   **Filter Plugins**: Process and enrich data (e.g., mutate, geoip, useragent).
    -   **Output Plugins**: Send data to destinations like Elasticsearch, databases, or other systems.
-   **Beats**: Lightweight data shippers that collect and send data to Logstash or Elasticsearch (e.g., Filebeat, Metricbeat).

---

### 2.8. **Kibana**

-   **Kibana**: A visualization tool for Elasticsearch data.
    -   **Dashboards**: Create interactive dashboards with various visualizations (e.g., graphs, maps, time series).
    -   **Lens**: A drag-and-drop tool for creating visualizations.
    -   **Maps**: Visualize geospatial data.
    -   **Time Series Visual Builder (TSVB)**: Perform advanced time series analysis.

---

### 2.9. **Best Practices**

-   **Shard Size**: Avoid too many small shards, as they increase overhead. Larger shards take longer to move.
-   **Mapping Updates**: Be cautious when updating mappings, as changes can invalidate existing data. Create a new index and reindex data if necessary.
-   **Query Optimization**: Use filters for exact matches and cacheable queries. Combine filters and queries for efficient searches.

---

### 2.10. **Comparison with Relational Databases**

-   **Elasticsearch vs. RDBMS**:
    -   **Table** → **Index**
    -   **Row** → **Document**
    -   **Column** → **Field**
    -   **Schema** → **Mapping**

---
