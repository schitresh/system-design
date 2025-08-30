# Yelp

-   Platforms: yelp, proximity server
-   Users can search for nearby places or events
    -   Like restaurants, theaters, shopping malls, etc.
    -   They can also view or add reviews for them

## Requirements

-   Functional Requirements
    -   Users can add, update or delete places
    -   Given the user's location, find all the nearby places within a given radius
    -   Users can add review about a place with text, pictures, rating
-   Non-Functional Requirements
    -   Real time search experience with minimum latency
    -   Should support a heavy search load
        -   There will be a lot of search requests compared to adding a place

## Estimation

-   Places: 500 M
-   Queries per second (QPS): 100K
-   Assume a growth of 20% in the number of places and QPS each year

## Database

-   Schema
    -   Place (id, name, description, category, latitude, longitude, rating)
    -   Review (id, place_id, rating, text)
    -   Media (id, path, type, review_id)
-   Length of place id
    -   4 bytes number can uniquely identify 500 M places
    -   But keeping future growth in mind, we can go with 8 bytes for place id

## API

-   search
    -   api_key, search_terms, category, user_location, radius
    -   max_results_to_return, page_token, sort

# Component Design

-   We need to store and index each dataset (places, reviews, media)
-   While searching for the nearby places, users expect to see the results in real-time
    -   For users to query this massive database, the indexing should be read efficient
-   Given that the location of a place doesn't change that often
    -   We don't need to worry about frequent updates of the data
-   To build a service where objects do change their location frequently
    -   Like people, taxis, etc.
    -   The design will be very different
-   Let's see the different ways to store this data and find the best suited method

## SQL Solution

-   Store all the data in a database like MySQL
    -   To perform a fast search, add index on the latitude and the longitude fields
-   To query nearby places for a given location L(X, Y) within a radius R
    -   We need to find the distance of each place from L
-   Efficiency
    -   We have estimated 500 M places which is a huge list
        -   Maybe we can narrow it down by storing the country of places
    -   We have two separate indexes, each can return huge list of places
        -   Performing intersection on the two lists won't be efficient
    -   This will have high latency, we need to narrow down the list of places

## Grids

-   Divide the world map into smaller grids to group places into smaller sets
    -   Each grid will store all the places residing within the range of latitude & longitude
-   We can uniquely identify each grid by assigning a grid id
    -   And index it for faster search
    -   We can keep the index in memory as hash
        -   With key as the grid id and value as the list of the places
-   Given a location L(X,Y) and a radius R
    -   Find all the neighboring grids
    -   And query the places within these grids to get nearby places
-   Grid Size
    -   Can be equal to the distance that we want to query
    -   It will limit the search to the grid with the location L and the eight neighboring grids
    -   Since the grids are statically defined (from the fixed grid size)
        -   Finding the grid number of any location would be easy

### Storage

-   Storage is required for indexing grid
-   Number of grids: 20 M
    -   Total area of earth is 200 M square miles
    -   Let's assume the search radius is 10 miles
-   Grid Id: 4 bytes
-   Place Id: 8 bytes
-   Index size: 4 bytes/grid_id _ 20 M grids + 8 bytes/place_id _ 500 M places = 4 GB

### Issues

-   Places are not uniformly distributed among grids
-   So it can still run slow for grids with large density of places
-   It can be solved by dynamically adjusting the grid size
    -   By breaking dense grids into smaller ones
    -   But how to map grids to locations and how to find neighboring grids?

## Dynamic Grids

-   Let's assume we don't want to have more than 500 places in a grid for faster search
-   When a grid reaches this limit
    -   Break it down into 4 equal grids and distribute places among them
-   This means that thickly populated areas like downtown San Francisco
    -   Will have a lot of small grids
-   And sparsely populated areas like Pacific Ocean will have large grids
    -   With places only around the coastal lines

### Data Structure

-   Quad tree (tree with 4 child nodes) can be used for this
    -   Each node will represent a grid and contain information about all its places
    -   If a node reaches the limit, we will break it down to create four child nodes
    -   So the leaf nodes will keep a list of places with them
-   Building a quad tree
    -   Start with the root node that will represent the whole world in one grid
    -   Break it down into 4 nodes and distribute places among them
    -   Repeat this process until no nodes have more than 500 places
-   Find the grid for a given location
    -   Start at the root and search downwards
    -   Check if the current node has child nodes
        -   If it has, move to the child node which contains the given location
        -   Else, it is the required node

### Finding Neighboring Grids

-   Only leaf nodes will contain a list of places
    -   We can connect all the leaf nodes through a doubly linked list
    -   This way we can iterate forward & backward among the neighboring leaf nodes
    -   Till we have enough places or it reaches the maximum radius
-   Another approach is to keep pointer to parent node in each node
    -   This way we can reach the sibling nodes through the parent node
    -   We can keep expanding our search for neighboring grids
        -   By going up through the parent pointers
        -   Till enough places are found or it reaches the maximum radius

### Storage for Quad Tree

-   To store places: 24 bytes \* 500 M = 12 GB
    -   For each place, if we can cache place id and latitude & longitude
    -   8 bytes for each field
-   Total Grids: 500 M places / (500 places/grid) = 1 M
    -   Each grid can have maximum of 500 places
-   Hence, there will be 1 M leaf nodes holding 12 GB data of places
-   Internal nodes: (1 M _ 1/3 nodes) _ (4 pointers \* 8 bytes) = 10 MB
    -   Quad tree with 1 M leaf nodes will approximately have 1/3rd internal nodes
    -   Each internal node will have 4 pointers for its childrend
    -   Assume that each pointer will take 8 bytes
-   Total storage: 12 GB + 10 MB = 12.01 GB
    -   This can easily fit into a modern day server

### Inserting New Place

-   When a new place is added by a user
    -   It needs to be inserted in the database as well as the quad tree
-   If the tree resides on one server, it is easy to add a new place
    -   But if it is distributed among multiple servers
    -   First find the server and the grid of the new place and add it there

## Ranking Service

-   We may also want to rank the search results not just by proximity
    -   But also by popularity or relevance
-   We can keep track of the overall popularity of each place
    -   For example, by using the average rating given by users
    -   We can store this in the database as well as in the quad tree
-   We can query top 100 places from each partition of quad tree
    -   And then aggregate these results to get the final top 100 places
-   But we didn't build our system to update data of places frequently
    -   Searching a place in the quad tree and updating its popularity
        -   Will take a lot of resources and affect search requests & system throughput
    -   Popularity of a place is not expected to be reflected within a few hours
        -   We can decide to update it once or twice a day
        -   When the load on the system is minimum

# Scalability

## Sharding

### Based on Region

-   Divide the places into region (like zip codes)
    -   Such that all the places for that region will be stored on a fixed node
-   Issues
    -   If a region becomes hot, there will be a high load on that server
    -   Over time, some regions can end up storing a lot of places than others
        -   This can cause non-uniform distribution of places
    -   To recover from these situations
        -   We will have to re-partition the data or use consistent hashing

### Based on Place Id

-   While building the quad tree, we will iterate through all the places
    -   And calculate the hash of each place id
-   To find the nearby places, we will have to query all the servers
    -   And then aggregate the results from all the servers
-   Will we have different quad tree structure on different partitions
    -   This can happen
        -   Since it is not guaranteed that we will have equal number of places
            -   In a given grid on all partitions
        -   However, we do make sure that
            -   All servers have approximately an equal number of places
    -   It will not cause any issue though
        -   As we will be searching all the neighboring grids
        -   Within the given radius on all partitions
-   Hence, this is good paritioning scheme for our use case

## Replication and Fault Tolerance

-   To distribute read traffic, we can have replicas of each quad tree server

### Master-slave replication

-   Replicas will only serve the read traffic
-   All write traffic will first go to the master and then applied to replicas
-   Replicas might not have some recently inserted places
    -   There can be a delay of a few milliseconds which is acceptable

### Primary and secondary servers

-   If a quad tree server dies, we can have a secondary replica to failover
-   If both primary and secondary servers die at the same time
    -   We will have to allocate a new server and rebuild the same quadtree
-   The brute force approach is to iterate through the whole database
    -   And filter place ids using the hash function
    -   To figure all the places that should be stored on this partition server
    -   This will be inefficient & slow
    -   We won't be able to server traffic during this
        -   And miss showing some places that should have been included
-   Another approach is to build a reverse index and rebuild the server from that
    -   That will map all the places to their quad tree server
    -   A separate quad tree index server will hold this information
    -   This can be store in a hash map
        -   Key will be the quad tree server id
        -   Value will be the hash set of all its places (will store place id, lat, long)
    -   Hash set will enable us to add or remove places from the index quickly
    -   We can also have its replica for fault tolerance

## Caching

-   To deal with popular places, we can introduce a cache in front of the database
-   We can use memcache with LRU
-   We can adjust the number of servers based on the usage pattern of users

## Load Balancing

-   Keep at two places
    -   Between user and application server
    -   Between application server and backend server
-   Modified Round Robin can be used which will also take server load into consideration
