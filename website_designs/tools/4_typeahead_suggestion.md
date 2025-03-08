# Typeahead Suggestion
- Real-time suggestion service that recommends terms to users as they enter text
  - Suggests known and frequently searched terms
- It tries to predict the query based on the characters the user has entered
  - And gives a list of suggestions to complete the query
  - It helps the user to articulate their search queries better

## Requirements
- Functional Requirements
  - Suggest top 10 terms starting with whatever user has typed
- Non-Functional Requirements
  - Suggestions should appear in real-time within 200ms
  - Efficiently store huge amount of strings that can be searched on any prefix
  - Serve large number of queries

## Estimation
- Traffic
  - If we are building a service that has the same scale as google
  - Searches: 5 B/day = 60 K/s
  - Unique searches: 20%
    - Since there will be a lot of duplicates in 5 billion queries
  - Daily unique queries to index: 5 B/day * 20% unique * 10% = 100 M
    - If we only want to index the top 10% of the search terms
    - We can get rid of a lot of less frequently search queries
- Storage
  - Average query: 15 characters
    - If on average each query consists of 3 words
    - And there are 5 characters in each word on average
  - Average query size: 15 chars * 2 bytes = 30 bytes
    - Assuming we need 2 bytes to store a character
  - Storage for current day: 100 M indexed queries/day * 30 bytes/query = 3 GB
  - Storage for year long queries: 3 GB/day * 365 * 20% = 22 GB
    - We can expect some growth in this data every day
    - But we should also remove some terms that are not searched anymore
    - Assume we have 2% new queries every day
  - Year Storage: 3 GB for current day + 22 GB for year long queries = 25 GB

## Database
- Database cannot serve large number of queries
  - To search on prefix in real-time with minimum latency
- Need to store index in memory using highly efficient data structure like trie

# Component Design
- We need to serve a lot of queries with minimum latency
  - We need to store a lot of strings in such a way that users can search with any prefix
  - We cannot depend on some database for this
  - We need to store our index in memory in a highly efficient data structure
  - One of the most appropriate data structures for this purpose is trie

## Typeahead Trie
- Trie is a tree like data structure used to store words & phrases
  - Each node stores a character of the phrase in a sequential manner
- We can merge nodes that have only one branch to save storage space
  - E.g. if capital is the only word in the T branch
    - Then we can merge TAL like C -> A -> P -> I -> TAL
  - Or if we have only words having AP after C
    - C -> AP -> I -> TAL
    - C -> AP -> T -> A -> IN
- Should we have case insensitive trie?
  - Let's assume our data is case insensitive

## Ranking Searches
- Store the count of searches that terminated at each node
  - E.g. if users have searched 'captain' 100 times
  - Store this number with the last character of the phrase
- When user types 'cap'
  - We can get the searches with phrase 'cap' by traversing that sub-tree
  - And also get the count of times they were selected to be searched
- Another criterias to consider
  - Recency, user location, language, demographics, personal history

## Traversing Trie
- Given a prefix, it can take a long time to search these results
  - Given the amount of data we need to index, we should expect a huge tree
  - Even traversing a sub-tree would take a long time depending on its depth
  - Large amount of data to be indexed, huge tree
- To solve this, we can store top 10 suggestions with each node
  - But it will require a lot of extra storage
  - We can optimize the storage
    - By storing only references of the terminal nodes of these top suggestions
    - Rather than storing the entire phrase
- To find suggested terms
  - We need to traverse back using the parent reference from the terminal node
  - We will also need to store the frequency with each reference
    - To keep track of top suggestions

## Building Trie
- We can efficiently build the trie bottom up
- Each parent node will recursively call all the child nodes
  - To calculate their top suggestions and their counts
- Parent nodes will combine top suggestions from all of their children

## Updating Trie
- Assuming five billion searches every day, i.e. 60 K queries per seconds
  - Updating trie for every query will be extremely resource intensive
  - And this can hamper the read requests
  - One solution for this could be to update the trie offline at regular intervals
- As the new queries come, log them and track their frequencies
  - Either we can log every query or do sampling and log every 1000th query
  - This will only track queries that are searched for more than 1000 times
- We can have a Map-Reduce setup
  - To process the logging data periodically, say every hour
  - These MR jobs will calculate frequencies of all the searched terms in the past hour
  - We can then update the trie with this new data
- Before updating, we can take the current snapshot of the trie
  - We can make a copy of the trie on each server to update it offline
    - Once done, we can switch to the updated one
  - Another option is to have a master-slave configuration for each trie server
    - We can update the slave while the master is serving traffic
    - Once updated, we can make the slave the new master
    - And later update the old master to start serving the traffic too

## Updating Frequencies
- We can update only the differences in frequencies
  - Rather than recounting all searched items from scratch
- We can add & subtract the frequencies
  - Based on Exponential Moving Average (EMA) of each term
  - It gives more weight to the latest data
- After inserting a new term in the trie
  - We will go to the terminal node of the phrase and increase its frequency
- Since we're storing the top 10 queries in each node
  - It is possible that this particular search term
    - Jumped into the top 10 queries of a few other nodes
  - So we need to update top 10 queries for those nodes then
    - We have to traverse back from the node to all the way up to the root
- For every parent
  - We check if the current query is part of the top 10
    - If so we update the correspoonding frequency
  - If not, we check if the current query's frequency is high enough
    - To be part of the top 10
    - If so, we insert this new term and remove the term with the lowest frequency

## Removing Term
- We may need to remove a term due to legal issue, hate, piracy, etc.
- We can completely remove such terms when the regular update happens
- Meanwhile, we can add a filtering layer on each server
  - Which will remove such terms before sending them to users

## Permanent Storage of Trie
- How to store trie in a file so that we can rebuild the trie easily
  - This will be needed when a machine starts
- We can take a snapshot of the trie periodically and store it in a file
  - This will enable us to rebuild a trie if the server goes down
- Start with the root node and save the trie level by level
  - With each node, store the character it contains and the children count
  - Right after each node, we should put all of its children
- Example
  - C
    - A
      - R -> T
      - P
    - O -> D
  - Stored as: C2, A2, R1, T, P, O1, D
- It is hard to store top suggestions and their count
  - Because the trie is being stored top down
    - Child nodes are not created before the parent
    - So there is no way to store their references
  - So we have to recalculate them while rebuilding the trie
    - Each node will calculate its top suggestions and pass it to its parent
    - Each parent node will merge results from all of its children
  - But how will we get this lost data since it's based on user activity?

## Client Side Optimizations
- Clients can use debouncing
  - Hit the servers at regular intervals (500ms)
  - Hit the server only if the user has not pressed any key for 50ms
  - If the user is constantly typing, cancel in-progress requests
- Clients can pre-fetch some data from the server to save future requests
- Clients can store the recent history of suggestions locally
  - We can also take location & language into account
  - We can also store these on the server for each user for better personalization
- Establish an early connection with the server
  - As soon as the user opens the website, the client can open a connection
  - So when a user types the first character
    - The client doesn't waste time in establishing the connection

# Scalability
## Data Partitioning
- Although the index can easily fit on one server
  - But we can still partition it to meet our requirements
  - For higher efficiency and lower latency

### Based on First Letter
- Store phrases in separate partitions based on their first letter
- We can also combine certain less frequently occuring letters
- But it can lead to unbalanced partitions
  - If we decide to put all terms starting with letter 'E' in one partition
  - And later we realize that we've too many terms starting with 'E'
    - That we cannot fit into one partition
  - This problem will occur with every statically defined scheme
    - It is not possible to calculate if each partition will fit on one server statically

### Based on Hash of Terms
- Pass the term to a hash function to generate the server number
- This will make our term distribution random and minimize hotspots
- To find a term, we will have to query all the servers and aggregate all the servers

## Based on Server Capacity
- Partition the trie based on maximum memory capacity of the servers
- Keep storing data on a server as long as it has memory available
- Whenever a sub-tree cannot fit into a server
  - Break the parition there to assign that range to this server
  - And move on to the next server to repeat the process
- E.g. Server 1: A to AABC, Server 2: AABD to BXA, Server 3: BXB to CDA
  - If the user has typed 'A' or 'AA', we have to query both server 1 & 2
  - But if the user has typed 'AAA', we need to query only server 1
  - We can have a load balancer in front of the trie server
    - Which can store this mapping and redirect traffic
- If we're querying from multiple server
  - We need to merge the results at the server side to aggregate results
    - This will require another layer of servers between load balancers & trie servers
    - To aggregate the results and return the top results to the client
  - Or make the clients do that
- If there are a lot of queries for terms starting with 'cap'
  - The server holding it will have a high load compared to others

## Caching
- Caching top searched terms would be extremely helpful
- Small percentage of queries will be responsible for the most traffic
- We can have separate cache servers in front of the trie servers
  - Holding most frequently searched terms and their typeahead suggestions
- We can aslo build a simple machine learning model
  - To predict the engagement on each suggestion
  - Based on simple counting, personalization, trending data, etc.
- Use CDNs to cache geographically searched terms that are used frequently

## Fault Tolerance
- We can have a master-slave configuration
  - If the master dies, the slave can take over after failover
  - Any server that comes back up, can rebuild the trie based on the last snapshot
