## Consistent Hashing
- Distributed hashing technique to achieve load balancing
  - Distributes keys uniformly across a cluster of nodes
  - Minimizes the need of rehashing when nodes are added or removed
- Represented by a virtual ring structure
  - Number of locations in this ring is not fixed
  - Server nodes are hashed at random locations on this ring
  - Request are also hashed on the same ring with the same hash function

## Requirement
- With n servers, using an intuitive hash function like (key % n) has two major drawbacks
- Not horizontally scalable
  - When a new host is added, all existing mappings are broken
  - And needs to be updated causing a downtime
- Not load balanced
  - Can cause non-uniformly distributed data
  - Some servers may have heavy load while others may remain idle or almost empty

## Working
- Deciding a server for a request
  - Assuming that the clockwise traversal of the ring
    - Corresponds to the increasing order of location addresses
  - Each request can be served by the server node
    - That appears first while traversing clockwise
  - Only k/n keys need to be reassigned on adding or removing a node
    - Where k is total number of keys & n is total number of nodes
- To map a key to server
  - Hash it to single integer
  - Move clockwise on the ring until the first host is encountered
    - That host contains the key
  - When a host is added, reassign all the predecessor keys on the ring to the new host
  - When a host is removed, reassign all the predecessor keys on the ring to next host
- For load balancing, the real data is essentially randomly distributed
  - This may make the keys unbalanced due to non-uniform data
  - To handle this issue and avoid non-uniform data from hash function
    - Virtual replicas are introduced
  - Instead of mapping each cache to a single point on the ring
    - Map it to multiple points on the ring, i.e. replicas

## Advantages
- Consistent hashing is a good strategy
  - For distributed caching system and DHT (Distributed Hash Table)
- Distributes data across cluster
  - That minimizes reorganization when nodes are added or removed
- When hash table is resized (new host is added)
  - Only k/n keys need to be remapped
  - Objects are mapped to same host if possible
  - Takes share from a few host without touching other's shares
- When host is removed
  - Objects on that host are shared by other hosts
