# CS352 Final Study Guide

## Inter-Domain Routing

### Routing Protocols
* Intra-AS protocols (intra-domain routing)
	* Protocols used within a single AS must be consistant
	* Different AS's may use different intra protocols
	* Also called interior gateway protocols (IGP)
	* Includes:
		* Link state protocols
		* Distance vector protocols	 
* Inter-AS progocols (inter-domain routing)
	* Common accross ASes
	* Each AS knows little about others
	* Includes:
		* Path vector protocols

### Border Gateway Protocol (BGP)
* BGP sessions contain two BGP routers (peers) exchanging BGP messages over semi-perminanty TCP connection
* Advertise paths to different destination network prefixes

### Path attributes and BGP routes
* AS-Path: list of ASes through prefix advertisement is passed
* Next-Hop: indicates specific internal-AS router to next-hop AS
* Policy-based routing:
	* Import policy determins whether to accept/decline a path
	* Export policy determines whether to advertise a path to neighboring ASes

### Policies in BGP
* Policy comes from business relationships
	* Customer-provider relationships
	* Peer-peer relationships
* Internet-eXchange Points (IXPs)are large PoPs where ISPs connect to each other 

### BGP messages
* BGP messages are exchanged over TCP
* Messages include:
	* OPEN: opens and authenticates a connection
	* UPDATE: update availible paths
	* KEEPALIVE: keeps connection alives and ACKs OPEN request
	* NOTIFICATION: reports errors in previous message, or close connection

### BGP route selection process
* Router may learn about more than one route to destination AS
* Route selection is based on:
	1. Local preference value attribute (policy decisim)
	2. Shortest AS-PATH
	3. Closest NEXT-HOP router (hot potato routing or early-exit routing)
	4. Additional criteria 

### Intra vs Inter AS routing
* Policy:
	* Inter-AS: multiple administrators want control on how traffic is routed
	* Intra-AS: single admin
* Scale:
	* Hierarchical routing saves table size, and reduces update traffic
* Performance:
	* Intra-AS: focus on performance
	* Inter-AS: policy dominates over performance 
 
## Quality of Service

### Network support for applications
* Best effor internat architecture does not offer many garuntees
* Many applications require delay and loss bounds
	* VOIP, video chatting, streaming, remote surgery...
* How to provide quality of service (QOS) for apps
	* Provisione enough resources
	* Mechanisms to handle traffic based on importance

### One approach: "dimension"
* Deploy enough link capacity so there is no congestion
	* Low complexity of network mechanics
	* High bandwidth costs
* Challenges
	* Network dimensioning: how much is enough?
	* Estimating network traffic demand
* Can be exceptional cicumstances such as national events, emergencies... etc

### Multiple classes of service
* Partition traffic into classes
* There exist ToS bits in IP header but they are not currently used
* Three main principles
	1. Packets should be marked so that routers can treat them accordingly
		* Priority is given to some forms of traffic (VOIP) over others (HTTP) 
	2. Provide isolation from one class from another
		* Enforce bandwith allocations  
	3. Use resources as effeciently as possible - also known as work conservation

## Packet scheduling

### Why it matters
* Significantly influences how packets are treated regardless of endpoint transport
* Implications in net neutrality debtates

### Scheduling vs. Buffer Management
* Determines how packets should enter vs leave the switch buffer
* Goal is to have fair and effecient sharing of the buffer

### Packet scheduling for QoS
* How to choose between queued packets to send on an outgoing link?
* Possible options include:
	* First in first out (FIFO)
	* Simple multi-class priority
	* Round robin
	* Weighted fair queueing (WFQ)
	* Rate limiting
* All are work conserving except rate-limiting

### Isolation by Rate Limits
* Average rate: how many packets can be sent per unit time in the long term
* Peak rate: maximum number of packets that can be sent unit time
* Burst size: number of packets that are sent consecutively

### Shaping and Policings
* Policing enforces rate by dropping excess packets immediately
	* High loss rates
	* Does not require memory buffer
	* No RTT inflation
* Shaping enforces rate by queueing excess packets
	* Only drops packets when buffer is full
	* Requires memory to buffer packets
	* Can inflate RTTs due to high queueing delay

### Leaky Bucket Shaper
* Packets can be generated in a bursty manner
* Once passing through leaky bucket they enter the network evenly
* The "bucket" buffers packets until a certain point
* When the "bucket" "overflows", packets are dropped
* A type of traffic shaper, changes downstream arrival times and characteristics of passing packets
* Some issues include:
	* Average rate == peak rate
	* Many short transfers just have a few packets
	* It may be benefitial to allow short high volume bursts

### Token Bucket Shaper
* There is a bucket of tokens and a packet buffer for tokens
	* Packets are dropped if the buffer is full
	* A token bucket policer drops any packet that doesn't have a token
* Short flows can exceed rate limit r since they use the reserve tokens in a bucket
* Long flows will be rate limited once they exceed token usage

## Link Layer

### Introduction
* Hosts and routers are nodes
* Links for communication channels between nodes
	* Wired links
	* Wireless links
	* LANs
* The data-link layer has responsibility for transfering datagrams between two physically adjacent nodes

### Analogy
* Trip from Piscataway to Lausanne
	* Limo: Piscataway to JFK
	* Plane: JFK to Geneva
	* Train: Geneva to Lausanne
* Datagram: tourist
* Communication link: road/flight/rail 
* Link layer protocol: car/plane/train
* Routing algorithm: travel agent

### Application
* Implemented in every host
* More specifically in the "adaptor" also called network interface card
* Attatches to host's system buses
* The link layer is a combination of hardware, software, and firmware

### Adapter Communication
* Sending side
	* Encapsulate datagram in frame
	* Add error checking bits 
* Receiving side
	* Check for errors
	* Extract datagram and pass it to upper layer 

### Link Layer Services
* Encoding: bits to signals, and signals to bits
* Framing, link access: encapsulate datagram
* Reliable delivery between adjacent nodes
	* Seldom used in low bit error links 

### Encoding
* Signals propagate over physical medium
* Encoding binary data onto signals

### Error Detection
* Single bit partity: detect single bit errors
* Two-dimensional bit parity: detect and correct single bit errors

### Cyclic redundancy check
* More powerful error-detection coding
* View databits as a binary number --> D
* Choose a r+1 bit pattern generator --> G
* Choose r CRC bits R, such that <D, R> exactly divisible by G
* If there is a non-zero remained, an error is detected
* Can detect all burst errors less that r+1 bits
* Widely used in practice

## ARP

### ARP: Address Resolution Protocol
* Returns a link layer address when given an Internet address
* Communication requires IP --> MAC address translation


## Medium Access Control 	

### Multiple access  
* Two types of links:
	* Point to point
	* Broadcast

### Multiple access protocols
* Single shared broadcast channel
* Two or more simultaneous transmissions cause interference
* Multiple access protocol
	* Distributed algorithm that determines how nodes share a channel
	* Communication about channel sharing must use channel itself
	* Solves the Medium Access Control problem

### Goals of multiple access protocol
 * Given: broadcast channel of rate R bits per second
 * Goals:
 	1. When one node wants to transmi, it can send at rate R
 	2. When M nodes want to transmit, each can send at average rate R/M
 	3. Fully decentralized
 	4. Simple

### MAC protocols: Taxonomy
* Channel partitioning
	* Divide channel into smaller pieces (time slots, frequency, code)
	* Allocate piece to node for exclusive use
* Random access
	* Channel not divided, allow collisions
	* Recover from collisions 
* Taking turns 
	* Nodes take turns, but nodes with more to send can take more or longer turns

### Time Division multiple Access (TDMA)
* Access to channel in "rounds"
* Each station gets fixed length slot "packet transmission time)
* Unused slots go idle

### Frequency Division Multiple Access (FDMA)
* Channel spectrum divided into frequency bands
* Each station assigned fixed frequency band
* Unused transmission time in frequency bands go idle

### Random Access Protocols
* When a node has a backet to send it sends it immediatly
* Collisions can occur when two or more transmitting nodes 
* The random access MAC protocol specifies:
	* How to detect collisions
	* How to recover from collisions

### Slotted ALOHA
* Assumptions:
	* All frames are the same size
	* Time is divided into equal size slots
	* Nodes start to transmit only after slot begins
	* Nodes are synchronized 
	* If two or more nodes transmit in slot all nodes detect collission
* Operation:
	* When node obtains fresh frame, it transmits the next slot
		* If no collision, node can send new frame
		* If collision, node retransmits frame in each subsequent slot with probability `p` until success  
* Pros:
	* Single active node continuously transmits at full rate of channel
	* Highly decentralized
* Cons:
	* Collisions
	* Idle slots
	* Clock synchronization
	* Ensure detect collision within a frame, even if detection is fast the whole frame time is wasted
* Effeciency: 37%  

### Pure (unslotted) ALOHA
* No synchronization
* When first frame arrives transmit immediatly
* Collision probability increases
* Effeciency: 18%

### CSMA (carrier sense multiple access)
* Idea is to listen before transmit
* If the channel is sensed idle, transmit entire frame
* If channel is sensed busy, defer transmission
* Effeciency: aproaches 100%

### CSMA/CD (collision detection)
* Collisions may still occur which waste transmission time
* Easy to detect in wired LANS, but difficult to detect in wireless LANs

### Taking Turns
* Looks for effeciency at both low and high loads
* Polling: orchestrator node "invites" sender nodes to transmit in turns
* Token Passing: passes a control token from one node to the next sequentially

## Wireless Lan

### Elements of a Wireless Network
* Wireless hosts
	* Laptops, smartphones
	* May be stationary or mobile
* Base station
	* Typically connected to wired network
	* Acts as a relay, sending packets between wired network and wireless hosts
* Wireless link
	* Connects mobile devices to base station
	* Used as backbone link

### Wireless Link Characteristics
* Decreased signal strength
* Interference from other sources
* Multipath propagation
* SNR: singal-to-noise ratio
* Hidden terminal problem
	* B, A hear each other
	* B, C hear each other
	* A, C cannot hear each other interfering at B
* Signal attenuation
	* B, A hear each other
	* B, C hear each other
	* A, C signals cancel out at B

### IEEE 802.11 Wireless LAN
* Ranges from 802.11a, 802.11b, 802.11g, 802.11n
* All use CSMA/CA for multiple access
* All have base-station and ad-hoc network versions

### 802.11: Channels, association
* Divided into 11 channels at different fequencies
	* AP admin chooses frequency for AP
	* Interference is possible if channel is the same as the neighboring AP
* Host must associate with an AP
	* Scans channels, listening for beacon frames containing AP's name
	* Selects AP to associate with
	* May perform authentication
	* Typically runs DHCP to get IP address in AP's subnet
   
### Wireless Multiple Access (CDMA)
* Unique code assigned to each user