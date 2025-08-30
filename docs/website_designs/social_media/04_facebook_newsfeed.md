# Facebook Newsfeed

-   Allows people to connect with different entities
    -   Can become friends with other people
    -   Can join a group
    -   Can follow a page
-   Posts can include text, photos, videos, links
    -   It can also include status updates like milestone, location, app activity
    -   Posts are visible depending on the privacy: public, private, group
    -   Other people can be tagged (@)
    -   Related topics can also be tagged (#)
-   Newsfeed
    -   Compilation of scroll-able version of life story of friends and followed pages
    -   Constantly updating list of stories
        -   Including posts, photos, videos, status updates
        -   Activities like liked, commented, followed, shared, etc.

## Requirements

-   Functional Requirements
    -   Generate a user's newsfeed based on the posts from people, pages, groups
        -   It may contain text, images, videos
    -   A user may have many friends and follow a large number of pages/groups
    -   Should support appending new posts as they arrive to the newsfeed for all active users
-   Non-Functional Requirements
    -   Should be able to generate any user's newsfeed in real time
        -   With maximum latency of 2s as seen by end user
    -   A post shouldn't take more than 5s to make it to a user's feed
        -   Assuming a new newsfeed request comes in

## Estimation

-   Assume on average a user has 300 friends and follows 200 pages
-   Traffic
    -   Daily Active Users: 300 M
    -   Newsfeed requests: 300 M \* 5 = 1.5 B/day = 17.5 K/s
        -   Assume each user fetches their timeline 5 times a day on average
-   Storage
    -   Posts in every user's newsfeed: 500 posts
        -   We want to keep them in memory for quick fetch
    -   Average post size: 1 KB
    -   Storage per user: 500 posts/user \* 1 KB/post = 500 KB
    -   Daily storage: 300 M users/day \* 500 KB/user = 150 TB
    -   Machines Required: 150 TB / 100 GB = 1500
        -   Assuming a server can hold 100 GB

## Database

-   Relationships
    -   A user can follow other entities and can become friends with other users
    -   Both users & entities can post feed items which can contain text, images or videos
    -   Each feed item will have a user id of the author
        -   And optionally an entity id pointing to the page or the group
-   Schema
    -   User (id, name, email, phone, dob, created_at, last_login_at)
    -   Entity (id, name, type, description, created_at, category, email, phone)
    -   UserFollow (user_id, entity_or_friend_id, type)
    -   Post
        -   id, user_id, entity_id, contents, created_at, likes_count
        -   location_latitude, location_longitude
    -   PostMedia (post_id, media_id)
    -   Media (id, type, description, path, location_latitude, location_longitude, created_at)
-   Type
    -   Relational database (SQL) to store data
    -   S3 for media storage
    -   Cache to store newsfeed

# Feed Generation

-   To generate the feed for a user
    -   Retrieve list of all users and entites that the user follows
    -   Retrieve last, most popular and relevant posts for these entities
    -   Rank these posts based on relevance to the user, recency, etc.
-   Use Pagination
    -   Store this feed in the cache and return top posts (say 20)
    -   When the user reaches the end of the current feed, fetch the next 20 posts
-   We also need to consider the new incoming posts
    -   If the user is online, rank and add incoming posts at regular intervals (say 5 mins)
    -   Then notify the user about the newer items in the feed that can be fetched
-   Pre-generate feed for frequent users

## Real-time Generation

-   If the feed is generated in real-time
    -   User will have to wait while it is generated and cause high latency
-   It will be very slow for users with a lot of friends and entities
    -   It will require sorting, merging, ranking of a huge number of posts
-   For live updates
    -   Each status update will result in feed updates for all followers
        -   This can result in high backlogs for the newsfeed generation service
    -   The server pushing (or notifying about) new posts could lead to heavy loads
        -   Especially for people or pages with a lot of followers

## Offline Generation

-   Pre-generate the newsfeed and store it in a memory
    -   We can have dedicated servers to continuously generate feeds for users
    -   New feed can be generated from the last time the feed was generated
-   When a user requests for new feed
    -   We can simply serve it from the pre-generated stored location

### Structure

-   This data can be stored in a hash table in memory
    -   Key will be user_id
    -   Value will be a struct containing
        -   Linked hash map or tree map of feed items
        -   Timestamp of the last generated feed
-   We can store 500 feed items per user initially and send 20 items per page
    -   We can adjust this number later based on the usage pattern
-   When the new feed is generated
    -   Add the new items to the head of the linked hash map
    -   And evict the old ones from the tail
-   When a user wants to fetch more items
    -   The client can send the last feed item id it currently has
    -   We can jump to that feed item id in the linked hash map
        -   And return the next batch or page of feed items

### User Activity

-   Generate & store newsfeed for all users or just avid and daily users
-   Basic approach: LRU cache
    -   Remove users that haven't logged in or accessed newsfeed for a long time
-   Smarter approach: Track login patterns of users
    -   At what time of the day a user is active
    -   Which days of the week a user accesses the newsfeed

# Feed Publishing

-   When the user loads the newsfeed
    -   He has to request and pull feed items from the server
    -   When current feed ends, he can pull more data from the server
    -   The feed capacity can be different based on mobile or desktop
-   Fanout is the process of pushing a post to all the followers
    -   The push approach is called fanout-on-write
    -   The pull approach is called fanout-on-load

## Pull Model

-   Keeps all the recent feed data in memory
-   Users can pull the data from the server regularly or when required
-   Issues
    -   New data might not be in feed until the user issues a pull request
    -   Hard to find the right pull cadence
    -   Most requests will return empty response (causing wastage of resources)

## Push Model

-   Once a user has published a post, push it to all the followers
-   To efficiently handle it, users can maintain a long poll request with the server
-   Advantages
    -   No need to get friends list and fetch posts for each of them
    -   Significantly reduces read operations
-   Issues
    -   User with millions of followers has to push updates to a lot of users
    -   Pushing every post to mobile users can consume a lot of bandwidth for them

## Hybrid Model

-   Pull from popular users with high number of followers
    -   And Push the updates of other users
-   Another approach can be to limit the fanout to only the online users
-   To get more benefits
    -   Combination of 'Push to notify' and 'pull for serving' end users is good

## Feed Ranking

-   Select factors that make a post important
    -   And figure out how to combine them to calculate a final ranking score
-   For example
    -   Number of likes, comments, shares, recency, time of the update
    -   Whether the post has images or videos
-   This can be further improved by constant evaluation based on metrics like
    -   User stickiness, retention, ads engagement, etc.

# Scalability

-   Refer to twitter & instagram
-   For feed data, which is being stored in memory, we can parition based on user id
    -   For a given user, we are stroing 500 feed items
    -   So it can fit on a single server
    -   And we can easily query them on a single server
