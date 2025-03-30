## Storage Types
### Block Storage
- Dividing data into fixed-sized blocks and storing them on block devices like hard drives or SSDs
- Accessed using low-level block protocols
  - Typically through Storage Area Networks (SAN) or Direct-Attached Storage (DAS)
  - Block can be read or written directly
- Low latency and high performance storage
- Suitable for databases, VMs, high performance applications
- Options
  - Amazon Elastic Block Store (EBS)
  - Azure Virtual Machines (VMs) with managed discs service for block volumes
  - Google Cloud zonal persistent disk

### Object Storage (or Blob Storage)
- Files are divided into little parts and dispersed over hardware in a flat structure
- Typically use a RESTful API for accessing and managing data
- Highly scalable
  - Provides built-in redundancy and fault tolerance
  - Can handle large number of requests concurrently
- Each object can have an associated metedata for rich data management and search capabilities
- Suitable for large amounts of unstructured data
  - Like media files (documents, images, videos, audios), backups, log files
- Can be used together with database systems to store and manage different types of data
- Options
  - Amazon Simple Storage Service (S3)
  - Azure Blob Storage
  - Google Cloud Storage buckets

### File Storage
- Files stored under a hierarchical directory structure
- Accessed using file level protocols like Network File System (NFS) or Server Message Block (SMB)
- Can be implemented using Network Attached Storage (NAS) devices or distributed file systems
- Supports metadata like permissions, timestamps, file attributes
- Suitable for file systems and file based applications
- Options
  - Amazon Elastic File System (EFS)
  - Azure Files
  - Google Cloud Filestore

## Databases
### RDBMS
- Relationship based tabular data model that requires a predefined schema
- Scalability
  - Excel in vertical scaling (by increasing horsepower like memory, cpu)
  - Scaling across multiple servers is challenging and time-consuming
- Why use SQL
  - ACID compliance: Reduces anomalies and protects integrity
  - Data is structured & unchanging, can be queried and accessed in a structured way
  - Provides strong relationship between entities
- Examples: MySQL, PostgreSQL, Oracle, SQL Server
- Data Examples: customer records, financial transactions

### NoSQL
- Unstructured data model using key-value, document, wide-column, graph
- Dynamic schema where columns can be added on the fly
  - Every row doesn't need to have data for each column
- Scalability
  - Excel in horizontal scaling (by adding more servers easily)
- Sacrifice ACID compliance for scalability and processing speed
- Instead of ACID, it complies with BASE
  - BASE: Basically Available, Soft State, Eventually Consistent
  - Prioritizes availability and performance over strict consistency
- Why use NoSQL
  - Data type is not a bottleneck
  - Storing large volumes of data that have little structure
  - Easily scale across multiple data centres
  - Rapid development, no prep required, data structure can be updated easily
- Examples: MongoDB, Cassandra, Redis

### Key Value Stores
- Type of NoSQL database where each piece of data is stored under a unique key
- Used to store data that is accessed frequently
- Types
  - In-memory: Stored in memory for fast access
  - Persistent: Stored on disk for durability
- Used for caching, session management, real-time analytics, metadata
- Simpler to use and more scalable than other types of databases
- Not well suited for complex & structured data that requires advanced querying capabilities

## Indexing
- Enhances the speed of data retrieval and finds rows like a library catalog
- Index columns that have large range of distinct values and are used frequently
  - Avoid indexing columns that are rarely read and frequently written to
- Affects write performance: insert, update, delete
  - Since an index can become large due to additional keys
  - While adding rows or making updates
    - We have to write the data as well as update the index
  - Avoid unnecessary indexes and delete the ones no longer being used
- Categories
  - Single-level: Direct mapping between the index and the actual data
  - Multi-level: Hierarchical layers with better performance, B & B+ trees
  - Clustered
    - Rows with similar values for the clustering key are stored collectively
    - Aligns information rows based on the order of the clustering key
    - Optimizes retrieval operations, in particular for range queries
    - Insert & update can be slower
  - Non-clustered
    - Has a unique value for each record
    - Separate order for the index and the records
    - Fast retrieval, insert & update faster than clustered
- Data structures: B tree, B+ tree, Hashmaps, Bitmap

## Eventual Consistency
- Consistency model where all data replicas are eventually converged to a consistent state
- It allows the replicas to be inconsistent for a short period
  - To enable high availability and partition tolerance
- Asynchronous updates
