# Instagram

-   Platforms: instagram, flickr, picasa
-   Social networking service to share photos & videos
    -   They can be shared publicly or privately
    -   Users can like these photos
    -   Users can search photos & videos based on captions & location
-   People can follow other and others can follow them
    -   The account can also be kept public or private
    -   In private accounts, only the followers can view the feed

## Requirements

-   Functional Requirements
    -   Users can upload, download, and view photos
    -   Users can perform searches based on photo tiles
    -   Users can follow each other
    -   System generates and shows the feed to the user
    -   The feed should consist of top photos from all the people the user follows
-   Non-Functional Requirements
    -   System should be highly available
    -   Acceptable latency for the feed generation is 200 ms
    -   Consistency over Availability: If a user doesn't see a photo for a while, it's fine
    -   Highly Reliable: Any uploaded photo should never be lost
-   Extended Requirements
    -   Adding tags to photos
    -   Searching photos on tags
    -   Commenting on photos
    -   Tagging users to photos
    -   Show follow suggestions

## Considerations

-   The system would be read heavy
    -   Users will view feed more often than they will upload photos
    -   So focus on building a system that retrieves photos quickly
-   Users can upload as many photos as they like
    -   Storage should be managed efficiently to retrieve these photos
    -   Data should be reliable and should never be lost
-   Should there be a limit on the photo size?
    -   We can skip it for now?
    -   For videos, it may be required, maybe 100 MB?

## Estimation

-   Traffic
    -   Total Users: 500 M
    -   Daily Active Users: 1 M
-   System would be read heavy, let's assume read-write ratio of 100 : 1
    -   Photos Upload: 2 M/day = 23/s
    -   Photos Read: 200 M/day = 2300/s
-   Storage
    -   Average Photo Size: 200 KB
    -   Daily Storage: 2 M photos/day \* 200 KB = 400 GB
    -   Storage required for 10 years: 400 GB _ 365 days _ 10 years = 1425 TB
-   Bandwidth
    -   Write: 400 GB/day = 400 GB/86400 s = 4.62 MB/s
    -   Read: 200 M requests/day \* 200 KB = 40000 G requests/86400 s = 462 MB/s

## Database

-   Schema
    -   User (id, name, email, dob, created_at, last_login_at)
    -   UserFollow (user_id, follower_id)
    -   Photo
        -   id, description, user_id, photo_path, created_at
        -   photo_latitude, photo_longitude, user_latitude, user_longitude
-   Type
    -   Use SQL since relationships and joins are required
    -   Use S3 for storing photos
-   Data Estimation
    -   User
        -   id (4) + name (20) + email (32) + dob (4) + created_at (4) + last_login_at (4)
        -   68 bytes \* 500 M users = 32 GB
    -   Photo
        -   id (4) + description (256) + user_id (4) + photo_path (256) + created_at (4)
        -   photo_latitude (4) + photo_longitude (4) + user_latitude (4) + user_longitude (4)
        -   540 bytes \* 2 M photos/day = 1 GB/day = 3.65 TB for 10 years
    -   UserFollow
        -   user_id (4) + follower_id (4)
        -   8 bytes _ 500 M users _ 500 followers/user = 1.82 TB
    -   Total space required for 10 years = 5.5 TB

## Application Servers

-   Uploading photos (writes) will be slow compared to reads
    -   System can get busy with uploading photos since it's a slow process
    -   Which can lead to read requests not being served
    -   This is because web servers have a limit for concurrent connections (say 500)
-   To handle this bottleneck
    -   We can split reads and writes into separate services
    -   We will have dedicated servers for reads and different servers for write
    -   It will also allow us to scale and optimize these operations separately

# Component Design

## Feed Generation

-   For a given user, we need to fetch the latest, most popular and relevant photos
    -   From the people that the user follows
    -   Let's assume we need to fetch top 100 photos for the feed
-   First get a list of people the user follows
    -   Then fetch metadata of the latest 100 photos from each user
    -   Finally submit all these photos to the ranking algorithm
    -   The ranking algorithm will determine the top 100 photos based on a decided criteria
-   There can be high latency to fetch all this data from multiple data
    -   And perform sorting, merging, ranking to get the final result
    -   To improve efficiency, we can pre-generate the feed and store it in a table
-   We can have dedicated servers that continuously generate user feeds and store them
    -   After this feed has been shown to the user
    -   New feed can be generated again
        -   If there are new posts since then, then they should be preferred
        -   Else old posts which didn't make to the last feed can be considered

## Showing Feed

-   What are the different approaches to send the feed contents to the users
-   Clients can pull the feed from the server
    -   On a regular basis manually whenever they need it
    -   Though new data might not be shown until clients issue a pull request
    -   If there is no new data, the pull requests might result in an empty response
-   Servers can push the feed to the users
    -   As soon as it is available
    -   Users will have to maintain a long poll request with the server
    -   If a user follows a lot of people or a celebrity with millions of followers
        -   The server will have to push updates quite frequently
-   Hybrid approach
    -   Move all the celebrity users with large number of follows to a pull based model
        -   Push data to only those users who have comparatively less number of follows
    -   Another approach can be to push updates at a certain frequency
        -   Letting users with a lot of follows/updates to regularly pull data

# Scalability

## Reliability and Redundancy

-   Losing files is not an option for our service
    -   So store multiple copies of each file
-   High availability is required
    -   So have multiple replicas of services running in the system
    -   This will also remove single point of failures
-   If only one instance of a service is required at any point
    -   We can run a redundant secondary copy that is not serving any traffic
    -   It can take control if the primary server fails

## Data Sharding

### Based on User Id

-   Will keep all photos of a user on the same shard
    -   If one db shard is 1 TB, we will need 6 shards to store 5.5 TB as estimated before
    -   Let's assume for better performance and scalability, we keep 10 shards
    -   So we can find the shard number by user_id % 10
-   To generate photo ids
    -   Each db shard can have its own auto-increment sequence for photo ids
    -   We can append shard number with each photo id
    -   This will make it unique throughout the system
-   Issues
    -   How would we handle hot users who are followed by a lot of people
    -   Some users will have a lot of photos than others
        -   Which can lead to non-uniform distribution of storage
    -   What if we cannot store all pictures of a user on one shard
        -   If we distribute photos of a user onto multiple shards, will it cause higher latency?
    -   If a shard is down or experiencing a high load
        -   It can cause unavailability of all of the user's data

### Based on Photo Id

-   Generate a unique photo id and then find a shard number like photo_id % 10
    -   This won't require appending shard number to the photo id
-   To generate photo ids
    -   Cannot have auto-incrementing sequence in each shard
    -   We can have a dedicated database instance to generate and store incrementing ids
-   This key generating db can be a single point of failure
    -   A workaround could be defining two such databases
        -   One generating even numbers and the other generating odd numbers
    -   We can put a load balancer in front of these databases with round robin
        -   These servers can be out of sync with one generating more keys than the other
        -   But this won't cause any issues in the system
    -   Alternatively, we can implement a key generation scheme like in 'URL shortening'

### Future Growth

-   We can have a large number of logical partitions to accomodate future data growth
-   In the beginning, multiple logical partitions can reside on a single physical db server
    -   One database server can have multiple instances
    -   So we can have separate databases for each logical partition on any server
-   Whenever a particular database server has accumulated a lot of data
    -   We can migrate some logical partitions from it to another server
    -   Maintain a config file (or separate database) to map logical partitions to db servers
    -   This will enable us to move partitions easily

### Feed Generation with Sharded Data

-   To fetch latest photos, we need to sort them by their time of creation
    -   To efficiently do this, we can make creation time part of the photo id
    -   As we have primary index on photo id, it will be quick to find the latest photo ids
-   Photo id can have two parts
    -   First part can have epoch time
    -   And second part will be an auto-incrementing sequence from the key generating db
    -   Then we can figure out the shard number from photo_id % 10
-   Size of photo id
    -   Let's say our epoch time starts today
        -   And we store the number of seconds for next 50 years
        -   86400 s/day _ 365 days/year _ 50 years = 1.6 billion seconds
        -   We would need 31 bits to store this number
    -   We are expecting 23 new photos per second
        -   We can allocate 9 bits for auto-incrementing sequence
        -   So every second we can store: 2^9 = 512 new photos
        -   We can reset auto incrementing sequence every second

## Cache

-   Our service would need a massive scale photo delivery system
    -   To serve globally distributed users
    -   So the content should be pushed closer to the user
    -   This can be done using
        -   Large number of geographically distributed photo cache server
        -   And using CDNs
-   We can have a cache for metadata servers for hot database rows
    -   LRU can be a reasonable cache eviction policy
-   We can cache frequently accessed photos and celebrity photos
