## Efficiency
- Two standard measures of efficiency are
  - Latency (or response time)
  - Throughput (or bandwidth)
- The two measures correspond to these unit costs
  - Number of messages sent by the nodes of the system regardless of the message size
  - Size of messages representing the volume of data exchanges
- The complexity of operations supported by distributed data structures
  - Can be characterized as a function of one of these cost units
- The analysis of a distributed system in terms of number of messages is over simplistic
  - It ignores the impact of many aspects like network topology, network load
  - Heterogeneity of the software & hardware components
    - Involved in data processing & routing
  - However, it is quite difficult to devlop such a precise cost model

## Latency (or Response Time)
- Time taken for a data or signal to travel from one point to another in a system
- It encompasses delays like processing time, transmission time, response time
- Low latency is important in payments, transactions, gaming

### Factors affecting latency
- Physical distance
- Network congestion
- Inefficient network infrastructure
- Wireless interference
- Slow hardware
- Software inefficiency
- Database access: complex query, overloaded databases
- Resource competition: CPU, memory
- Data compression and encryption

### Measurement
- Ping: Send data packeets to target service and measure round trip time
- Traceroute: Analyse path taken by data packets, which network hops contribute the most to latency
- MTR (traceroute with ping)
- Insert timestamps at various points
- Network monitoring tools
- Performance profiling tools

## Throughput (or Bandwidth)
- The rate at which a system, process, or network
  - Can transfer data or perform operations in a given time period
- Difference from Latency
  - Latency: Time consumed for the transfer of one data packet
  - Throughput: Number of data packets arrived within a second
- Measurement in
  - Network: Amount of data transmitted in a given period
  - Disk: How quickly data can be read or written
  - Processing: Number of operations/instructions completed in a unit time
