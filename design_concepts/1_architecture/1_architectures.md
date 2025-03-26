## Monolithic Systems
- The system where the whole application is deployed as a single unit
- Benefits
  - Easy to develop, deploy and test, ideal for small organizations
  - Easy to rollback changes and debug
- Challenges
  - Any changes requires complete redeployment
  - Less scalable since each element can have different scalability requirement
- Sections
  - Client Tier / User Layer / Presentation Layer / Front-end Layer
    - Takes input from users, interacts with server and displays the result to the user
  - Middle Tier / Service Layer
    - Includes the business logic
    - Receives requests from the client, acts on them and stores the data
  - Data Tier / Persistence Layer
    - Databases, message queues
    - Handles communication with other applications

## Microservices
- Small services that handle a specific part of the functionality and data
- They communicate with each other directly using APIs and lightweight protocols like HTTP
- Each mircoservice has its own database
- Easy to scale independently
- Other services continue to operate in case of failure or maintainence

## Distributed Systems
- Collection of multiple individual systems
  - Connected through network sharing resources to achieve a common goal
- Requests and data are shared across multiple servers
  - Increasing availability and decreasing latency
- Highly scalable and solves single point of failure
- Complex to manage data and hardware
  - In a network failure, conflicting information can be passed
- Race Conditions
  - Bugs that arise in systems due to timing mismatch
    - Of the execution order of multiple system services

## SOA (Service Oriented Architecture)
- SOA is an enterprise-wide concept
- It enables existing applications to be exposed over loosely-coupled interfaces
- Each interface corresponding to a business function
  - That enables applications in one part of an extended enterprise
  - To reuse functionality in other applications

## Event Driven Architecture
- System components communicate by generating, detecting, responding to events
- Events represent significant occurences like user actions or changes in system state
- When an event occurs, a message is sent to other components triggering an appropriate response
- Benefits
  - Rapid response to changing conditions and new information
  - Flexibility due to loose coupling of components
  - Decentralized communication reducing the need for point to point connections
  - Improved fault tolerance and scalability
- Challenges
  - Can become complex as the number of events and components grow
  - Maintaining the order of events and ensuring consistency
  - Overhead of event bus
  - Debugging and tracing events can be challenging

### Events
- Triggered by occurrences like user actions, changes in data, external stimuli, system processes
- Represented as messages or signals that convey information about an occurrence
- Asynchronous communication allowing for parallel processing
- Typically handled using pub-sub model
- Includes event type and payload to provide context and details about an event
- Event broker or event bus handles distribution, filtering and routing of events
  - Aggregator may be used to combine related events into a single meaningful event
  - Dispatcher may be used to route events to appropriate event handles and manage flow
  - Filters and rules engine to determine which subscribers should receive them
