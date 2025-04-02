# Twitter
- Social networking service where users post and read tweets
- Tweets are short 140 character messages
- Only the registered users can post tweets

## Requirements
- Functional Requirements
  - User can post new tweets (with a limit of 140 chars)
  - User can follow other users
  - User can mark a tweet as favorite
  - Timeline should be displayed to the user
    - Consisting of top tweets from the people that the user follows
  - Tweets can contain photos and videos
- Non-Functional Requirements
  - Service should be highly available
  - Acceptable latency for timeline generation: 200 ms
  - Consistency over Availability
    - It's fine if a user can't see a tweet for a while but it should be consistent
- Extended Requirements
  - Searching for tweets
  - Replying to tweets
  - Retweet
  - Trending topics & searches
  - Tagging other users
  - Push notifications
  - Show follow suggestions
  - Moments (Snapshot of top news, tweets & topic)

## Estimation
- Total users: 1 B
  - Each user follows: 200 people
- Traffic
  - Daily active users: 200 M
  - Tweets: 100 M/day = 1150/s
  - Daily favorites: 200 M users * 5 favorites/user = 1 B
  - Daily timeline visits: 7
    - Assume a user visits their timeline 2 times a day on average
    - And visits pages of 5 other users
  - Average number of tweets per timeline: 20
  - Tweet views: 28 B/day = 325 K/s
    - 200 M users/day * 7 timelines/day * 20 tweets/timeline
- Storage
  - Tweet size: 280 bytes + 30 bytes = 310 bytes
    - Content size per tweet: 140 chars * 2 bytes = 280 bytes
    - Metadata size per tweet (id, timestamp, user_id, etc.): 30 bytes
  - Tweet storage per day: 100 M tweets * 310 bytes/tweet = 30 GB
  - Media storage per day: 100 M tweets * (20% photos * 200 KB + 10% videos * 2 MB) = 24 TB
    - Assume that every fifth tweet has a photo on average
    - And every tenth tweet has a video
    - Assume that a photo is 200 KB and a video is 2 MB on average
- Bandwidth
  - Incoming: 24 TB / 86400 sec = 290 MB/s
  - Outgoing: 35 GB/s
    - Text: (28 B * 100%) * 280 bytes / 86400 sec = 93 MB/s
    - Photos: (28 B * 20%) * 200 KB / 86400 sec = 13 GB/s
    - Videos: (28 B * 10%) * 2 MB * 30% users watch video / 86400 sec = 22 GB/s

## Database
- We need a system that can efficiently
  - Store 1150 tweets per second
  - Read 325 K tweets per second
  - Read heavy system
- This traffic won't be distributed evenly throught the day
  - At peak time, we should expect a few thousand write requests
  - And 1 M read requests per second
- Schema
  - User (id, name, email, dob, created_at, last_login_at)
  - Tweet
    - id, user_id, content, created_at, favorites_count
    - tweet_latitude, tweet_longitude, user_latitude, user_longitude
  - UserFollow (id, user_id, follower_id)
  - Favorite (id, tweet_id, user_id)
- Type
  - Relations are required, so SQL can be used
  - Media storage: S3

# Component Design
- Refer design for 'facebook'

# Scalability
## Data Sharding
### Based on User Id
- This will store all the data (tweets, favorites, follows) of a user on the same shard
- We can get shard id using hash function like user_id % total_shards
- Issue 1: What if a user becomes popular
  - This will increase the load on that server increasing latency
  - If the shard goes down, the data would be unavailable
- Issue 2: Avid Users who end up having a lot of data
  - All of their data might not be accomodated on the same shard
  - Multiple shards will lead to non-uniform distribution and higher latency

### Based on Tweet Id
- Map each tweet id to a random server by using hash function like tweet_id % total_shards
  - Solves the problem of popular users but increases latency
  - To search for tweets, we will have to query all the servers
  - A centralized server will aggregate these results
- Timeline generation
  - Find all the people the user follows
  - Query all db servers to find tweets of these users
  - Each db server will find the tweets, sort them by recency and return top tweets
  - Merge all results and sort them again to get the top results

### Based on Tweet Creation Time
- We will have to query only a small set of servers to fetch the top tweets
- Issue: Traffic load won't be distributed
  - All the new tweets will be written to and read from one server
  - The recent server will have very high load while other servers will be sitting idle

### Based on Tweet Id and Creation Time
- We can get benefits of both the approaches
  - If we don't store tweet creation time separately and use tweet id to reflect that
  - It will be quick to find the latest tweets
  - For this, tweet id should be universally unique in the system
- Benefits of both recency and popular users
  - Decreases latency of only tweet id
  - Decreases load on single server of only creation time
- We still have to query all the servers for timeline generation
  - But the reads & writes will be substantially quicker
  - Since we don't have any secondary index (on creation time)
    - This will reduce the write latency
  - While reading, we don't need to filter on creation time
    - As the primary key has epoch time included in it

#### Generating Tweet Id
- To generate the tweet id
  - We can take the current epoch time and append an auto-incrementing number to it
  - Tweet Id = Creation Epoch Seconds + Sequence Number
  - We can reset the auto-incrementing sequence every second
  - We can figure out the shard number from this tweet id
- Size of tweet id: 48 (31 for epoch sec + 17 for seq number)
  - Let's say our epoch time starts today
  - Seconds in 50 years: 50 years * 365 days * 86400 secnds = 1.6 B
  - Bits required for epoch seconds: 1.6 B < 2^31 ~= 31
  - Bits required for sequence: 17 (can store 2^17 = 130 K new tweets)
    - On average, we are expecting 1150 new tweets/s
    - But we can get much higher number at peak times
- For fault tolerance and better performance
  - We can have two database servers to generate auto-incrementing keys
  - One generating even numbered keys and other generating odd numbered keys

## Caching
- We can introduce cache for database servers to cache popular tweets & users
  - We can use off-the-shelf solution like memcache that can store the whole tweet objects
  - Based on usage patterns of clients we can determine how many cache servers are required
  - LRU can be reasonable eviction policy
  - We can also use CDN to push content closer to geography
- If we go with 80-20 principle, 20% of tweets will generate 80% of the traffic
  - We can try to cache 20% of daily read volume from each shard
- What if we cache the latest data
  - Let's say 80% of the users view tweets from the past 3 days only
  - Storage required: 30 GB data/day * 3 days = 100 GB
  - This data can easily fit into one server
    - But we should replicate it to multiple servers
    - To distribute all the read traffic and reduce the load
- While generating a user's timeline
  - We can ask the cache servers if they have all the recent tweets
  - If we don't have enough tweets in the cache, then only we query backend servers
- Cache structure
  - It can be a hash table with
    - Key: user_id
    - Value: Doubly linked list of all the tweets from the user in past 3 days
  - Since we want the most recent data
    - We can insert new tweets at the head of the linked list
    - We can remove older tweets from tail to make space for newer tweets

## Replication and Fault Tolerance
- Since the system would be read heavy
  - We can have multiple secondary db servers for each DB partition
- The secondary servers will be used for read traffic only
  - All the writes will first go to the primary servers
  - And then will be replicated to secondary servers
- Whenever the primary server goes down, we can failover to a secondary server

## Load Balancing
- We can add load balancing layer at three places in our system
  - Between clients and application servers
  - Between application servers and database replication servers
  - Between aggregation servers and cache servers
- Initially a simple round robin approach can be adopted
  - But it won't take server load into consideration
  - To handle this, a more intelligent solution can be placed
    - That periodically queries servers about their load and adjusts traffic

## Monitoring Metrics
- Collect data to get an instant insight into how our system is doing
  - New tweets per day & per second?
  - What is the daily peak time?
  - Timeline delivery stats, how many tweets per day/second it is delivering?
  - Average latency to refresh timeline
- These will help us track if we need more replication, load balancing, caching
