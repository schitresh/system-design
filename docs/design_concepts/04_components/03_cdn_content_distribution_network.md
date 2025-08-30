# CDN (Content Distribution Network)

-   Globally distributed network of servers
    -   Designed to enhance the performance and availability of web content
-   CDNs reduce latency and accelerate the delivery of static assets
    -   Like images, videos, scripts, main page of website
-   Stores copies of these assets on servers strategically positioned around world
-   Automatically routes requests to the nearest server, reducing load time
-   Offloads some traffic from the origin server and distributes to multiple servers

## Factors

-   Content Distribution
    -   How the system should distribute web content material globally
    -   Includes techniques for replication & synchronization of content material
    -   Ensuring that the latest content is updated across all CDN nodes
-   Caching
    -   Specifying caching approach for each static and dynamic content material
    -   Figuring what content needs to be cached, for how long, cache eviction policy
    -   Keeping it coherent with database is a maintenance, solution is cache invalidation:
        -   Write-through (to database and cache)
        -   Write-around (to database)
        -   Write-back (to cache and later to database)
-   Load Balancing across CDN nodes
-   Content Purge Mechanism
    -   Mechanism to invalidate or update cached content material
    -   Situations under which content material should be purged
    -   How rapidly the purge must propogate
-   Content Optimization
    -   Techniques for image compression and minification
        -   To enhance the rate of content delivery
    -   Techniques to optimize content and delivery for mobile devices
        -   Considering responsive design and adaptive content

## Components

-   Edge Servers
    -   Distributed servers located in proximity to end users
    -   Responsible for caching and delivering content quickly
-   Origin Server
    -   Central repository where original content is stored and managed
    -   Serves as the source for content distribution
-   Content Distribution Nodes
    -   Network nodes responsible for routing and optimizing content delivery within CDN
    -   Ensures efficient traffic management
-   Control Plane
    -   Software or services that manage and orchestrate the CDN's operations
    -   Including content caching, routing, and load balancing

## Applications

-   Streaming media: Delivering video and audio content with minimal buffering and lag
-   Software distribution
    -   Distributing software updates and patches efficiently to global users
    -   Providing faster and reliable downloads for software applications
-   E-commerce
    -   Optimizing online shopping experience by ensuring fast and reliable content delivery
    -   For product images and descriptions
-   Gaming: Minimizing latency and providing seamless gameplay experience
-   API delivery: Improving performance and scalability of APIs used by mobile apps & other servies
