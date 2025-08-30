# Sharding (or Data Partitioning)

-   Splitting a large dataset into smaller chunks (or shards)
    -   And distributing them across multiple machines
    -   Required when the database or the table becomes large and affects performance
-   Technique for horizontal scaling of databases
    -   After a certain point, it is cheaper and more feasible to scale horizontally
    -   Rather than growing it vertically by adding powerful servers
-   Benefits
    -   Manageability
    -   Performance: Reduces query times since it has to go through fewer rows
    -   Availability: If an outage happens, only the specific shards will be affected
    -   Load balancing
-   Challenges
    -   Complicated task and can result in data loss if not implemented properly
    -   Sometimes shards become unbalanced (may be it outgrows or is more frequently accessed)
    -   Joining data from multiple shards may be expensive

## Considerations

-   How to split the data across shards
-   How to balance the data across shards as the amount of data changes over time
-   How the queries will be directed to the correct shard
    -   Either by using a dedicated routing layer or by including the shard info in the query
-   How the system will handle the failure of one or more shards
    -   Including recovery and data redistribution
-   How the sharded database will perform in terms of read & write speed

## Partitioning Methods

-   Horizontal
    -   Row and range based
    -   Examples
        -   Sharding based on the first letter of names
        -   Sharding based on the created_at timestamp
    -   Choose ranges carefully to avoid unbalanced servers
        -   E.g. names starting with letters in shard A-M may be more than in M-Z
-   Vertical
    -   Splitting an entire column(s) from a table
        -   Or hosting similar tables on the same server
    -   Some shards may contain highly accessed tables or columns leading to uneven workload
    -   Examples
        -   Keep users and profile on the same server and photos to different servers
        -   Split preference based columns from user table to a different table
-   Directory Based
    -   Maintains a lookup service
        -   That stores the mapping for each entity to the database servers
    -   Can dynamically scale by adding or removing shards
        -   Without any changes to the application logic
    -   The central directory that stores the mappings can be a single point of failure
    -   Introduces an additional layer of overhead for referring the directory

## Partitioning Criteria

-   Key or hash based
    -   An attribute or a group of attributes
        -   Are selected to work as the input for hash function
    -   Applying the hash function to this input yields a partition number
        -   The function should always return the same partition for a given input
    -   Adding a new server will require changing hash function and data redistribution
        -   This data redistribution can be minimized using consistent hashing
    -   It can become a bottleneck
        -   If the sharding key is not well-distributed
        -   Or if certain keys are accessed more frequently
-   List
    -   Each partition is assigned a list of values
    -   For example, users from a particular cities will be stored in a particular partition
-   Round robin
    -   Data is evenly distributed across partitions in a cyclic manner
    -   Each partition is assigned the next available data item sequentially
    -   May result in uneven data distribution and inefficient data retrieval
    -   Does not optimize specific query patterns or access patterns
-   Composite
    -   Combining any of the above partioning schemes
    -   E.g. first apply a list partitioning scheme and then a hash based partioning
    -   Consistent hashing could be considered a composite of hash and list partitioning
        -   Where the hash reduces the key space to a size that can be listed

## Challenges

-   Joins and denormalization
    -   Joins can be costly after sharding
        -   Since data has to be compiled from multiple servers
    -   Workaround: Denormalize database so that queries can be performed in single table
    -   Issue: Perils of denormalization like data inconsistency
-   Referential integrity
    -   Enforcing data integrity constraints like foreign keys can be difficult
    -   Most RDBMS don't support multi-server foreign keys
        -   So they have to be enforced in application code
    -   In such cases applications have to run regular SQL jobs
        -   To clean up the dangling references
-   Rebalancing
    -   We may need to change our sharding scheme
        -   Maybe the data distribution is not uniform
        -   Or there is a lot of load on a shard
    -   Hence, we may have to create more shards or rebalance existing shards
        -   And move the existing data to new locations
    -   Doing this without downtime is extremely difficult
    -   Using a scheme like directory based paritioning can make it easier
        -   But at the cost of increasing the complexity of the system
        -   And creating a single point of failure (for lookup service/database)
