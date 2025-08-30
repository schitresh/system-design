# Uber

-   Platforms: uber, ola, lyft
-   Ride sharing service that connects passengers and drivers
-   Passengers will see all the nearby available drivers
-   Drivers need to regulary notify the service
    -   About their availability to pick passengers and their current location

## Requirements

-   Functional Requirements
    -   Customers can request a ride
    -   Nearby drivers are notified about this request
        -   Which they can accept or decline
    -   Once a driver accepts
        -   They can constantly see each other's current location
        -   And contact each other until the trip finishes
    -   Upon reaching the destination, driver will mark the ride complete
        -   And becomes available for the next ride

## Estimation

-   Number of Users: 300 M
-   Number of Drivers: 1 M
-   Traffic
    -   Daily Active Users: 1 M
    -   Daily Active Drivers: 500 K
    -   Daily Rides: 1 M
    -   Assume that all active drivers notify their current location every 3 secs

# Component Design

## Dynamic Grid

-   We will take the solution discussed in yelp
    -   And modify it to make it work for the use cases for uber
    -   The quad tree is efficient to find the nearby drivers
        -   But not built to support frequent updates
-   All active drivers report their locations every 3 seconds
    -   We need to be update the data structure to reflect that
    -   Updating the quadtree will take a lot of time and resources
    -   To update a driver to its new location
        -   We must find the right grid based on the driver's previous location
        -   If the new position does not belong to the current grid
            -   Remove the driver from current grid and insert in the correct grid
        -   After this, if the new grid reaches the maximum limit of drivers, repartition it
-   We need to quickly propagate the current location of all the nearby drivers
    -   To all the active customers in that area
    -   When ride is in progress
        -   System needs to notify about the current location of the car
        -   To both the driver and the customer

## Updating Driver location

-   Do we need to modify the quad tree every time a driver reports their location
    -   If we don't update, it will have some old data
        -   And won't reflect the current location of drivers correctly
    -   Our purpose is find the nearby drivers efficiently
-   What if we keep the latest position reported by all the drivers in a hash table
    -   And update the quadtree a little less frequently (say 15 seconds)
    -   We need to store the driver id, their present and old location

### Storage

-   Driver Location: { driver_id: { old_lat, old_long, new_lat, new_long } }
-   One driver entry will require 35 bytes
    -   The id will take 3 bytes (enough for 1 M drivers)
    -   And other fields will take 8 bytes
-   The request for update location will require 19 bytes
    -   Since it won't include old lat & long
-   Storage: 1 M \* 35 bytes = 35 MB
-   Bandwidth: 1 M \* 19 bytes = 19 MB/3s (location is update every 3 second)

### Distribution

-   Do we need to distribute this hash onto multiple servers
    -   All this information can fit on one server
    -   And our memory and bandwidth requirements don't require this
-   But for scalability, performance and fault tolerance
    -   We can distribute the driver location hash table on multiple servers
    -   We can distribute based on driver id to make it completely random
-   These servers will also broadcast the location updates to active customers
-   And notify the respective quad tree server to refresh the driver location every 15s

## Broadcasting Driver Location

-   How can we efficiently broadcast the driver's location to customers
-   We can have a push model
    -   Where the server will push the positions to all the relevant users
-   We can have a dedicated notification service
    -   That can broadcast the current location of drivers to all the active customers
    -   This can be based on pub/sub model
-   Pub/sub model
    -   When a customer opens the app
        -   The client will query the server to find nearby drivers
    -   On server side
        -   The customer will be subscribed to the updates of nearby drivers
        -   And then return the list of these drivers to the customer
    -   It can maintain a list of customers (subscribers)
        -   And broadcast the current location of the driver
        -   Whenever we have an update in the driver location hash table
-   Notify respective quad tree server to refresh driver's location every 15 seconds

### Storage

-   We need to store the driver id (3 bytes) and customer ids (8 bytes)
-   Assume that five customers subscribe to one driver
-   Storage requied: 500 K _ 3 + 500 K _ 5 \* 8 = 21 MB
    -   500 K daily active drivers \* 3 bytes for driver id
    -   500 K daiy active drivers _ 5 subscriptions/driver _ 8 bytes for customer id
-   Bandwidth
    -   For active drivers: 5 subs/driver \* 500 K = 2.5 M
    -   For active customers: 500 K _ 5 subs _ 19 bytes = 47.5 MB/s
        -   The location request takes 19 bytes (driver id (3), lat (8), long (8))

### New Driver Info

-   What will happen when a new driver enters the area the customer is looking at
-   To add a new customer/driver subsription dynamically
    -   We need to keep track of the area the customer is watching
    -   But this will make our solution complicated
-   What if the client pulls this information from the server instead
    -   Clients can send their current location
    -   And the server will find all the nearby drivers from the quad tree
    -   Upon getting the driver list, the client can update their current location
    -   Clients can query every five seconds to limit the number of round trips to server
    -   This is simpler than the push model

## Repartioning Grid

-   Do we need to repartition the grid as soon as it reaches the maximum limit
-   We can have a buffer to let each grid grow a little bigger beyond the limit
    -   Before we decide to partition it
-   Let's say the grids can grow/shrink an extra 10% before partitioning/merging
    -   This will reduce the load for a grid partition/merge on high traffic grids
    -   And will account for dynamic movement

## Requesting Ride

-   The client will put a request for a ride
-   One of the aggregator servers will take the request
    -   And ask the quad tree servers to return nearby drivers
    -   It will collect the results and sort them by ratings
    -   It will send a notification to the top (say 3) drivers simultaneously
        -   The one to accept the request first will be assigned the ride
        -   Other drivers will receive a cancellation request
        -   If none of them accepts, it will send a notification to next three drivers
-   Once a driver accepts the request, the client will be notified

## Ranking Service

-   Refer yelp

# Advanced Issues

-   How to handle clients on slow and disconnecting networks
-   What if a client gets disconnected when they are a part of a ride?
    -   How will we handle billing in such a scenario?
-   How about if clients pull all the information instead of servers always pushing it?

# Scalability

-   Refer yelp
