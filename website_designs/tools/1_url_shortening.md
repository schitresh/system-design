# URL Shortening Service
- Platforms: tinyurl.com, bit.ly, goo.gl, qlink.me
- Provides short aliases that redirects to the original URL
- Short links save a lot of space when displayed, printed, messaged, tweeted
- Used for tracking and analytics
  - Audience, campaign performance, redirection frequency

## Requirements
- Functional Requirements
  - Generate a short & unique alias for a given URL
  - When users access a short link, redirect them to the original link
  - Links will expire after a standard default timespan
  - Users can optionally specify custom short link and custom expiration time
- Non-Functional Requirements
  - Should be higly available (or else URL redirections will fail)
  - URL redirection should be real-time and with minimal latency
  - Shortened links should not be predictable
- Extended Requirements
  - Tracking & analytics
  - Should be accessible through REST API

## Estimation
- System would be read-heavy
  - There will be lots of redirection requests than new URL shortenings
  - Assume the read-write ratio to be 100 : 1
- Traffic
  - Write (New short links): 500 M/month = 200/s
  - Read (Redirections): 500 M * 100 = 50 B/month = 20 K/s
- Storage
  - Storage time for each short link: 5 years
  - Total short links: (500 M/month * 12) * 5 years = 30 B
  - Storage size for each short link: 500 bytes
  - Total storage size: 30 B * 500 bytes = 15 TB
- Bandwidth
  - Write: 200/s * 500 bytes = 100 KB/s
  - Read: 20 K/s * 500 bytes = 10 MB/s
- Memory
  - Cache frequently accessed links: 20%
    - Assuming 20% URLs generate 80% traffic (80-20 Principle)
  - Read requests: 20 K/s * 86400 s/day = 1.7 B/day
  - Memory/day: 1.7 B/day * 500 bytes = 170 GB

## API
- createShortLink (returns the short link)
  - Required: api_key, original_url
  - Optional: user_id, custom_alias, expiration_date
- getShortLinkInfo
  - Required: api_key, short_link
- editShortLink
  - Required: api_key, short_link
  - Payload: original_url, user_id, custom_alias, expiration_date
- deleteShortLink
  - Required: api_key, short_link
- Throttling
  - A malicious user can put us out of the business by consuming all URL keys
  - Limit number of creations and redirections via api_key

## Database
- Constraints
  - We need to store billions of records (15 TB)
  - Each object is small (500 bytes)
  - Read-heavy (20K requests/s)
  - No relationship between records
- Schema
  - ShortLink (id, link, original_url, created_at, expiration_at)
  - User (id, email, name, api_key, created_at, last_login_at)
- Type
  - We anticipate storing billions of rows and don't need relationships
  - NoSQL key-value store (like Dynamo or Cassandra) is a good choice
  - It would be easier to scale as well

# Component Design
## Generating Key for URL
- We need to generate a short and unique for a given URL
- We can explore two solutions here
  - Encoding actual URL
  - Generating keys offline

### Encoding URL
- Compute unique hash using MD5 or SHA256 for the given URL
  - Let's use MD5 (produces 128-bit hash value)
- Encode the hash for displaying
  - Could be base36 ([a-z][0-9]), base62 ([A-Z][a-z][0-9]), or base64 ([A-Z][a-z][0-9][-.])
  - Let's use base64
  - Encoding 128-bit hash value will give a string with 21 characters
    - Each base64 char encodes 6 bits of hash value
- What should be the length of the key?
  - Using 6 letters with base64: 64^6 = 69 B keys
  - Using 8 letters with base64: 64^8 = 281 T keys
  - 6 letters should suffice (Since we estimated 30 B links for 5 years)
  - We can take first 6 letters from the encoded string with 21 chars
    - This can result in duplication since we're taking partial encoded string
- Issue 1: If multiple users enter the same URL, they will get the same key
  - Append an increasing sequence number to each URL to make it unique
    - And then generate the hash
    - Will the sequence number overflow since it will keep increasing?
    - Will appending the number impact the performance?
  - Append user_id to the URL
    - If the user is not logged in, we would need to ask the user for a unique key
    - If we have a conflict after this, we have to keep generating until a unique key
- Issue 2: What if parts of the URL are URL-encoded
  - `http://www.example.com?id=design`
  - `http://www.example.com%3Fid%3Ddesign`

### Generating Keys Offline
- We can have a standalone Key Generation Service (KGS)
  - That generates random 6 letter strings beforehand and stores them in database
- While shortening a URL, we can pick a key from this key-db
  - This will be simple & fast
  - We won't have to encode anything
  - We won't have to worry about duplications or collisions
- How to handle concurrency (multiple servers picking keys)
  - As soon as a key is picked, it should be marked as used
  - KGS can maintain 2 tables: used keys & unused keys
  - As soon as a key is given to a server, it can be moved to the used keys table
  - What if multiple servers pick up the same key?
    - Get a lock (or synchronize) before removing & passing it to server
- Keep some keys in memory so that they can be quickly provided
  - Though they should be first moved to used keys before loading in the memory
  - If KGS dies before assigning these loaded keys, they will be wasted
    - Which is acceptable given the huge number of keys we have

### Common Considerations
- Storage
  - With base64 encoding, 69 B unique 6 letter keys can be generated
  - Since one byte is required to store one char, so one key will take 6 bytes
  - Storage: 69 B * 6 bytes = 414 GB
- KGS is single point of failure
  - Can be solved by keeping standby replica of KGS
  - It will take over if the primary server dies
- Caching
  - Each app server can keep some keys from key-db to speed things up
  - If a server dies, we'll lose these cached keys
    - Which is acceptable given that we have 69 B keys
- Keep size limit on custom aliases
  - Impose a size limit on a custom alias to ensure we've consistent URL database
  - Let's assume users can specify a max of 16 chars per customer key

## Permissions
- Can users create private URLs that allow only specific users to access it?
- Store permission level (public/private) with each URL
- Create a table to store user_ids that have permission to view a specific URL
- Return HTTP 401 if the user is not permitted
- Schema: Permission (key, [user_ids])

# Scalability
## Data Partitoning
### Range Based
- Based on first letter of the URL or the hash key
- We can combine less frequently used consequent letters
- But it can lead to unbalanced Servers
  - E.g. if we decide to keep one partition for E
  - And then later realize that we have too many URLs starting with E

### Hash Based
- Based on the hash calculated on the URL or the hash key
- Hashing function will randomly distribute URLs to different partitions
  - It will map to a partition number between (1..256)
- This can still lead to over overloaded partitions
  - Which can be solved by using consistent hashing

## Caching
- We can cache frequently accessed URLs
- We can start with 20% of daily traffic
  - And then adjust the number of cache servers based on the user usage pattern
  - This requires 170 GB memory as calculated in the estimation
  - One machine will be sufficient (since a modern server can have 256 GB memory)
  - Alternatively, we can use a couple of small servers
- Eviction Policy
  - LRU (Least Recently Used)
  - We can use Linked Hash Map to store URLs, hashes & recent access timestamp
- We can replicate our caching servers to distribute load between them
  - During a cache miss, servers would hit the backend database
  - When this happens, update the cache and pass the new entry to all the cache replicas

## Load Balancers
- We can add load balancing layer at three places
  - Between Client & Application servers
  - Between Application servers & Database servers
  - Between Application servers & Cache servers
- Initially we can use a simple Round Robin approach
  - It will distributes incoming requests equally amond backend servers
  - It is simple and introduces no overhead
  - If server is dead, it will be taken out of rotation
- A problem with round robin is that server load is not taken into consideration
  - If a server is overloaded or slow, LB won't stop sending requests to it
  - Intelligent RR can be used to solve this
    - It will periodically query servers about its load and adjust traffic

## Purging / DB Cleanup
- We should purge entries when they are expired
- Have a default expiration time for each link (e.g. 2 years)
- Actively searching for expired links to remove them will put a lot of pressure on db
  - Instead, we can slowly remove expired links and do a lazy cleanup
  - Whenever a user tries to access expired link, delete the link & show an error to user
- Run a cleanup service periodically to remove expired links from storage and cache
  - It should be lightweight and run only when traffic is expected to be low
  - Should we remove keys not been used for some time, say 6 months?
    - Storage is getting cheap, we can decide to keep links forever
- After removing an expired link, put the key back to the unused-keys table

## Telemetry
- Metrics
  - How many times a short link has been used
  - Location of the users that accessed it
  - Date & time of the access, at what time of day it has highest traffic?
  - From which web page it was referred
  - Browser or platform where it was accessed
- How to store this info?
  - Keep a metadata table with a unique row per short link
  - It will get updated on each view?
  - What happens when a popular url is slammed with a large number of concurrent requests?
