# Dropbox
- Platforms: dropbox, google drive, one drive
- File hosting service (or cloud file storage)
- Enables users to store their data on remote servers
- Simplifies the storage and exchange of digital resources
  - Users can access data across multiple devices with different platforms & OS
  - Gives portable access from various geographical locations at any time
  - Offers 100% reliability and durability of data by keeping multiple copies of data
  - Unlimited storage space, though after certain limit it needs to be paid for

## Requirements
- Functional Requirements
  - Users can upload & download their files from any device
  - Files or folders can be shared with other users
  - Files & folders should be automatically synced among devices
  - Should support storing large files up to a GB
- Non-Functional Requirements
  - ACID properties should be followed for all file operations
    - Atomicity, Consistency, Isolation, Durability
- Extended Requirements
  - Support offline editing, the changes should be synced as soon as the device is online
  - Snapshotting data so that user can go back to any version of a file

## Considerations
- We should expect huge read & write volumes
- Read-write ratio is expected to be nearly the same
- Internally, files can be stored in small parts or chunks (say 4 MB)
  - If a file upload fails, only the failed chunks will be required to be re-uploaded
  - Will reduce data exchange by transferring only the updated chunks of files while syncing
  - For small changes, clients can intelligently upload diffs instead of the whole chunk
- Keeping a local copy of metadata (file name, size, etc) with client
  - Can save a lot of round trips to the server

## Estimation
- Traffic
  - Total users: 500 M
  - Daily active users: 100 M
  - Active connections: 1 M/min
  - Devices per user: 3
- Storage
  - Average number of files per user: 200
  - Average file size: 100 KB
  - Total files: 500 M users * 200 files/user = 100 B
  - Storage required: 100 B files * 100 KB/file = 10 PB

## Application Servers
- User will specify a folder to be used as workspace on their device
  - Data placed in this workspace will be synced with the cloud
  - Any modifications & deletions will be reflected to other devices
  - So we need some servers that can help clients to upload & download files to cloud
- We need to store files and their metadata (filename, size, directory, etc.)
  - And who this file is shared with
  - Some servers that can facilitate updating metadata
- We also need some mechanismm to notify all clients whenever an update happens
  - So that they can synchronize their files

# Component Design
## Client
- A client application will monitor and sync data in the workspace folder with the cloud
- It will work with storage servers to upload, download & modify files
- It will interact with synchronization service to handle any metadata updates
  - Like change in file name, size, modified date, etc.
- It will also handle conflicts due to offline or concurrent updates

### Handling File Transfer
- We can break each file into smaller chunks (say 4 MB)
  - So that only the modified chunks need to be transferred and not the whole file
  - Server and clients can calculate a hash (e.g., SHA-256)
    - If we already have a chunk with a similar hash (even from another user)
    - We don’t need to create another copy, we can use the same chunk
- We can calculate an optimal chunk size based on
  - Type of storage devices used in the cloud
    - To optimize space utilization
    - And input/output operations per second (IOPS)
  - Network bandwidth
  - Average file size in the storage
- In the metadata, keep a record of each file & the chunks that constitute it
  - Keep a copy of metadata with the client
  - It will enable us to do offline updates
  - And save a lot of round trips to update remote metadata

### Syncing Changes
- Handling slow servers
  - Clients should exponentially back off if the server is busy or not responding
  - If the server is too slow, clients should delay their retries
    - And this delay should increase exponentially
- For web clients, check for file changes regularly
- For mobile clients, sync on demand to save user's bandwidth and space

### Listen Changes on Other Clients
- Polling
  - Clients can periodically check with the server for changes
  - But since this is periodic, there can be delay in reflecting changes locally
  - If the client checks frequently, it will waste bandwidth for both client & server
    - Server will have to keep serving these requests and return empty response
  - Hence, this is not scalable
- Long Polling
  - Clients can request info from server
    - With the expectation that server may respond immediately
  - If the server has no new data for the client
    - It will hold the request open & wait for new data instead of sending empty response
    - Once the server has some new info, it will send the response to the client
  - After receiving the response
    - Client can immediately issue another request for future updates

### Components
- Based on the above considerations, we can divide the client into four parts
- Interal Metadata Database
  - Will keep track of all the files, chunks, their versions, their locations
- Chunker
  - Will split the files into chunks
  - Will reconstruct a file from its chunks
  - Will detect the chunks that have been modified and transfer those to cloud
- Watcher
  - Will monitor the local workspace folder
  - Will notify the indexer about any actions performed by users
    - E.g. creating, deleting, updating files or folders
  - Will listen to any changes happening on the other clients
    - That are broadcasted by synchronization service
- Indexer
  - Will process the events received from the watcher
  - Will update the internal metadata database about chunks of modified files
  - Once the chunks are successfully submitted/downloaded to the cloud
    - It will communicate with the remote synchronization service
    - To broadcast changes to other clients and update the remote metadata database

## Metadata Database
- Responsible for maintaining versioning
  - And metadata info about files, chunks, users, devices, workspaces
- Can be SQL or NoSQL
  - NoSQL data stores don't support ACID in favor of scalability and performance
    - So ACID properties need to be supported in synchronization service programmatically
  - Relational database can simplify the implementation of the synchronization service
    - Since it natively supports ACID properties
- Regardless of the type of database
  - Synchronization service should be able to provide a consistent view of the files
  - Especially if multiple users are working on the same file simultaneously

## Synchronization Service
- Most important part due to its critical role in managing metadata & synchronizing files
  - It will process and sync file updates made by a client
  - It will sync local databases of clients with the info stored in the metadata db
- Desktop clients will communicate with synchronization service to
  - Obtain files & updates from the cloud storage
  - Send files & updates to the cloud storage, other clients, and other users
  - If a client was offline for a long period
    - It can poll the system for new updates as soon as they come online
- When the synchronization service receives an update request
  - It will check with the metadata db for consistency and then proceed with the update
  - A notification will be sent to all subscribed users or devices to report the file update
- It should employ file transfer with low response time
  - To reduce the amount of the data that needs to be synchronized
    - It can employ a differencing algorithm
    - Which transmits only the difference between two versions of a file
    - This also decreases bandwidth consumption and cloud data storage for the end user
  - Further details have been discussed in 'handling file transfer' above

## Message Queuing Service
- It should support
  - Asynchronous message-based communication between clients and synchronization service
  - Asynchronous and loosely coupled message-based communication between distributed components
- For an efficient and scalable synchronization protocol
  - We can use this message queuing service
  - And add a communication/messaging middleware between the clients and the service
- Messaging middleware should be able to handle a substantial number of requests
  - It should be able to efficiently store any number of messages
  - In a highly available, reliable and scalable queue
- It will implement two types of queues in our system
  - Global Request Queue that all the clients will share
    - Client requests to update the metadata db will be sent to this queue first
    - Synchronization service will take these requests to update the metadata
  - Response Queues for individual subscribed clients
    - Responsible for delivering the update messages to each client
    - Required since a message will be deleted from the global queue once received by a client
- Multiple instances of synchronization service can be deployed
  - Which will receive requests from a global request queue
  - And the middleware will be able to balance their load

## Cloud/Block Storage
- Stores chunks of files uploaded by the users
- Clients directly interact with the storage to send and receive objects
- Separation of the metadata from storage
  - Enables us to use any storage either in cloud or in-house

## File Processing Workflow
- Interaction when client A updates a file that is shared with client B and C
  - Client A uploads chunks to cloud storage
  - Client A updates metadata and commits changes
  - Client A gets confirmation
  - Notifications are sent to Clients B and C about the changes
  - Client B and C receive metadata changes and download updated chunks
- If they are not online at the time of the update
  - Message queuing service keeps the update notifications
  - In their respective queues until they become available later

## Data Deduplication
- Technique for eliminating duplicate copies of data to improve storage utilization
- Can also be applied to network data transfers to reduce the number of bytes that must be sent
- For each new incoming chunk, calculate a hash
  - Compare that with all the hashes of the existing chunks
  - To see if we already have the same chunk present in our storage
- Can implement deduplication in two ways in our system

### Post-process deduplication
- New chunks are first stored on the storage device
- Later some process analyzes the data looking for duplication
- Benefits
  - Clients will not need to wait for the hash calculation
    - Or lookup to complete before storing the data
  - Ensuring that there is no degradation in storage performance
- Drawbacks
  - Unnecessarily storing duplicate data, though for a short time
  - Duplicate data will be transferred, consuming bandwidth

### In-line deduplication
- Deduplication hash calculations can be done in real-time
  - As the clients are entering data on their device
- If our system identifies a chunk which it has already stored
  - Only a reference to the existing chunk will be added in the metadata
  - This approach will give us optimal network and storage usage

## Permissions
- One of the primary concerns users will have is the privacy and security of their data
- Especially since in our system
  - Users can share their files with other users or even make them public
- To handle this, we will be storing permissions of each file in our metadata DB
  - To reflect what files are visible or modifiable by any user

# Scalability
## Metadata Partitioning
- To scale out metadata db, we need to partition it
  - So that it can store information about millions of users
  - And billions of files/chunks
- We need to come up with a partitioning scheme
  - That would divide and store our data to different db servers

### Vertical Partitioning
- Store tables related to one particular feature on one server
  - Store all the user related tables in one database
  - Store all files/chunks related tables in another database
- Although it is straightforward to implement, it has some issues:
  - Will we still have scale issues?
    - What if we have trillions of chunks to be stored?
    - And our database cannot support storing such huge number of records?
    - How would we further partition such tables?
  - Joining two tables in two separate databases can cause performance and consistency issues
    - How frequently do we have to join user and file tables?

### Range Based Partitioning
- Store files/chunks in separate partitions based on the first letter of the file path
  - Save all the files starting with letter ‘A’ in one partition
  - Those that start with letter ‘B’ into another partition
- We can even combine certain less frequently occurring letters into one database partition
- We should come up with this partitioning scheme statically
  - So that we can always store/find a file in a predictable manner
- Drawback: Can lead to unbalanced servers
  - If we decide to put all files starting with letter E into a DB partition
  - And later we realize that we have too many files that start with letter E
  - To such an extent that we cannot fit them into one DB partition

### Hash-Based Partitioning
- Take a hash of the object and figure out the DB partition
  - Can take the hash of the FileID of the File object
- Hashing function will randomly distribute objects into different partitions
  - Example: Map any ID to a number between 1 to 256, and this number would be the partition
- Drawbacks
  - Can still lead to overloaded partitions
  - Can be solved by using Consistent Hashing

## Caching
- Can have two kinds of caches in our system
- To deal with hot files/chunks, we can have a cache for Block storage
  - Can use Memcache that can store whole chunks with their respective IDs/Hashes
  - Before hitting block storage, block servers can quickly check if the cache has desired chunk
- Based on clients’ usage pattern we can determine how many cache servers we need
  - A high-end commercial server can have up to 144GB of memory
  - So one such server can cache 36K chunks
- Which cache replacement policy would best fit our needs?
  - How do we choose to replace a chunk with a newer/hotter chunk?
  - Least Recently Used (LRU) can be a reasonable policy
- Similarly, we can have a cache for Metadata DB

## Load Balancer
- We can add load balancing layer at two places
  - Between Clients and Block servers
  - Between Clients and Metadata servers
- Initially, a simple Round Robin approach can be adopted
  - Distributes incoming requests equally among backend servers
  - Simple to implement and does not introduce any overhead
  - If a server is dead, LB will take it out of the rotation and stop sending any traffic
  - But it won’t take server load into consideration
  - If a server is overloaded or slow, LB won't stop sending new requests to it
  - To handle this, a more intelligent LB solution can be placed
  - That periodically queries backend server about their load and adjusts traffic
