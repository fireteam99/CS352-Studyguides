# CS352 Midterm 1 Study Guide

## Networks

### What are they?
* Carry Information betwen two or more entities
* Communicating entities are endpoints
* There are many physical mediums that can carry information
	* Copper wires
	* Fiber optic
	* Microwave
	* Cable
	* Satellite

### Single link multiple access networks
* Datas are sent in *packets* or *frames* 
* Hosts have addresses to differentiate them
* Who should speak? Medium access control problem
* Converting from digital to physical signals (encode/decode)


### Multilink Networks
* Multiple links connected by routers
* How to move packets from one host to the next? Routing problem
* Best effort delivery makes it simple to build networks
* How quickly should endpoints send data? Congestion control problem

### Guarantees
* The transport software is responsibile for implementing guarantees, not the network

### Components of a network
#### Link
* Communication links for transmission

#### Host
* Computer running applications of end user

#### Router
* Computer for routing packets from input line to another output line

#### Gateway
* A device directly connected to two or more possibly different networks (serves as an access point), provides access

#### Network
* A group of hosts, links, routers capable of sending packets among its
members

### Why are networks useful?
* Easy availability
* Performance & load sharing
* Reliability
* Human to human communication

### Web evolution
* The web is becoming increasingly complex
* Content is exploding
* There are massive impacts on the economy and people

## How machines talk

### Machine communication
* 1s and 0s are converted to analog signals using modulation

### Switching schemes
1. Circuit Switching
2. Message Switching
3. Packet Switching

### Circuit switching
1. Setup: Control message sets up path from origin to destination
2. Return signals indicates transmission can proceed
3. Transmission begins
4. The entire path remains allocated to the transmission (whether it is being used or not)
5. When the transmission is complete the source releases the circuit

![circuit-switching](https://almawhoob.net/wp-content/uploads/2016/07/circuit-sqitching_virtual_datagram.png)

### Message switching
* Header: metadata about message
* Message hops from node to node while allocating only one link at a time
* Analogy: Postal Service
* When the entire message arrives at a router it is sent to the next router
* If the next router is busy it his held in a queue

![message-switching](https://erg.abdn.ac.uk/users/gorry/course/images/message-setup.gif)

### Packet switching
* Messages are split into smaller pieces called packets
* Pipelining means multiple parts of the message are sent concurrently

### Store and forward
* Router waits for all message bits to arrive before sending the first bit to outgoing link
* The internet uses store and forward packet switching

### Cut through
* Bits are sent to the next node as soon as they arrive

### Comparisons
* Header overhead (how many bits are in metadata)
	* Circuit < Message < Packet
* Total Delay
	* Short Bursty Messages: Packet < Circuit
	* Long Continuous Messages: Circuit < Packet

### Measuring Networks
* Packet size: length of packet in bits or bytes, including the header
* Bandwidth: For a single link, the amount of data it can transmit per unit type
* Propagation delay: The time needed to move one bit across
	* Imposed by communication medium (link length)
* Transmission delay: Time from the first bit leaving the sender to the last bit leaving the sender
	* Determines by link bandwidth and packet size 
* Queueing delay: Time the packet is waiting for transmission
* Total packet delay: Time from first bit from sender to last bit recieved at reciever
	* propagation delay + queueing delay + transmission delay

### Conveyer belt analogy
* Propagation delay: The time takes for first box to travel the length of belt
* Bandwidth: the number of boxes put on belt per minute
* The shipment transmission time is the number of boxes in one shipment divided by the rate at which we can put boxes onto the belt
* Total transfer time is the transmission time plus propagation delay

## Protocols and Layering

### Protocols
* Message foormat
	* The structure of messages exchanged by endpoints
* Actions
	* The operations upon receiving or not recieving messages

### Layering
* Software and hardware for networking are arranged in layers which provides modularity
* Each layer has a distinct function and interacts with outher layers through interfaces
* Layers include:
	* Application layer: useful user-level functions
	* Transport layer: provides guarantees to apps
	* Network: nest effort global packet delivery
	* Link: best effort local packet delivery
* Generally endpoints have all four layers while routers only have network and link layers

## App-layer concepts

### Application-layer protocol
* Request and response messages are exchanged
* Message format contains rules on syntax and semanics
* Actions tell when and how processes send and respond to messages
* Public-domain protocols like HTTP and SMPT are dfined in RFCs which allow anyone to use them
* Some protocols are proprietary like Skype

### Application "addresses"
* We need ways to connect applications that are residing on different endpoints on a network
* Telephones use 10 digit numbers
* Computers use IP addresses
	* IPv4 has 32 bits
	* IPv6 has 128 bits
* We also need an identity for the application at the endpoint: port number
* Think of the IP address being the house and the port number being the person living there you want to contact
* A socket represents the connection between the OS/network and the application proccess
	* Think of it as an API to the network
* A app-layer connection is a 4-tuple consisting of (source IP, source Port, destination IP, destination Port)

### Application architectures
#### Client-server
* Server: an always on host that has a perminant IP address
* Cient: communicates with the serverm has a dynamic IP addressm and may be intermittently connected
* Note that clients do not communicate directly with each other 

#### P2P (Peer to Peer)  
* Peers: intermittently connected hosts that directly talk to each other
* There is little to no reliance on always-up servers
* Many applications use a hybrid model

### DNS
* It's hard for people to remember random numbers (IP addresses)
* Instead, we use alphanumeric names to refer to hosts and map them to IP addresses
* This proccess is called Address Resolution
* A really simple design would be a single server that looks up a table of domain names mapped to addresses
	* This is not scalable nor reliable
* A centralizzed DNS design is problematic
* Instead we use distributed and hierarchical databases to enable scaling

![distributed-dns](https://i1.wp.com/domain.powerhoster.com/wp-content/uploads/2016/11/dns-top.png?resize=640%2C480&ssl=1)  

### DNS protocol
* Client & Server model over Port 53 on server
* DNS server of IP address is already known
* Two types of messages
	* Queries
		* Standard 0x0
		* Updates 0x5
	* Responses
* Lets say a client wants the IP address of a host name
	1. Client sends NDS query to "local" namer server on network
	2. The local name server will return the address if it has it, otherwise will forward the request to root name server
	3. The requests works its way down the tree towards host until to reaches a name server iwth the correct mapping

### Query Types
#### Iterative
* The contacted server replies with the name of the next server to contact
* Queries are iterative for local DNS server

![iterative](https://www.omnisecu.com/images/tcpip/recursive-iterative-dns-query.jpg)

#### Recursive
* The burdern of the name resolution is put on each contacted name server
* The root DNS server must answer every query
	 	
![recursive](https://i.stack.imgur.com/00S6Q.jpg)

### DNS records
* Type=A
	* name is hostname
	* value is IP address
* Type=AAAA
	* name is hostname
	* value is IPv6 address
* Type=NS
	* name is the domain name
	* value is the hostname of the authoritative name server for the domain
* Type=CNAME
	* name is the alias for the canonical name
	* value is the canonical name
* Type=MX
	* value is associated mail server 

### Caching and updates
* Once a name server learns the name to IP address mapping it cachnes the mapping
* Entries will time out after a period of time
* Root servers aren't visited that much due to caching

## HTTP

### Overview
* Http follows the client server model
	* Client requests web objects
	* Server servers web objects

### Method Types
* GET
* POST 
* HEAD
* PUT 
* DELETE

### Non/Persistant HTTP
#### Non-Persistant
* At most one object is sent over a TCP connection

#### Persistant
* Multiple objects can be sent over a single TCP connection
* Used by default since HTTP/1.1

### HTTP Response time
* RTT defines the time it takes for a small packet to travel from client to server and back
* The sum of propagation and queueing delays
* The total response time includes one RTT to initiate TCP connection, one RTT for the HPP request and the first few bytes of the HTTP response to return, and the file transmission time
* Total = `2 * RTT + transmit time`
* Persistant HTTP is more effecient becase it does not require an additional RTTs to make the TCP connection when sending multiple objects

### Cookies
* HTTP is stateless by default (the server doesn't remmeber anything)
* Cookies rely on four components
	1. Cookie header line of HTTP response message
	2. Cookie header line in HTTP request message
	3. The cookie file is kept on the user endpoint, managed by their browser
	4. The back-end DB will map the cookie to user data at the web endpoint

### Web caches
* Caches reduce response time for client requests
* Caches reduce traffic on an access link
* Caches can be implemented in the form of a proxy server 
* Operation
	* Configure an HTTP proxy on computer's network settings
	* Browser will send all HTTP requests to proxy (cache)
	* On hit, the cache will return the object
	* On miss, the cache will fetch the object, store it, and return it
* Conditional GETS have a `last modified` field in its reponse header to make sure content is up to  date

### CDN (Content Distribution Networks)
* CDNs are a global network of web caches
* They are provisioned by ISPs or content providers
* Uses
	* Reduce bandwith requirements on content providers
	* Reduce cost of maintaining origin servers
	* Reduce traffic on a network's internet connection
	* Inproves response time for user requests
* CDN servers are spread out geographically to help reduce propagation delays

### Themes from HTTP
* Request/response nature of protocols
* ASCII based message strucutre
* Caching
* Scaling using indirection


## SMTP

### Three major components
1. User agents
	* Client facing app that reads the mail
	* Gmail, outlook, apple mail
2. Mail Servers
	* A mailbox contains incoming messages for a user
	* A message queue has outgoing mail emssages from a user
	* The sender mail server makes a connection to the reciever mail server at their IP address and port 25
3. SMPT Protocol
	* Used to send email messages
	* Client talks to user agents and mail server
	* Server recieves mail messages

### Mail message
* Header lines
	* To:
	* From:
	* Subject:
* Body
	* The message, comprised of ASCII characters only   

### Mail access protocols
* SMPT: deals with selivery/storage between mail servers
* Mail access protocols: deals with retrieval of mail from mail servers
	* POP: Post Office Protocol (extremely simple)
	* IMAP: Internet Mail Access Protocol (more features)
	* HTTP: Used by web-based emails

### POP vs IMAP
#### POP3
* Stateless server
* User agent heavy processing
* The user agent retrieves email from server which is then deleted from server
* Very simple protocol

#### IMAP4
* Stateful server
* User agent and server does processing
* Server can see folders and other organizational components availible to user agents
* Changes are visibile at the server
* Complex protocol

#### Web-based emails
* Browsers speak HTTP
* Email servers speak SMTP
* There needs to be a bridge to retrieve email using HTTP

### Themes from app-layer protocols 
* Separation of concerns
* In-band vs out-of-band control
* Keep it simple until you need complexity


## Transport Layer

### Overview
* The transport layer provides logical communication between app processes running on different hosts
	* The send side breaks app messages into segments and passes it to the network layer
	* The recv side reassembles those segments into messages and passes it to the app layer
* UDP and TCP are examples of transport protocols 

### Guarantees you want
* Reliablilty
* Performance
* Ordering

### UDP
* Unreliable, unordered delivery
* User Datagram Protocol
* A small extension to underlying network layer protocol

### TCP
* Reliable, in-order delivery
* Transmission Control Protocol
* Features include congestion control, flow control, and connection setup

### Services not availible
* delay guarantees
* bandwidth guarantees

### User Datagram Protocol
* UDP is best effort so segments may be lost or out of order
* UDP is connectionless so each segment is handled independently
* DNS uses UDP because of it's one-off req/resp nature
* Also good for loss-tolerant/delay-sensitive apps like video calling

### Demultiplexing
* Host recieves IP datagrams
	* Datagrams contain a transport level segment
	* Each segment has a source and destination IP address
	* They also have a source and destination port number 
* Host will use the IP address and port numbers to direct segments into the appropriate app-level socket
 
## Error Detection

### Data corruption
* Bits can get flipped
* Bits can get lost
* We can ditect errors by computer a function over the binary data
* The function bust by easy to compute

### UDP Checksum
#### Sender
* Treat segment contents as sequences of 16-bit integers
* Checksum is the 1's complement addition of segment contents
* That check sum is placed in the UDP checksum field

#### Reciever
* Computers checksum of recieved segment
* Check if comptuer checksum equals checksum field value
* Note that when adding numbers a carryout from the most significant bit needs to be added to the result

## Reliable Data Delivery (Stop & Wait)

### ACK (acknoledgement)
* The reciever sends an ACK back to the sender to notify them that they recieved the packet
* Ther reciever can send a negative ACK (NAK) if the data was corrupted
* TCP only uses positive ACKs

### RTO
* If no ACK is received in a certain amount of time (RTO), then the sender retransmits the packet
* Retransmission works works even if ACKs are lost or delayed

### Sequence Numbers
* In a worst case senario ACKs might be dropped or delayed so that the sender retransmits already recieved packets after the RTO
* Each packet needs to have a unique sequence number

### Stop and wait reliability
* The sender waits for an ACKRTO before sending another packet
* Called "stop and wait"

## Reliable Data Delivery (Pipelined Transmission)

### In-flight data
* Any unACKed data
* We want as many packets in flight as possible to improve throughput

### Reciever strategies upon packet loss
* Selective Repeat - if we ACK subsequent packets
	* Which sequence number are on the ACK?
		* Cumulative ACK - the last packet in order
		* Selective ACK - a range of the packets recieved so far
* Go-back-N - if we do not ACK subsequent packets

### Sliding Window with Go Back N
* If the sender doesn't detect a ACK for one of it's packets, it will write off all subsequent packets
* The server will retransmit all packets following the one that errored out even if they were received successfully

### Cumulative ACK
* The receiver will store the sequence numbers of all the packets that arrived successfully (requires a memory buffer)
* When the receiver notices a skipped sequence number it stores it as the last good sequence number (cumulative ACK)
* When the sender times out waiting for an ACK, it will retransmit the one unacknowledged frame
* Timeouts apply independently for each sequence number

### Sliding window
* The window size is the amount of in-flight data
* The window is the sequence of numbers that are currently in-flight
* Note that the window of the receiver is always ahead of the window of the sender

## Ordered Delivery

### Reordering at reciever side
* Packets can arrive out of order for many reasons
* The receiver needs to make sure that the data presented to the application is in the same order as the sender pushed it
* This is accomplished using a memory buffer at the reciever (reciever socket buffer)
	* This is already used to avoid duplciate transmissions
* An app using a TCP socket reads from the TCP recieve socket buffer
* The TCP reciever software only releases data if the data is in order to all other data already read by the application (you can only read up to the first unreceived packet) 
* This process is called TCP reassembly

### Implications of ordered delivery
* Packets can't be delived to the app if there is an in-order packet missing from the receiver's buffer
* It won't matter if additional packets successfully arrive if they are ordered after the missing packet
* Thhis could result in reduced thoughput


