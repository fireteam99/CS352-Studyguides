# CS352 Midterm 2 Study Guide

## Ordered Delivery
### Reordering
* Reordering is a common occurance
	* Re-sending dropped packets
	* Individual packets may take different routes
*  Solution is to keep a buffer on the reciever side (socket buffer)

### Apps and TCP
* Applications can only read up to the last contiguous piece of data in the buffer
* This can cause delays

## Flow Control
### Limits of buffering
* What if the recievers rate of reading data is slower than the rate of data being pushed in
* This read rate may vary over time
* Data sent to a full buffer is dropped
* The amount of free buffer space availible known as
	* Receiver window size
	* Advertised window size
* Flow control means the sender's window size is bounded by the advertised window size

### Sizing the Receiver Socket Buffer  
* Must maintain a balance
	* Too small - sender can't keep many packets in flight
	* Too large - too much memory is consumed
* Data tends to be sent in bursts
* The buffer should be the size of the largest possible burst

### Bandwith-delay Product
* The maximum amount of in-flight data the sender can have
* Bandwidth between sender and receiver multiplied by RTT
* Example:
	* Sender sends data at `1 MBit/s`
	* RTT = `100 ms`
	* Maximum burtst = `1 Mbit/s * 100ms` = `100Kbit`

## Congestion Control
### Sharing the network
* There are many variables and little information on the network at any given time
* The Internet uses a distributed algorithm to converge to a **fair** and **effecient** outcome
* There is no central control system, each endpoint must try to reach a global good outcome
* Effeciency: if there is spare bandwidth, it should be used
* Fairness: If there are N endpoints sharing a bottleneck link they should get equitable shares of the capacity

### Flow vs Congestion Control
* Flow control tries to not overwhelm receiver
	* Constraint: receiver buffers
* Congestion control tries to not overwhelm routers
	* Constraint: bottleneck link capacity and bottleneck router's buffers 

## Feedback & Actions
* Signals
	* Dropped packets (RTO fires)
	* Delayed packets
	* Rate of incoming ACKS
* Knobs
	* Congestion window: amount of in-flight data per RTT
	* Increase congestion window
	* Decrease congestion window

### TCP Congestion Window (cwnd)
* An estimate of the in-flight data needed to keep pipe full and achieve self clocking
* Sending window = min(congestion window, receiver advertised window)

## Finding the Right Window
### TCP Slow Start
* `cwnd` starts at 1 MSS but is doubled every RTT
* On a loss we restart the cwnd at 1 MSS
* Issues with slow start
	* Congestion window increases too rapidly
	* Congestion window decreases too rapidly

### TCP additive increase
* Slow start until nearing correct cwnd
* Correct cwnd can be predicted using the last good cwnd without packet drop
	* Called slow start threshold (ssthresh)
* Once we hit the ssthresh transition from slow start to additive increase
	* For each ACK received, cwnd = cwnd + (MSS * MSS) /cwnd
	* One MSS increase for each cwnd worth data ACK'ed
* Upon a TCP timeout
	* Reset cwnd to 1 MSS
	* Set the sshresh to max(2 * MSS, 0.5 * cwnd)
	* The next linear increase will start at half the current cwnd
* Problems with Additive Increase
	* `cwnd` still drops to 1 MSS upon timeout
	* Maybe we could detect loss earlier

## Detecting and Reacting to Loss
### Duplicate ACKs
* Successive cumulative ACKs that contain the same ACK number
	* Also called duplicate ACKs
	* Strong indication that the requested sequence number was lost
	* Reduce the cwnd when we see more than 3 duplicate ACKs

### TCP Fast Retransmit
* Halve the `cwnd` (multiplicative decrease)
* The sequence number of duplicated ACKs are immediately retransmitted
* Sender keeps reduced cwnd until a new ACK arrives

### Additive Increase Multiplicative Decrease (AIMD)
* Convergest to fairness
* Converges to efficiency
* Increments grow smaller as fairness increases 

### Calculating TCP Timeout
* Idea is to use the average observed RTT (sampleRTT) as the RTO
* RTT can vary significantly over time
* EstimatedRTT = (1- a) * EstimatedRTT + a * SampleRTT
* Note that RTT can have a large variance so there needs to be a safety buffer
* Timeout = estimatedRTT + safety
* DevRTT = (1-b) * DevRTT + b * |SampleRTT - EstimatedRTT|
	* (typically, b = 0.25)
* TimeoutInterval = EstimatedRTT + 4 * DevRTT
	* 4 * DevRTT = Safety Margin 	

### Managing a Single Timer
* Data recieved from app
	* Create a segment with sequence number
	* Sequence number is byte-stream number of first data byte in segment
	* Start timer if not already running
		* Timer is for oldest unacked segment
		* Expiration interval for timer is the time out interval
* On timeout:
	* retransmit segment that caused timeout
	* restart timer
* On receiving ACK:
	* If ack acknowledges previously unacked segments
		* Update what still needs to be ACKed
		* Restart timer if there are any unacked segments  

### Retransmission Ambiguity
* It's difficult to get an accurate RTT measurement on retransfers
* We could never update RTT measurements for retramsmitted packets but that could cause our updates to be very late

### Karn's Algorithm
* Whenever there is a packet loss, RTO is increased by a factor
* We use this increased RTO as the estimate for the next segment set out
* When receiving an ACK for a successful transmission, we use the RTT from the new estimated RTT

## Connection Management
### 2-way Handshakes
* Client makes a request to the server
* Server replies
* Can result in half opened connection when the initial client request is dropped and retransmitted

### 3-way Handshakes
* Client makes request to server (LISTEN)
* Server replies with ACK (SYN RECVD)
* Client ACKs to confirm that it is ready (ESTAB)

### Closing a connection
* Client and server must close their side of the connection
	* FIN bit = 1 	
* TCP is full duplex which means both sides can send
* FIN is unidirectional and only stops one side of the communication
* It is neccessary to respond to a recieved FIN with ACK
* Steps:
	1. Connection is active (ESTAB)
	2. One side sends FIN (CLOSE_WAIT)
	3. Otherside acks FIN and sends respective FIN (LAST_ACK)
	4. Connection is closed (CLOSED )

## The Network Layer
* Runs on every router
* Moves data from sending to receiving endpoint
* Encapsulates transport segments into datagrams

### Two Key Functions
* Forwarding: moves packets from router's input to approriate output
	* Getting through a single interchange
* Routing: determine the route taken by packets from source to destination
	* Getting from address A to address B  

## IP: Addressing
### Addresses
* Helps implement effecient routing and forwarding

### IPv4 Addresses
* 32 bits long
* Identifier for host, router internface
	* Corresponds for point of attatchment
	* Not an identifier for endpoint
* Written in decimal in MSB order, seperated by dots
* 128.195.1.80 stands for: 10000000 11000011 00000001 01010000

### Types
* Unicast: destination is a single host
* Multicast: destination is a group of hosts
* Broadcast: 255.255.255.255 - destination is all hosts

### Classes
* Class A:
	* 16 million hosts allowed
* Class B:
	* 65 thousand hosts allowed
* Class C:
	* 255 hosts allowed
* Class D
	* Multicast addresses, no network/host hierarchy  

### Scaling
* IP addresses classing based on prefix length
* IP addresses witht he same prefix take identical paths from a far-away remote endpoint
* This reduces the amount of information needed to route packets in the Internet
* Also helps in delegating prefixes from ISPs to customers

### Problems with Class-based Routing
* Many small networks requesting multiple class C
* Running out of class B but not enough class A usage
* Lack of diversity in network sizes

## CIDR
* Classless InterDomain Routing
* Allows for subnet portion of arbitrary length
* Format of: a.b.c.d/x where x is the number of bits in the subnet portion
* Note the subnet portion is on the left and the host part is on the right
* An ISP can obtain a block of addresses and partition it to its customers

### Subnet Masks
* Determine if another IP address is on the same subnet or network
* Assume addresses A and B share subnet mask M
	* Compute logical AND (A & M)
	* Compute logical AND (B & M)
	* If (A & M) == (B & M) then A and B are on the same subnet 

## What's in a Router?
### Basic Components
* Control plane: per route-change processing (~ a few seconds)
* Data plane: per-packet processing (~ tens of nanonseconds)

### Input Port Functions
* Physical layer --> Data Link Layer --> Switching
* Switching:
	* Looking at the header field values lookup output port using forwarding table
	* Goal is the complete impot port processing at line speed
	* If datagrams arrive faster than they can be set there is queueing

### Switching Fabrics
* Memory
* Bus
* Crossbar

### Output Ports
* Buffering occurs when datagrams arrive from fabric faster than output port rate
	* Buffer management policy decides which packets to keep and drop if buffers are filled up
* Scheduling discipline chooses among queued datagrams for transmission 

## Prefixes and IP lookup
### Lookup
* We want to prioritize the IP with the longest prefix length among all prefixes that match the distination address
* CIDR greatly reduces the routing table size

## The Internet Protocol
### IP Datagram Format
* Total of 32 bits
* Overhead with TCP = 20 bytes of TCP + 20 bytes of IP = 40 bytes + app layer overhead

### IP Fragmentation & Reassembly
* MTU defines the max transfer size or largest possible link-level frame
* Large IP datagrams are divided when set through the net
	* One datagram --> several datagrams
	* Reassembled at the destination, at the IP layer
	* IP header bits are used to order the fragments
* Frag flat and offset are used for reassembly

## Dynamic Host Configuration (DCHP)
* How does a host get an IP address?
	* Hardcoded in a file
	* DHCP: dynamically get one from a server
* A host also needs to know:
	* Local DNS server
	* Subnet mask
	* Gateway routers 
* The DHCP server will "lease" an address to the host

### Client - Server Scenario
* Arriving client broadcasts (DHCP discover)
* DHCP server offers an IP address (DHCP offer)
* Client accepts the offer (DHCP request)
* DHCP server confirms the client accepted (DHCP ACK)

## Internet Control Message Protocol (ICMP)
* Protocol for error detection and reporting
* ICMP messages are delivered in IP packets
* Assists with network troubleshooting
* Specific uses:
	* Echo request reply
	* Destination unreachable
	* TTL expired 

### Ping
* Uses ICMP echo request/reply
* If response comes back can calculate RTT
* If no reponse then destination is unreachable

### Traceroute
* Records the route that packets take
* Uses the TTL field as each router jump, it gets decremented
	* Idea is to start TTL at 1 and increment it each time to successively visit the next router 

## Network Address Translation (NAT)
* All datagrams leaving a local network have the same source IP address but with different port numbers
* WAN side addresses are outward facing towards the rest of the internet
* LAN side addresses are inward facing to the local network
* Can change addresses of devices in local network without notifying outside world
* Can change ISP without changing addresses of devices in local network
* Devices inside local network are not explicitly addressable from the outside world - unless device inside connects first
* 16-bit port number field allows for 60,000 simultaneous connections with a single LAN-side address
* NAT is controversion - violates "end-to-end argument"

## IPv6
### Recent Developments
* We are running out of IPv4 addresses
* IPv6 has 128-bit addresses which allows for 3.4 * 10^38 unique addresses
* Fixed length headers (40 bytes)

### Datagram Format
* Priority
* Flow label
* Next header

### Other changes from IPv4
* Removed checksum
* Othions are allowed outside of the header
* ICMPv6 is a new version of ICMP with additional message types

### Flows
* A number in the IPv6 header that is used by routers to see which packets belong to the same stream
* Supports real-time service

### Adoption
* Adoption is very slow

## Routing Protocols
### Network-layer Functions
* Data plane
	* Forwarding: move packets from router's input to output
* Control Plane
	* Routing: determine route taken by packets from source to destination
* Two approaches
	* Traditional: per-router control
	* Software defined networking: logically centralized control   

## Link State Algorithms
* A centralized algorthm where we have complete knowledge of the network
* Routers are nodes
* Edge weights represent the link cost
* Use Dijkstra's algorithm to find the shortest path between two nodes (routers)

## Distance Vector Protocols
* The topology of the network is unknown
* Each router only know the distance to each one of its neighbors (distance vector) and its neighbors' distance vectors
* We can use the Bellman-Ford equation to find the least cost path from x to y
* Let the path be the minimum of the distances of going from x to each neighbor v + the cost from v to destination y
* Good news travels quick, but bad news travels slowly
* Poisoned reverse helps fix the count to infinity problem

## Scaling Routing to the Internet
### Scaling Routing
* The internet splits routers into regions called "autonomous systems" or "domains"
* intra-AS routing referes to routing to routers in the same domain
* inter-AS routing referes to routing between domains
* routers in the same domain must use the same protocol

### Routing Protocols
* Link state protocols (intra-AS routing)
* Distance vector protocols (intra-AS routing)
* Path vector protocols (inter-AS routing)

## Border Gateway Protocol
* De factor inter-domain routing protocol
* eBGP: obtain subnet reachability information from neighboring ASes
* iBGO: propagate reachability information to all AS-internal routers
* Determines "good" routes to other networks beased on reachability and policy

