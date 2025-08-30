# Caching

-   Short term memory with fast access that contains most recently accessed items
    -   Used in hardware, OS, web browsers, web applications
-   Not all information should be stored in cache
    -   Since it's expensive and will increase search time
-   When an item is queried
    -   Check in the cache, and return the result if present (cache hit)
    -   If not present (cache miss), it's fetched from the database and saved in the cache

### Application

-   OS: kernel extensions, application files
-   Browser: most visited websites
-   Twitter: viral tweets, celebrity profiles
-   Instagram: images keep loading but the text is displayed in slow network

### Eviction policy

-   FIFO (First In First Out)
-   LIFO (Last In First Out)
-   LRU (Least Recently Used)
-   MRU (Most Recently Used)
-   LFU (Least Frequently Used)
-   RR (Random Replacement)

### Cache Invalidation

-   If the original data changes, the cached version becomes outdated
    -   Invalidation mechanism ensures that the outdated entries are refreshed or removed
    -   Can be done using time-based expiration, event-driven invalidation, etc.
-   Write-through cache
    -   Data is written into the database and the cache at the same time
    -   Ensures data consistency and nothing is lost in case of crash, system disruptions, etc.
    -   But increases the latency of write operations
-   Write-around cache
    -   Data is written directly to permanent storage, bypassing the cache
    -   Reduces the flooding of cache with write operation that may not be re-read
    -   But a read request for recently written data will have cache miss
        -   Which may increase the latency
-   Write-back cache
    -   Data is written to cache alone and completion is immediately confirmed to the client
    -   The write to the permanent storage is done
        -   After specified intervals or under certain conditions
    -   This results in low latency and high throughput for write-intensive applications
    -   But this speed comes with the risk of data loss in case of a crash

## Levels

-   Application Server Cache
    -   User requests are stored in cache and returned when the same request comes again
    -   Can be added in in-memory alongside the application server
        -   If the number of results is small
    -   If LB randomly distributes requests
        -   It will go to different servers resulting in cache misses
        -   Can be solved using global cache, distributed cache
-   Distributed Cache
    -   Each node has a part of the whole cache space
    -   Using consistent hashing, each request is routed to where the cached request is found
-   Global Cache
    -   One single cache space is used by all the nodes
    -   If a request is not found in the global cache
        -   Approach 1: The cache finds out the missing piece of data from the database
        -   Approach 2: The node directly communicates with the database
    -   One global cache can get overwhelmed by large number of requests
-   CDN (Content Distribution Network)
    -   Group of servers that are geographically distributed over different locations
    -   Used when a large amount of static content is served by the website
