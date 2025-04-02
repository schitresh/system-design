# Youtube or Netflix
- Platforms: youtube, netflix, vimeo, dailymotion
- Users can upload, view, search, share videos
- Users can also like, dislike, comment, report videos
- It also shows number of views & comments on a particular video

## Requirements
- Functional Requirements
  - User can upload, view, search, share videos
  - Record stats of videos like likes, dislikes, total views, etc.
  - User can also add & view comments on a video
- Non-Functional Requirements
  - Should be highly reliable, any uploaded video should not be lost
  - Should be highly available
  - Consistency over availability
  - Real-time experience while watching videos and should not feel any lag
- Extended Requirements
  - Video recommendations
  - Trending videos
  - Channels
  - Subscriptions
  - Watch later
  - Favorites

## Estimation
- Traffic
  - Total users: 1.5 B
  - Daily active users: 800 M
  - Views: 800 M users/day * 5 views/user = 4 B/day = 46 K/s
  - Assume that upload-view ratio is 1 : 200
  - Uploads: 46 K / 200 = 230 videos/s
- Storage
  - Assume that every minute, 500 hours of videos are uploaded
  - Assume that 1 min of video requires 50 MB on average
    - Since videos need to be stored in multiple formats
  - Storage required for videos: 1.5 TB/min = 25 GB/s
    - 500 hours/min * 60 mins/hour * 50 MB/min
    - This does not include video compression & replication
- Bandwidth
  - Assume that each video upload takes a bandwidth of 10MB/min
  - Incoming: 500 hrs/min * 60 mins/hr * 10 MB/min = 300 GB/min = 5 GB/sec
  - Outgoing: 1 TB/sec (1:200 ratio)

## API
- upload_video
  - api_key, title, description, tags, category_id, language, location, video
- search_video
  - api_key, search_query, location, maximum_videos_to_return, page_token
- stream_video
  - api_key, video_id, offset, codec, resolution
  - Codec & resolution are required to support different devices

## High Level Design
- Processing Queue
  - Each uploaded video will be pushed to processing queue
  - To be dequeued later for encoding, thumbnail generation, storage
- Encoder: To encode each uploaded video into multiple formats
- Thumbnail Generator: To generate a few thumbnails for each video
- Database
- Video & thumbnail storage

## Database
- Schema
  - Video (id, title, description, size, thumbnail, uploader, likes, dislikes, views)
  - Comment (id, video_id, user_id, comment, created_at)
  - User (id, name, email, age, address, registration details)
- Storage
  - Regular Database: SQL
  - Video Storage: Distributed file storage like HDFS, GlusterFS

### Thumbnails
- Thumbnails are small files, say a maximum of 5 KB each
  - Assuming every video will have five thumbnails
  - We would need a efficient storage system that can serve a huge read traffic
- Read traffic for thumbnails will be huge compared to videos
  - Users will watch one video at a time
  - But they might look at a page that has 20 thumbnails of other videos
- If we store them on a disk storage
  - Given that we have a huge number of files
  - We will have to perform a lot of seeks to different locations on the disk
  - This is inefficient and will result in high latencies
- Bigtable can be a reasonable choice
  - It combines multiple files into a block to store on the disk
  - Efficient in reading a small amount of data
- Cache popular thumbnails, it's easy to cache them since they are small in size

# Component Design
- System would be read heavy
- So we will focus on building a system that can retrieve videos quickly

## Manage Read Traffic
- We should segregate the read traffic from write traffic
- Since we will have multiple copies of each video
  - We can distribute the read traffic on different servers
- For metadata, we can have master-slave configuration
  - Writes will go to the master first and then gets applied to the slaves
  - But slaves may return the stale results for some time
    - This might be acceptable in our system as it would be for short while
    - And user can see the new videos after a few milliseconds

## Uploading & Encoding
- Videos could be huge
  - If the connection drops while uploading
  - We should suppport resuming from the same point
- Encoding
  - Store newly uploaded videos on the server
  - Add a new task to processing queue to encode the video into multiple formats
  - Once the encoding is completed, notify the uploader
    - And make the video available for viewing & sharing

## Video Deduplication
- With a huge number of users uploading a massive amount of video data
  - Our service may have to deal with widespread video duplication
- Duplicate videos often differ in aspect ratios or encodings
  - Can contain overlays or additional borders
  - Or can be excerpts from a longer original video
- Proliferation of duplicate videos can have an impact on many levels
  - Wastage of storage space
  - Degraded cache efficiency by taking up space that could be used for unique content
  - Network usage
    - Will increase the amount of data that must be sent over the network
    - To in-network caching systems
  - Energy wastage due to higher storage, inefficient cache and network usage
- User experience will also be impacted
  - Duplicate search results
  - Longer video startup times
  - Interrupted streaming

### Inline Deduplication
- De-duplication makes most sense early when a user is uploading a video
  - As compared to post-processing it to find that the video is duplicate
- Will save resources used to encode, transfer and store the duplicate video
- When user starts uploading, run video matching algorithm to find duplicates
  - Like block matching, phase correlation
- If a duplicate is found
  - Stop the upload and use the existing copy
  - Or continue the upload and use the new video if it is of higher quality
- If the uploaded video is a subpart of an existing video or vice versa
  - We can intelligently divide the video into smaller chunks
  - So that we only upload the parts that are missing

# Scalability
## Sharding
- We will have a huge number of new videos every day
  - And our read load is extremely high
  - Hence, we need to distribute the data on multiple machines

### Based on User Id
- All user data will stored on the same server
- To search videos by title, we will have to query all servers
  - And each server will return a set of videos
  - A centralized server will then aggregate and rank these results
- Issues
  - If a user becomes popular
    - It can increase the load on that specific server
    - And create a performance bottleneck
  - Over time, some users can end up storing a lot of videos
    - This can lead to non-uniform distribution of data
- To recover from these issues
  - We will have to repartition/redistribute the data
  - Or use consistent hashing to balance the load between the servers

### Based on Video Id
- The hash function will map each video id to a randonm server
- To find videos of a user, we will have to query all servers and aggregate them
- This solves the problem of popular users but shifts it to popular videos
- We can introduce cache in front of db servers to store popular videos

## Replication
- Read traffic: Secondary servers
- Meta-data: Master-Slave, write -> master, read -> slave
- Multiple copies of each video

## Caching
- To serve globally distributed user, we need a massive-scale video delivery system
- The service should push its content closer to the user
  - Using a large number of geographically distributed video cache servers
- We need a strategy to maximize user performance
  - And evenly distribute the load on cache servers
- We can go with the 80-20 principle
  - Cache 20% of daily read volume that will generate by 80% traffic
  - We can change this strategy after getting more data

### Content Delivery Network (CDN)
- CDN is a system of distributed servers that deliver web content to a user
  - Based on the geographic location of user
  - The origin of the web page and a content delivery server
- CDNs replicate content in multiple places
  - There's a better chance of videos being closer to the users
  - With fewer hops, videos will stream from a friendlier network
- CDN machines make heavy use of caching and can mostly serve out of memory
- Less popular videos (1-20 views per day) that are not cached by CDNs
  - Can be served by the servers in various data centers

## Load Balancing
- We should use consistent hashing among the cache servers
  - This will also help in balancing the load among the cache servers
- We can use a static hash-based scheme to map videos to hostnames
  - But it can lead to uneven load on logical replicas
    - Due to different popularity of each video
  - These uneven loads for logical replicas
    - Can then translate into uneven load distribution on corresponding physical servers
- To resolve this issue
  - Any busy server in one location
    - Can redirect to a less busy server in the same cache location
  - We can use dynamic HTTP redirections for this scenario
- Drawbacks of redirections
  - Since the service tries to load balance locally
    - It leads to multiple redirections if the receiver host can't serve the video
  - Each redirection requires a client to make an additional HTTP request
    - It leads to higher delays before the video starts playing back
  - Inter-tier (or cross-data-center) redirections
    - Lead a client to distant cache location
    - Because the higher tier caches are only present at small number of locations

## Fault tolerance
- Use consistent hashing for distributing load among database servers
- It will also help in replacing a dead server
