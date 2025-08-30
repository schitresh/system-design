# Factors To Consider

## Scalability

-   Capability of a system to grow, manage and balance the load without performance loss
-   Required to manage increased demand for requests, data, transactions
-   It's said that when designing a system for load x, plan for 10x and test for 100x
-   Factors
    -   Number of requests per day
    -   Number of database calls (read & write) per day
    -   Amount of cache hit or miss
    -   Daily active users
    -   Traffic distribution
-   Examples
    -   Surge in e-commerce platforms on big sale or holidays
    -   Buffering or low quality video streams due to strained network
    -   E-banking app can experience authentication bottlenecks due to high volume of requests

### Horizontal Scaling

-   Adding more machines to the pool of resources
-   Distributes workload across the individual units
-   Increases fault tolerance since the number of nodes increase
-   Can introduce complexity and requires load balancing management
-   Ideal for handling massive scalability needs
    -   Unlimited potential to grow
    -   Effective for microservices allowing to scale independently
    -   Useful where responsiveness and speed are critical
    -   Cost effective than vertical scaling in the long run due to flexibility

### Vertical Scaling

-   Adding more power (CPU, RAM, storage) to existing machines
-   Limited to the capacity of machines
-   Limited fault tolerance since the number of nodes remain the same
-   Suitable for moderate scalability requirements
    -   If the application isn't expected to experience rapid growth in traffic and resource demands
    -   Simpler and requires fewer changes

### Serverless

-   If the traffic fluctuates a lot, serverless is a good option
    -   Unpredictable traffic patterns or infrequent bursts of activity
-   Cost-effective because only the actual used resources are charged
    -   Beneficial when the service is idle
-   Platforms like AWS Lambda, Azure Functions handle the underlying infrastructure reducing operational overhead

## Availability

-   Measure of time that a system, service or machine remains operational
-   It takes into account maintainability, repair, spares, and other logistics time
-   Strategies to achieve high availability
    -   Redundancy
    -   Load balancing
    -   Failover mechanisms
    -   Monitoring and alerting
    -   Performance optimization
    -   Disaster recovery plan

## Reliability

-   Capability of a system to deliver its services when some components fail
    -   If some item is added to cart, it shouldn't be lost
    -   If something fails, another cart with same items should replace it
-   Reliability is 'availability over time' considering all conditions that can occur
    -   If a system is reliable, it is available but vice-versa may not be true
-   Achieved by Redundancy and Availability factors
-   Avoid single point of failures
-   Measured by mean time between failures and mean time to repair

## Serviceability (or Manageability)

-   How easy it is to operate and maintain
-   Simplicity and speed with which system can be repaired
-   Ease of diagnosing and understanding problems, making updates or modifications
-   It affects availability if fixing a failure takes long time

## Fault Tolerance

-   Can a system continue to function even in the presence of faults
-   How quickly the system can detect & handle failures and become operational again
-   Faults are the errors that arise in a component, it may not be failure of the system
-   Failure is the state when the system is not able to perform and provide services as expected

## Redundancy

-   Having duplicate resources or backups
    -   To make sure that the system keeps working even if something breaks
    -   Duplication of critical data or services to increase reliability of the system
    -   It can remove single point of failure and provide backups
    -   Load balancing ensures that everything works efficiently and reliably
-   Example: Extra servers, database replicas, RAID, geographic servers or data centers
-   Types
    -   Active: Multiple resources doing the same job at the same time
        -   If one of them fails, the others take over to keep running smoothly
    -   Passive: Backup resources that don't peform any job until required
        -   They stay in background, ready to jump in whenever a problem occurs
-   Testing
    -   Redundancy: Simulating failures to verify that redundant components function correctly
    -   Validation: Ensuring that data synchronization and consistency are maintained
    -   Load: Assessing how the system performs under heavy loads to identify bottlenecks
-   Shared-nothing architecture
    -   Each node can operate independently, there shouldn't be any central service managing activities
    -   New servers can be added without conditions and there is no single point of failure

## Modularity

-   Dividing a complex system into smaller independent modules
    -   That can be developed and tested individually and then integrated into the overall system
    -   Improves flexibility and reliability of the system
-   Each module is designed to perform a specific function and is self-contained
    -   With well defined interfaces to other modules
-   Allows different teams to work on different modules concurrently
-   Interfaces
    -   Set of rules or guidelines that define how different components of a system interact with each other
    -   Specifies inputs, outputs, and behaviors of a component
