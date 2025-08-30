# Network Protocols

-   Effective functioning of networks is essential
    -   For seamless communication and efficient data transmission
-   Network protocols are a set of rules and conventions that initiate the communication
    -   And data exchange between different devices in a network
-   They define the standards for data encoding, transmission, reception
    -   Ensuring that devices can understand & interpret each other's messages

### Importance

-   Interoperability
    -   They permit devices from different manufacturers
        -   And with various functionalities to communicate seamlessly
-   Optimizes data transmission minimizing latency and packet loss
-   Allow scalability of data traffic without compromising overall performance
-   Protocols like TCP ensure reliable & ordered data transmission
    -   Crucial for applications that require data integrity (file transfers, email communication)
-   Security protocols like SSL/TLS encrypt data during transmission
    -   Protect sensitive data from unauthorized access and ensure confidentiality of communications

## Internet Protocol (IP)

-   Responsible for addressing and routing packets throughout networks
-   Assigns a unique IP address to each device
-   Determines the best suitable path for data packets to reach their destination
-   Fundamental to all networked systems and is used with other protocols for data transmission

## Transmission Control Protocol (TCP)

-   Core protocol that operates at the transport layer of the OSI model
-   Ensures that data packets are delivered reliably and in order from the sender
-   Establishes a connection earlier than data transfer
    -   Ensures the receipt of packets and retransmits if there is any loss or corrupted data
-   Used where data integrity and sequencing are critical
    -   Web browsing, data transfers, e-mail delivery

## User Datagram Protocol (UDP)

-   Simple connectionless protocol that offers minimal services than TCP
-   Also referred as fire and forget protocol
-   Does not guarantee delivery or order of packets
-   Doesn't provide error checking or flow control mechanisms
-   Doesn't setup a connection before data transfer
-   Used in real-time applications where low latecy is important
    -   Like VoIP, online gaming, video streaming

## Domain Name System (DNS)

-   Translates user-friendly domain names (www.example.com) to numerical IP addresses (192.0.2.1)
-   Computers and network devices use only numerical addresses to locate one another on the internet
-   This removes the tedious need of remembering the addresses by users or systems

## HyperText Transfer Protocol (HTTP)

-   Used for transferring hypertext files on World Wide Web
-   Defines how messages are formatted and transmitted between different web servers & clients
-   Used in web browsers to retrieve and display web pages

### HTTP + SSL/TLS (HTTPS)

-   Secure Socket Layer / Transport Layer Security
-   SSL/TLS protocols provide stable and secure connection
    -   For information exchange between different devices on a computer network
-   They encrypt information during transmission

## File Transfer Protocol (FTP)

-   Used to transfer documents between a client and a server on a computer network
-   Allows users to upload, download, manipulate files on remote servers
-   Also used for website maintenance and software distribution

## SMTP and POP3/IMAP

-   SMTP: Simple Mail Transfer Protocol
    -   Used for sending email messages
    -   Transfers outgoing mail from a user to a server
-   POP3: Post Office Protocol, IMAP: Internet Message Access Protocol
    -   Used for receiving email messages
    -   Retrieve incoming mail from a server to a user
