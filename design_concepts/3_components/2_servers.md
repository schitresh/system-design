## Web Server
- Hosts websites and serves requests from clients
- Types: shared host, dedicated host, custom host
- Other uses
  - Processing data sent from another server
  - Hosting hypervisors (programs that manage virtual machines)
  - Distributing and monitoring software updates

### Working
- Web client sends a request to the web server via HTTP
- If the data is present for the request in the static database within the web server
  - Then immediately the corresponding response is sent back
- Else the web server sends a servlet request for processing to the application server
  - The application server looks up the application datastore to run the servlet and fetch the required info
  - And sends back the servlet response to the web server

## Application Server
- Provides environment and computational power to run specific applications
- Required when intensee computation is required which web server cannot handle

## Proxy Server
- Intermediary hardware or software sitting between a client and a server
  - That filters, logs & transforms requests
  - E.g. adding/removing headers, encrypting/decrypting, compressing a resource
- Proxies can sit on the client's local server side
  - And also between the client and remote servers
- Acts as an intermediary
  - For requests from clients seeking resources from other servers

### Features
- Filtering
  - Look addresses in its database of allowed or disallowed sites
  - Filter out content or websites based on different policies
  - Blocks malicious content and filter out dangerous websites
  - Prevent unauthorized access to sensitive resources
- Logging: Transform requests using encryption, compression, etc.
- Caching: Cache frequently used pages, can serve a lot of requests
- Batching
  - Batch multiple client requests for the same URI
  - To be processed as one request to the backend server
- Coordinating requests from multiple servers
  - And optimize request traffic from a system-wide perspective
- Collapsed Forwarding
  - Collapse requests for similar data or data that is spatially close together in storage
  - It minimizes the data reads and returns single result
- Security
  - Provide network address translation (NAT)
    - Which makes individual users on the network anonymous
  - Makes it harder for hackers to access individual computers on the network
- Though it can add latency due to all these extra features

## Forward Proxy
- Sits in front of one or more client computers
  - And serves as a conduit between clients and internet
- Stores and forwards internet services like DNS or web pages
  - For users within a networking group
- Working
  - Receives request from the client machine
  - And sends it to the internet on the client's behalf
  - On receiving a response from the internet, it forwards it to the client
- Usage
  - To reduce and control the bandwidth used by the group
  - To hide the IP of clients from the internet or web servers
  - To block a specific website
- Clients -> Forward Proxy -> Internet -> Web Servers

## Reverse Proxy
- Sits in front of one or more web servers
  - And serves as a conduit between web servers and internet
- Working
  - Typically sits behind a firewall in a private network
  - Directs client requests to an appropriate backend server
  - Provides an additional level of abstraction and control
  - Ensures a smooth flow of network traffic between the clients & the servers
- Used for
  - Traffic control and caching server response
  - Reduces DDoS attacks because only proxy server is accessible to the outer world
  - Provides SSL encyption that reduces the strain on web server
- Clients -> Internet -> Reverse Proxy -> Web Servers
