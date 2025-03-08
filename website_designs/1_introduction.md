## Quick Tips
- Explain 80% of the time and ask interviewer 20% of the time
  - Engage in constructive dialogue, solicit feedback, incorporate suggestions
- Clarify requirements to ensure clear understanding of the problem statement
  - This avoids making assumptions and aligns your design with their expectation
- Be prepared for 'why' questions on using specific technologies
  - Interviewer may ask for more details and justifications
  - Discuss trade-offs and alternatives
- Don't go into detail prematurely since there is strict timeframe
  - Wait for interviewer's feedback or response on what needs to be discussed
- Requirements may change during the interview to test your flexibility
  - Don't try to fit the requirements somehow in a set architecture
- Be honest and don't try to fake
  - Find common solutions and show the willingness to learn

## Gather Requirements
- System design interviews are by nature vague or abstract
  - System design is open-ended and there is limited time in the interview
  - Ask questions about the exact scope of the problem and parts to focus on
  - Clarify functional requirements and ambiguties early in the interview
- Functional Requirements
  - Basic functionalities that the end user specifically demands
  - Need to be necessarily incorporated into the system as part of the contract
- Non-functional Requirements
  - Quality constraints that the system must satisfy
  - Priority or extent of these factors varies from one project to another
  - Factors: portability, maintainability, reliability, scalability, security
  - Examples: minimum latency of a request, system should be highly available
- Extended Requirements
  - Nice to have requirements that might be out of scope of the system

## Estimation & Constraints
- Clarify system's constraints and identify use cases
- Gather information about the problem at hand and design a solution
- Estimate the scale of the system we need to design
- Define what APIs are expected from the system
- Questions
  - What is the scale that the system need to handle?
  - What is the read & write ratio of the system?
  - How many users? How many of them are daily active? Dozens or millions?
  - How many requests per second?
  - How much storage is required?

## Abstract Design
- Outline all important components with constraints and use cases
  - Sketch main components and connections between them
  - First breadth then depth
- Data Model Design
  - Define database schema in early stages of interview to understand the data flow
  - Define all entities and relationships between them
- API Design
  - Design APIs to define the expectations from the system
  - Just a simple interface defining API requirements
    - Like parameters, functions, classes, types, entities, etc.

## High Level Design
- Roadmap for creating complex software systems
  - Outlines overall structure, components and interactions within the system
  - Represents flows and relationships between modules
  - Selection of components, platforms, and different tools
- Draft the first design of our system
  - Design monolithic or microservice architecture?
  - What type of database to use?
  - Identify system components required to solve our problems
    - Like load balancer, API gateway
- Capacity Estimation
  - Predicting the resources required to meet the expected workload
    - Processing power, memory, bandwidth
  - How the system can handle the current and the future demands efficiently

## Low Level Design
- Describe components and interactions from the high level design in more detail
  - Addressing specific algorithms, data structures, interfaces
  - Present different approaches, advantages, disadvantages
- How the components and entities are structured
  - How should we partition our data?
  - Load distribution
  - Should we use cache?
  - How to handle sudden spike in traffic
  - Object oriented principles, design patterns, solid principles
- Discuss with the interviewer which component may need further improvements
  - Demonstrate your experience in the area of your expertise
  - Explain your design decisions and back them up with examples

### Bottlenecks
- Identify bottlenecks and discuss approaches to mitigate them
  - Do we have enough database replicas?
  - Is there any single point of failure?
  - Is database sharding required?
  - How can we make the system more robust?
  - How to improve the availability of our cache?
  - Handling requests and load balancing
  - Is storage of huge data required?
    - Will database distribution on multiple machines solve it?
    - Downsides of db distribution: Is db slow? Does it need some in-memory caching?
  - Each solution is a trade-off
- Scaling
  - Knowing scalability principles and problems it solves
  - Discussing pros, cons and alternatives
  - Abstraction of layers to isolate each component
- Read engineering blog of the company to get a sense of their technology stack
  - And which problems are important to them
