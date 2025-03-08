### DSA for System Design
- Consistent Hashing
- Vector Clocks
- Paxos Algorithm
- Map Reduce
- Distributed Hash Tables (DHT)
- Gossip Protocol
- Quorum-based replication

### DSA Examples
- Hash tables for caching frequently requested web pages
- Graphs for social networks
  - Network of friends can be represented as a graph
  - BFS traversal can be used to find connections between users & suggest new connections
- Trie for autosuggestion innsearch engines or messaging apps
  - Helps to predict and suggest most likely words or phrases as user types
- Priority queues for task scheduling
- Dijkstra's algorithm for routing in GPS navigation systems
- Binary search for searching rec ords based on a unique identifier
- Segment tree for range queries
  - Can be utilized to efficiently calculate total revenue in financial systems

### Maintain Concurrency and Parallelism
- Concurrency: Executing multiple task in overlapping time periods
- Parallelism: Simultaneous execution of multiple tasks
  - By dividing into subtasks that can be processed concurrently
- Concurrency Control: Locks (read-write locks), Mutexes, Semaphores
- Atomic Operations
- Task Scheduling
- Pipeline Processing
- Parallel Reduction: Aggregate data from multiple sources
- Fork Join: Divide a task into subtasks, execute concurrently, combine the results
