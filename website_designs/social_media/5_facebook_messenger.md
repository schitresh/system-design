# Facebook Messenger
- Instant message service where users can send text messages to each other

## Requirements
- Functional Requirements
  - Support one-on-one conversations between users
  - Keep track of online/offline status of users
  - Support persistent storage of chat history
- Non-Functional Requirements
  - Real-time chat experience with minimum latency
  - System should be highly consistent
    - Users should see the same chat history on all of their devices
  - High availability is desired but not over consistency
- Extended Requirements
  - Group chats
  - Push notifications for new messages users got when they were offline

## Estimation
- Traffic
  - Daily Active Users: 500 M
  - Daily Messages: 40/user = 20 B/day
- Storage
  - Average size of a message: 100 bytes
  - Storage required per day: 20 B messages/day * 100 bytes = 2 TB/day
  - Storage required for 5 years: 2 TB/day * 365 days * 5 years = 3.6 PB
- Bandwidth
  - Incoming: 2 TB / 86400 sec = 25 MB/s
  - Outgoing: Same as incoming (25 MB/s)
    - The same incoming message will go out to another user

## Application Server
- We will need a chat server that will be the central piece
  - Orchestrating all communications between users
  - A user will send a message to another user through this chat server
  - The server will store the message to its database and pass it to the recipient
- Workflow
  - User A sends message to User B through chat server
  - Server receives the message and sends an acknowledgement to user A
  - Server stores the message in its database
  - Server sends the message to User B
  - User B receives the message and sends an acknowledgement to the server
  - Server notifies user A that message has been delivered successfully to user B

# Commponent Design
- Let's try to build a simple solution where everything runs on one server
- At the high level, our system needs to handle the following use cases
  - Receive incoming messages and deliver outgoing messages
  - Store and retrieve messages from the database
  - Track which user is online or has gone offline
    - Notify all the users about changes in this status

## Message Handling
- To send messages to another user
  - A user needs to connect to the server and post messages for that users
- To get messages from the server
  - The user has two options: pull model & push model
- Traffic
  - Possible concurrent connections: 500 M (daily active users)
  - Assuming a modern server can handle 50K concurrent connections
  - Total servers required: 10 K
- How do we know which server holds connection to which user
  - We can introduce a software load balancer in front of our chat servers
  - That will map user_id to a server_id to redirect a request

### Pull Model
- User can periodically ask server if any new messages
  - Server needs to keep track of messages that are still waiting to be delivered
    - Server will return all these pending messages once the user connects to the server
- To minimize latency the user will have to check the server frequently
  - And most of the time they will get an empty response
  - This will waste a lot of resources
  - So this does not look like an efficient solution

### Push Model
- User can keep a connection open with the server
  - Server will notify the user whenever there are any new messages
  - Server does not need to keep track of pending messages
- It has minimum latency since messages are delivered instantly
- Open connection can be maintained through long polling or web sockets
- Long Polling
  - Client requests info from the server
  - Server holds the request and sends response whenever there is new data
    - Instead of sending an empty response immediately
  - After getting response
    - Client can immediately issue another request for future updates
  - The request can timeout, in which case the client has to open a new request
- Tracking open connections
  - Maintain a hash table with user_id as key and connection_object as value
  - Whenever server receives a message for a user
    - It can look up the hash table for the connection_object
    - And send the message on the open request
- If the receiver is offline or if its long poll request timed out
  - Server can store the message for a while and retry when the receiver reconnects
  - Or, we can ask the sender to retry sending the message
  - If there are any deliver failures, they can be notified to the user

### Sequencing of messages
- We can send the timestamp of the message from the client
  - But what if the time in user devices are not synced (e.g. 5 mins ahead)
- We can store the timestamp when the message was received by the server
  - But it can still have ordering issues
    - U1 sends M1 to U2 received at T1 by the server
    - U2 sends M2 to U1 received at T2 by the server, such that T2 > T1
    - The server will then send M1 to U2 and M2 to U1
    - U1 will see M1 first and then M2
    - U2 will see M2 first and then M1

## Storing and Retrieving Messages
- Whenever the chat server receives a new message, it needs to store it in the database
- To do so, we have two options
  - Start a separate thread, which will work with db to store the message
  - Send an async request to db to store the message
- Things to keep in mind
  - How to efficiently work with the database connection pool
  - How to retry failed requests
  - Where to log requests that failed after multiple retries
  - How to retry these logged requests when the issue is resolved

### Storage System
- We will have a large number of messages that need to be inserted into database
- A user will access messages in sequential order, latest to oldest
- So we need to have a database that can support
  - Very high rate of small updates
  - Fetching a range of records quickly
- We cannot afford to read/write a row from database
  - Every time a user receives/sends a message
  - That will have high latency for basic operations
  - And will create a huge load on databases
  - So we cannot use RDBMS like MySQL or NoSQL like MongoDB
- These requirements can be met with a wide column database like HBase
- Clients should paginate while fetching data from the server
  - Page size could be different for different clients
  - E.g. mobiles have smaller screens, so fewer messages would suffice

### HBase
- HBase is a column-oriented key-value NoSQL database
- Can store multiple values against one key into multiple columns
- Modeled after google's big table, runs on top of hadoop distributed file system (HDFS)
- Groups data together to store new data in a memory buffer
  - Once the bugger is full, it dumps the data to the disk
  - Helps storing a lot of small data quickly
  - Also helps fetching rows by the key or scanning ranges of rows
- Efficient in storing variably sized data

## User Statuses
- We need to keep track of online/offline status of users
  - And notify all relevant users whenever these statuses change
- Since we are maintaining a connection object on the server for all active users
  - In a hash table (refer 'push model' under message handling)
  - We can figure a user's current status from this
- Broadcasting each user status change to relevant active users
  - Will consume a lot of resources (500 M active users at any time as estimated before)
- When a client starts app, it can pull the current status of all the friends
  - Whenever the user sends a message to another user that has gone offline
    - Update the status on the client
  - Whenever any new friend comes online
    - The server can broadcast its status with a delay of few seconds
    - Keep this buffer time in case the friend goes offline immediately
  - For the friends that are being show on the user's viewport
    - Client can pull their status at regular non-frequent intervals
    - Stale status is alright for a while, new user will anyway broadcast their status
  - When the client starts a new chat with another user, pull the status at that time

# Scalability
## Data Partitioning
- Since we will be storing a lot of data (3.6 PB for 5 years)
- We need to distribute it onto multiple database servers efficiently

### Based on User Id
- All messages of a user will be on the same database
  - This will be quick to fetch chat history of a specific user
  - Required shards (if one shard is 4 TB): 3.6 PB / 4 TB = 900
  - Let's say we keep 1 K shards, so shard number will be hash(user_id) % 1000
- In the beginning, we can start with fewer database servers
  - With multiple shards residing on one physical server
  - Since we can have multiple database instances on a server
    - We can easily store multiple partitions on a single server
  - Our hash function needs to understand this logical paritioning scheme
    - So that it can map multiple logical partitions on one physical server
- Since we will store an unlimited history of messages
  - We can start with a large number of logical partitions
    - Which will be mapped to fewer physical servers
  - As our storage demand increases, we can add more physical servers
    - To distribute our logical partitions

## Based on Message Id
- Not a good scheme
- Messages of a user will scattered on separate shards
- Fetching a range of messages of a chat would be very slow

## Cache
- We can cache a few recent conversations (say last 5)
  - With a few recent messages (say last 15)
- Since we decided to store all of the user's messages on one shard
  - Cache for a user should entirely reside on one machine too

## Load Balancing
- We will need a load balancer in front of the chat servers
  - That can map user_id to a server that holds the connection for the user
- Similarly, we would need a load balancer for the cache servers

## Fault Tolerance
- If a server goes down
  - Should we have a mechanism to transfer the active connections to some other server?
- It's extremely hard to failover TCP connections to other servers
- An easier approach can be to have clients automatically reconnect
