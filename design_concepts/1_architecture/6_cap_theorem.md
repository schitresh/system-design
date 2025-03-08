## CAP Theorem
- Factors: Consistency, Availability, Partition Tolerance
- Only two out of CAP can be guaranteed simultaneously
- We cannot build a general data store
  - That is continually available
  - Sequentially considtent
  - And tolerant to any partition failures
- When designing a distributed system, trading off among CAP is the first thing to consider

## Consistency
- All nodes have the same data at the same time
- Achieved by updating the nodes before allowing further reads

## Availability
- Every request gets a response in a bounded amount of time
- System is up despite node failures
- Achieved by replicating data across servers

## Partition Tolerance
- System continues to work despite message loss or partial failure
- Communication or network failure between nodes is a common cause of partitions
- System can gracefully recover from partitions once the partition heals
- Data is sufficiently replicated across combinations of nodes and networks
  - To keep the system up through intermittent outages

## Trade Offs
- CAP system is not possible
- To be consistent, all nodes should see the same set of updates in the same order
  - But if the network suffers a partition
    - Updates in one partition might not make it to other partitions
    - Before a client reads from the out-of-date partition
  - The only thing to cope with this possibility
    - Is to stop serving requests from the out-of-date parition
    - But then it is no longer 100% available
- If two servers (S1, S2) cannot communicate (partition)
  - And data is updated in S1, we can do one of these for S2:
    - Send the last consistent data (no-tolerance: CA)
    - Deny the service for S2 requests (unavailable: CP)
    - Return whatever data is present in S2 (unconsistent: AP)

### CA System
- Delivers consistency and availability unless there is a partition between any nodes
- In that case, it sends the last consistent data
- Examples: RDBMS, PostgreSQL

### CP System
- When a partition occurs, the system shuts down the non-available nodes
- Application: Banking Transactions
  - Each transaction must be accurately reflected across all servers
  - Even if individual branches face network disruption
- Examples: MongoDB, Redis, BigTable, HBase

### AP System
- When a partition occurs, all nodes remain available
- But those at the wrong end of partition might return an older version of data
- Application: Social Media
  - Users expect immediate access to their newsfeeds
  - Even if parts of the netwrok are temporarily down
  - Slight inconsistencies are tolerable like latest post not visible, old likes count
- Examples: Cassandra, DynamoDB, CounchDB

## Hybrid System
- Switch seamlessly between modes at different stages of the worflow
- Application: Online Shopping Cart (AP + CP System)
  - Allow uninterrupted browsing if any network glitches occur (AP)
  - But when confirming the order and processing payment, it should be consistent (CP)
