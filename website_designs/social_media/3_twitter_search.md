# Twitter Search
- Stores and allows searching user tweets

## Requirements
- Search query with multiple words combined with AND, OR

## Estimation
### Tweets
- Users: 1.5 B
- Traffic
  - Daily active users: 800 M
  - Daily tweets: 400 M
  - Daily searches: 500 M
- Storage
  - Tweet size: 300 bytes
  - Daily Storage: 400 M tweets * 300 bytes = 120 GB/day = 1.38 MB/s
  - Storage for 5 years: 120 GB * 365 days * 5 years = 200 TB
  - Total storage for 5 years: 500 TB
    - Assuming that we want to keep 20% buffer: 250 TB
    - We may also want to keep an extra copy of all tweets for fault tolerance: 500 TB

### Index
- Total words: 500 K
  - Assuming that we have around 300 K english words
  - And around 200K famous nouns like names, cities, etc.
- Words index size: 500 K * 5 bytes = 2.5 MB
  - Assuming average length of a word as 5 characters
  - And that each character requires 1 byte
- Words per tweet to index: 15
  - Assume that each tweet has 40 words on average
  - We won't index prepositions, determinants, and other small words like etc.
  - Let's assume we will have around 15 words that need to be indexed
- Storage
  - Assume that we want to keep the index for the tweets in last 2 years
  - Tweets per year: 400 M tweets/day * 365 days = 146 B
  - Size of each tweet id: 5 bytes
  - Storage for tweet ids: 146 B * 2 years * 5 bytes = 1460 GB
  - Total Memory: 1460 GB tweets * 15 words/tweet + 2.5 MB words in english = 21 TB

## API
- search (returns json)
  - api_key, search_terms, max_results_to_return, sort_criteria, page_token

## Database
- We need to build an index to keep track of which words appear in which tweets
  - This will help us quickly find tweets that users are trying to search
- Since the tweet queries will consist of words
  - We need to build a index that can tell which word comes in which tweet
- The index would be a big distributed hash table
  - Key will be the word
  - Value will be a list of tweet ids that contain the word

# Component Design
## Ranking Service
- Rank search results by social graph distance, popularity, relevance, recency
- We can calculate a popularity number based on likes and comments
  - And store it with the index
- Each partition can sort the results based on this popularity number

# Scalability
## Storage
- Tweets
  - Since we need 500 TB of storage for 5 years
  - And if we assume a modern server can store up to 4 TB
  - We will need 125 servers to hold the required data
- Index
  - Since we need 21 TB for index
  - And if we assume that the server has 144 GB capacity
  - We will need 152 servers

## Sharding
### Based on tweet id
- We will have to query all the servers and aggregate the results

### Based on Words
- While building the index, we will iterate through all the words of a tweet
  - And calculate the hash of each word to find the server
  - All tweets containing a given word can be found on one server
- Issues
  - What if a word becomes popular? There will be high load on that server
  - Over time, some words can end up storing a lot of tweet ids
    - This can lead to non-uniform distribution
- To solve this, we may have to repartition the data or use consistent hashing

## Fault Tolerance
- We can have a secondary replica of each index server
  - If the primary server dies, it can failover to this server
- What if both primary & secondary index servers die at the same time?
  - We have to allocate a new server and rebuild the same index on it
  - But we don't know what words & tweets were kept on this server
    - The brute force solution would be to iterate through whole database
    - This is inefficient and we won't be able to serve any queries during this
  - We can build a reverse index to map tweet id to their index server
    - Index builder server will hold this information
      - Index server Id : Set of tweet ids
    - Whenever an index server has to rebuild itself, it can ask the index builder server

## Caching
- Memcache with LRU can store all the popular tweets in memory

## Load Balancing
- We can add load balancing layer at two places
  - Between clients and application servers
  - Between application servers and backend servers
- We can use weighted round robin to server traffic
  - And also take the server load into consideration
