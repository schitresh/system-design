## Consistency
- Ensuring that all the nodes have the same view of the data at any given time
- Despite possible concurrent operations and network delays

## Strong Consistency
- Also known as strict consistency
- Every read operation receives the most recent write operation's value or an error
- Ensures that all clients see the same sequence of updates
  - And that updates appear to be instantaneous
- Requires coordination and synchronization between distributed notes
  - Which can impact system performance & availability
- Example: Traditional SQL database system with a single master node and multiple replicas

## Eventual Consistency
- Allows replicas of data to diverge temporarily
  - But ensures that they will eventually converge to the same value
- Better availability and performance than strong consistency
- Example: Amazon DynamoDB

## Causal Consistency
- Preserves the causality between related events
- If event A causally precedes event B, all nodes will agree on this ordering
- Ensures that clients observing concurrent events
  - Maintain a consistent view of their causality relationship
  - Essential for maintaining application semantics and correctness
- Example: Collaborative document editing application
  - Where users can concurrently edit different sections of a document
  - If user A makes edits that depend on the content written by user B
  - All users should observe these edits in the correct causal order

## Read-Your-Writes Consistency
- Ensures that the individual clients observe their own updates immediateely
- After a client writes a value to a data item
  - It should always be able to read that value or any subsequent value it has written
- It is important for maintaining session consistency in applications
  - Where users expect to see their own updates reflected immediately
- Example: Social media platform
  - Where users expect to immediately see the new post or comments published by them

## Monotonic Consistency
- Ensures that if a client observes a particular order of updates (reads or writes) to a data
  - It will never observe a conflicting order of updates
- Prevents the system from reverting to previous states
- Example: Distributed key-value store
  - If a client reads A, B, C, then it will never later observe C, A, B

## Weak Consistency
- Does not provide any guarantees about when or if replicas will converge
- Allows for concurrent updates and may result in temporary inconsistencies
- Used in systems where low latency and high availability are prioritized over strict consistency
- Example: Distributed caching system like Redis or Memcached

## Conflict Resolution Techniques
- Last Writer Wins (LWW)
  - Favor the update with the latest timestamp or version
  - Might lead to data loss or inconsistency in some scenarios
- Merge Strategies
  - Use custom merge strategies depending on the specific requirements
  - Reconcile conflicting updates based on application specific semantics and user preferences
