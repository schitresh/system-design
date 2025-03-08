## Rate Limiters
- Used to limit the rate at which a system of application processes requests or performs actions
- Types
  - Request
    - Protects system from being overloaded by too many requests within a given time period
  - Action
    - Limits the number of times a specific action can be performed within a given time period
  - User
    - Prevents specific users from making excessive requests to undermine the performance of the system
    - E.g. Denial of service attacks
  - Token bucket
    - Limits the rate at which requests are processed by a system
    - Any excess requests are held in a bucket until the next time period

## Monitoring System
- Collects, analyzes and reports metrics and performane related data
- Helps to monitor if the desired service levels are being met
- Types
  - Network: routers, switches, servers
  - System: CPU, memory, disk usage
  - Application: web servers, databases
  - Infrastructure: virtual machines, containers

## Unique ID Generator
- Generates unique identifiers that are used to identify objects or entities in a distributed system
- Helpful to provide a stable identifer for a resource accessed over a network
- Can be done using centralized service, distributed consensus algorithm, timestamps

## Search Services
- Using multiple nodes or servers to index and search large datasets in a distributed system
- Allows parallel processing of search queries and distribution of data
- Approaches
  - Distributed Search Engine that scales horizontally across nodes like Elastic search, Apache Solr
  - Database with built-in search capabilities like Mongodb, Cassandra
  - Cloud based search services like AWS Elastic Search, Google Cloud Search

## Logging Services
- Collecting, storing and analyzing log data from multiple sources
- Used to track health & performance, and for debugging issues
- Approaches: Centralized, Distributed, Cloud based

## Task Scheduler
- Used to automate execution of tasks at regular intervals
  - On a specific schedule or in response to certain events
- Approaches
  - Standalone
    - Simple to implement, allows flexibility, complex to manage, requires infrastructure
  - Built-in task scheduler in container orchestration platforms or cloud-based serverless paltforms
    - Simple to implement and mangage, less flexible
  - Cloud based like AWS SNS (Simple Notification Service), Google Cloud Scheduler
    - Highly scalable and fault-tolerant

## API Gateway
- Simplifies the client interaction with the underlying services and enhances security
  - Routes requests to appropriate mircoservice or backend service
  - Provides versioning and ensures backward compatibility
  - Logging and monitoring, captures metadata like timestamps, client IPs, request headers
- Security
  - SSL (Secure Socket Layer) is used to establish an encrypted link between the server and the client
  - Handles SSL/TLS termination
  - Rate limiting and throttling to prevent one service from overloading
  - Rate limiting sets maximum number of requests per unit time from a client
  - Throttling delays requests beyond the defined rate limit
- Authentication
  - Authentication users using username & password, API keys, OAuth tokens, JWTs
  - Verifies that the user or application has the necessary permissions
- Serialization & De-serialization of data
  - Transforms requests and responsed as they pass through to ensure compatibility between services
  - For example, converting JSON to XML and vice versa
  - Can aggregate data from multiple services into a single response
- Filtering
  - Filter and sort the required resources
  - Default filters can be based on geographic factors
- Pagination
  - Break the large set of resources into pages of a given size
  - Makes it easy to query results and reduces load time
