# Web Crawler

-   Also known as web spiders, robots, worms, walkers, bots
-   Browses the world wide web (WWW) in a methodical and automated manner
    -   Collects documents by recursively fetching links from a set of starting pages
-   Many sites, particularly search engines, use web crawling
    -   As a means of providing up-to-date data
    -   Search engines download all the pages to create an index on them to perform faster searches
-   Other uses
    -   Test web pages and links for valid syntax and structure
    -   Monitor sites for changes in their structure or contents
    -   Maintain mirror sites for popular sites
    -   Search for copyright infringements
    -   Build a special-purpose index
        -   E.g. for understanding of the content stored in multimedia files on the web

## Requirements

-   Crawl only HTML over HTTP
-   Scalability
    -   Can crawl the entire web
    -   Can fetch hundreds of millions of web documents
-   Extensibility
    -   Modular design with the expectation that new functionality will be added
    -   Newer document types can be added later to be downloaded and processed

## Estimation

-   Crawling target: 15 B/month = 6200 pages/sec
-   Row Size: 100.5 KB
    -   Page sizes vary a lot, but we will be dealing with html text only
    -   Assume page size of 100 KB
    -   With each page we're storing 500 bytes of metadata
-   Storage
    -   Required Storage: 100.5 KB \* 15 B = 1.5 Petabytes
    -   Total Storage: 1.5/0.7 = 2.14 Petabytes
        -   Assuming 70% Capacity Model
        -   We don't want to go above 70% of the total capacity of the storage

## Considerations

-   Crawling the web is a complex task, and there are many ways to go about it
-   Decide the objective because it can change the design
    -   Is it a crawler for HTML pages only?
        -   It should be extensible
        -   It should be easy to add support for new media types later
    -   Is it a general-purpose crawler to download different media types?
        -   We should break down the parsing module into different sets of modules
            -   One for HTML, another for images, and so on for videos & music
        -   Each module extracts what is considered interesting for that media type
-   Decide the protocols that the crawler should handle
    -   HTTP, FTP, etc.
    -   Should be extensible
-   Expected number of pages it will crawl
    -   How big will the url database become?
    -   Let's assume we need to crawl 1 B websites
    -   A website can contain many other urls
    -   Let's assume an upper bound 15 B different web pages that will be reached
-   Robots Exclusion Protocol
    -   Allows webmasters to declare parts of their sites off limits to crawlers
    -   Requires a web crawler to fetch robot.txt containing these declarations
        -   From a website before downloading any real content

# Crawling Workflow

-   The basic algorithm executed by any web crawler
    -   Is to take a list of seed urls as its input
    -   And repeatedly execute the given process

## Process

-   Pick a URL from the unvisited URL list
-   Determine the IP Address of its hostname
-   Establish a connection to the host to download the corresponding document
-   Parse the document contents to look for new URLs
-   Add the new URLs to the list of unvisited URLs
-   Process the downloaded document, e.g. store it or index its contents
-   Repeat

## Breadth-first or Depth-first

-   BFS is usually used
    -   Will cover more ground though the given set of links
    -   DFS may get stuck in nested links
-   DFS is also used in some situations
    -   If the crawler has already established a connection with the website
    -   It might just DFS all the URLs within this website
    -   To save some handshaking overhead

## Path-ascending crawling

-   Helps discover a lot of isolated resources
    -   Or resources for which inbound link won't be found in regular crawling
-   Crawler would ascend to every path in each URL that it intends to crawl
    -   E.g. given a seed URL of http://foo.com/a/b/page.html
    -   It will attempt to crawl /a/b/, /a/, and /

## Challenges

-   Large volume of web pages
    -   Web crawler can only download a fraction of the web pages at any time
    -   Crawler should be intelligent enough to prioritize download
-   Rate of change on web pages
    -   Web pages on the internet change very frequently
    -   By the time the last page is downloaded from a website
        -   The pages may change or a new page may be added

# Crawler Structure

-   A bare minimum crawler needs at least these components
-   URL Frontier: To store the list of URLs to download and prioritize them
-   HTTP Fetcher: To retrieve a web page from the server
-   Extractor: To extract links from HTML documents
-   Duplicate Eliminator: To make sure the same content is not extracted twice
-   Datastore: To store retrieved pages, URLs, and metadata

## Working

-   Let's assume the crawler is running on one server
    -   And the crawling is done by multiple working threads
-   Each thread performs all the steps required to download & process documents in loop

### Working Thread

-   Remove an absolute URL from the shared URL frontier for downloading
    -   An absolute URL begins with a scheme like HTTP
    -   Which identifies the network protocol that should be used to download it
    -   Based on the scheme, the worker calls the appropriate protocol module to download it
-   After Downloading
    -   Downloaded document is placed into a Document Input Stream (DIS)
    -   It enables other modules to re-read the document multiple times
-   After document has been written to DIS
    -   The worker thread invokes a dedupe test to determine whether this document
        -   Has been seen before associated with a different URL
    -   If so, document is not processed further
-   Repeat the process

### Document Processing

-   Based on the downloaded document's MIME type (like HTML, image, video, etc.)
    -   Invoke the process method of each processing module associated with that MIME type
    -   Implement the protocols and the MIME types in a modular way for extensibility
-   HTML processing module will extract all links from the page
    -   Each link is converted into an absolute URL
    -   The links are filtered using user-supplied URL filter
    -   Then URL-seen test is performed to see if it is already in frontier or downloaded
    -   If the url is new, it is added to the frontier

## URL Frontier

-   Stores list of URLs to download and prioritizes them
-   We can crawl by performing breadth first traversal of the web
    -   Starting from the pages in the seed set
    -   Such traversals are easily implemented using a FIFO queue
-   The URL list will be huge, so distribute the frontier into multiple servers
    -   Each server can have multiple worker threads performing crawling tasks
    -   A hash function will map each URL to a server which will be responsible for crawling it

### Politeness Requirements

-   Politeness requirements must be kept in mind while designing a distributed URL frontier
    -   Crawler should not overload a server by downloading a lot of pages from it
    -   We should not have multiple machines connecting a web server
-   To implement this politeness constraint
    -   Crawler can have a collection of distinct FIFO sub-queues on each server
        -   Each worker thread will have its separate sub-queue
        -   From which it can remove URLs for crawling
        -   This will avoid overloading the web server
    -   When a new URL needs to be added
        -   The sub-queue for it will be determined by the URL’s canonical hostname
        -   A hash function will map each hostname to a thread number
        -   Hence, at most one worker thread will download documents from a given server

### Storing URLs

-   There will be hundreds of millions of URLs, hence we need to store them on a disk
-   Queues can have separate buffers for enqueuing and dequeuing
-   Enqueue buffer, when filled, will be dumped to the disk
-   Dequeue buffer will keep a cache of URLs to be visited
    -   It can periodically read from disk to fill the buffer

## Fetcher

-   Downloads the document corresponding to a given URL
    -   Using the appropriate network protocol like HTTP
-   Web masters create robot.txt
    -   To make certain parts of their websites off limits for crawler
-   To avoid downloading robot.txt file on every request
    -   The HTTP protocol module can maintain a fixed-sized cache
    -   Mapping hostnames to their robot’s exclusion rules

## Document Input Stream

-   The crawler's design enables the same document to be processed by multiple processing modules
    -   To avoid downloading a document multiple times
    -   We can cache the document locally using an abstraction called Document Input Stream (DIS)
-   It is an input stream that caches the entire contents of the document
    -   It also provides methods to re-read the document
    -   It can cache small documents (< 64 KB) entirely in memory
    -   While larger documents can be temporarily written to a backing file
-   Each worker thread has an associated DIS, which it reuses from document to document
    -   After extracting a URL from the frontier
        -   The worker passes that URL to the relevant protocol module
    -   The protocol module initializes the DIS from a network connection
        -   To contain the document’s contents
    -   The worker then passes the DIS to all relevant processing modules

## Document Dedupe Test

-   Many documents on the web are available
    -   Under multiple different URLs or mirrored on various servers
-   To avoid downloading duplicate documents, we can perform a dedupe test
    -   Calculate a 64-bit checksum (using MD5 or SHA) of every processed document
        -   And store it in a database
    -   For every new document, we can compare its checksum to previously calculated ones
-   For 15 billion distinct web pages
    -   We would need: 15 B \* 8 bytes = 120 GB
    -   This can fit in a server’s memory
        -   But if we don't have enough memory available
        -   We can keep smaller LRU cache on each server
    -   Check for the checksum in the cache first and then the persistent storage
        -   If not present, add the new checksum to cache and the back storage

## URL Filters

-   Controls the set of URLs that are downloaded
-   Used to blacklist websites for the crawler to ignore them
-   Used to restrict URLs by domain, prefix, or protocol type
-   Before adding each URL to the frontier
    -   The worker thread consults the user-supplied URL filter

## Domain Name Resolution

-   Before contacting a web server
    -   The crawler must use DNS to map the web server's hostname to an IP address
-   DNS name resolution will be a big bottleneck given the amount of URLs
    -   To avoid repeated requests, cache DNS results by building our local DNS server

## URL Dedupe Test

-   While extracting links, we will encounter multiple links to the same document
    -   To avoid downloading duplicate documents
    -   We can perform a URL dedupe test before adding to the frontier
-   We can store all the URLs seen by the crawler in canonical form in a database
    -   To save space, we can store a fixed-sized checksum rather than the textual form of URL
    -   To reduce the number of operations on the database store
        -   We can keep an in-memory cache of popular URLs on each host shared by all threads
-   Storage estimation: 15 B \* 4 bytes = 60 GB
    -   Considering 15 B distinct URLs and 4 bytes for checksum

### Bloom Filters

-   Can we use bloom filters for deduping?
    -   Bloom filters are a probabilistic data structure for set membership testing
    -   That may yield false positives
-   A large bit vector represents the set
    -   An element is added to the set by computing ‘n’ hash functions of the element
    -   And setting the corresponding bits
-   An element is deemed to be in the set
    -   If the bits at all ‘n’ of the element’s hash locations are set
    -   Hence, a document may incorrectly be deemed to be in the set
        -   But false negatives are not possible
-   Disadvantage: Each false positive will cause the URL not to be added to the frontier
    -   Hence the document will never be downloaded
    -   The chance of a false positive can be reduced by making the bit vector larger

## Checkpointing

-   A crawl of the entire web takes weeks to complete
-   To guard against failures, the crawler can write regular snapshots of its state to disk
-   An interrupted or aborted crawl can easily be restarted from the latest checkpoint

# Scalability

## Fault tolerance

-   We should use consistent hashing for distribution among crawling servers
    -   It will not only help in replacing a dead host
    -   But also help in distributing load among crawling servers
-   All the crawling servers will be performing regular checkpointing
    -   And store their FIFO queues to disks
-   If a server goes down, we can replace it
    -   Meanwhile, consistent hashing will shift the load to other servers

## Data Partitioning

-   Our crawler will be dealing with three kinds of data
    -   URLs to visit
    -   URL checksums for dedupe
    -   Document checksums for dedupe
-   Since we are distributing URLs based on the hostnames
    -   We can store all of this data on the same host
    -   Each host will store its set of URLs
        -   That needs to be visited
        -   Checksums of previously visited URLs
        -   Checksums of all the downloaded documents
-   Since we are using consistent hashing, URLs will be redistributed from overloaded hosts
-   Each host will perform checkpointing periodically
    -   And dump a snapshot of all the data onto a remote server
    -   This will ensure that if a server dies, another can replace it using the last snapshot

## Crawler Traps

-   There are many crawler traps, spam sites, and cloaked content
-   A crawler trap is a URL or set of URLs that cause a crawler to crawl indefinitely
    -   Some are unintentional
        -   Like a symbolic link within a file system can create a cycle
    -   Others are intentional
        -   Like written traps that dynamically generate an infinite web of documents
-   Solutions
    -   Anti-spam traps are designed to catch crawlers
        -   Used by spammers looking for email addresses
    -   Other sites use traps to catch search engine crawlers
        -   Used to boost their search ratings

### Adaptive Online Page Importance Computation

-   AOPIC algorithm can help mitigate common types of bot-traps
-   AOPIC solves this problem by using a credit system
    -   Start with a set of N seed pages
    -   Before crawling starts, allocate a fixed X amount of credit to each page
-   Process the pages
    -   Select a page P with the highest amount of credit
        -   Or select a random page if all pages have the same credit
    -   Crawl page P (assume the credit of 100)
        -   Extract all the links from page P (let's say there are 10 links)
        -   Set the credits of P to 0
        -   Take a 10% tax and allocate it to a Lambda page
    -   Allocate equal amount of credits to each link found on page P
        -   Distribute credits from P subtracting the tax
        -   (100 credits from P - 10% tax) / 10 links = 9 credits per link
    -   Select another page

### AOPIC Explaination

-   Since the Lambda page continuously collects tax
    -   Eventually it will have the largest amount of credit, and we’ll have to crawl it
-   By crawling the Lambda page, we just take its credits
    -   And distribute them equally to all the pages in our database
-   Since bot traps only give internal links credits
    -   And they rarely get credit from the outside
    -   They will continually leak credits (from taxation) to the Lambda page
-   The Lambda page will distribute those credits out to all the pages in the database evenly
    -   Upon each cycle, the bot trap page will lose more and more credits
    -   Until it has so little credits that it almost never gets crawled again
-   This will not happen with good pages
    -   Because they often get credits from backlinks found on other pages
