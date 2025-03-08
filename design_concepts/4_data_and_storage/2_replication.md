## Replication
- Sharing information to ensure consistency between redundant resources
- Creating and maintaining duplicate copies of a database on different servers
- Data remains accessible even if one server fails
- Improves data reliability
  - Since there are multiple copies to restore data in case of corruption or loss
- Helps distributing the workload among servers improving performance and scalability
- Can be used to bring data closer to users reducing latency

## Types
- Master-Slave
  - Copy and synchronize data from a primary database (master)
    - To one or more secondary databases (slaves)
  - Master database is responsible for all write operations
  - The changes made to the master database are replicated to the slave databases
- Master-Master
  - There are multiple master databases
  - Changes made to any master database are replicated to all other master databases
  - If conflicting writes occur, conflict resolution mechanisms are needed
- Snapshot
  - Creates a copy of the entire database at a specific point in time
  - And then replicates that snapshot to one or more destination servers
  - Typically done for reporting, backup, or distributed database purposes
- Transactional
  - Keeping multiple copies of a database synchronized in real-time
  - Any changes in one database (publisher) are immediately replicated to other databases (subscribers)
  - Example: Live stock market with constantly changing prices
- Merge
  - Allows both the central server (publisher)
    - And its connected devices (subscribers) to update data
  - With multiple parties editing the data, conflicts are bound to occur
  - Pre-defined rules or user interventions are employed to resolve conflicting changes
  - Example: Shared documents

### Strategies
- Defines how data is selected, copied and distributed across databases
- Full Replication
  - Copying the whole database ensuring that replicas have exact copies
  - Example: Product catalog of e-commerce websites
- Partial Replication
  - Only specific tables, row or columns are replicated
  - Example: Most frequently accessed account information in financial institution
- Selective Replication
  - Replicating data based on predefined criteria allowing granular control
  - Example: Posts liked or shared by a large number of users on a social media platform
- Hybrid Replication
  - Example: Healthcare organization
    - Full replication for critical patient data
    - Partial replication for less critical data that is accessed occasionally

## Configurations
- Synchronous
  - Changes are replicated in real-time
  - Transaction is not considered committed
    - Until at least one replica has acknowledged receiving the changes
  - Example: Banking applications
- Asynchronous
  - Changes are sent to replicas without waiting for acknowledgement
  - Allows fast processing of transactions at the cost of delay in consistency
  - Example: Product inventory data
- Semi-synchronous
  - Data changes are replicated to at least one replica synchronously
    - Ensuring consistency for critical data
  - Other replicas are updated asynchonously for better performance
  - Example: Funds transfer
