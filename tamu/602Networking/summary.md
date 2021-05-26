ECEN602 Computer Communication and Networking Summary
---

Part1: Fundation
---
- OSI 7-Layer Model
```
       |  - Application  Layer  |  High-level APIs, including resource sharing, remote file access
Host   |  - Presentation Layer  |  Translation of data between a networking service and an application; including character encoding, data compression and encryption/decryption
Layers |  - Session      Layer  |  Managing communication sessions, i.e., continuous exchange of information in the form of multiple back-and-forth transmissions between two nodes
       |  - Transport    Layer  |  Reliable transmission of data segments between points on a network, including segmentation, acknowledgement and multiplexing
--------------------------------+
Media  |  - Network      Layer  |  Structuring and managing a multi-node network, including addressing, routing and traffic control
Layers |  - Link         Layer  |  Reliable transmission of data frames between two nodes connected by a physical layer
       |  - Physical     Layer  |  Transmission and reception of raw bit streams over a physical medium
```
- Network Performance
    - Bandwidth, Throughput
    - Latency ( RTT )
        - Propagation
        - Transmit
        - Queue
    - Delay x Bandwidth Product
        - whether the network is fully utilized

Part2: Physical Layer
---
- Encoding
- Modulation
    - What?
    
        modulation is the process of varying one or more properties of a periodic waveform, called the carrier
        signal, with a separate signal called the modulation signal that typically contains information to be transmitted. 
    
    - Why?
        
        The purpose of modulation is to impress the information on the carrier wave, which is used to carry the information
        to another location.
        
    - How?
- Media —— Links

    All practical links rely on some sort of electromagnetic radiation propagating through a medium or through free space

    - Copper wire
    - Optical fiber
    - Air/free space (wireless links)
    
Part3: Link Layer (Host-to-Host)
---
- Basics
    - Functionality
        - framing
        - error detection & correction
        - reliable transmission
        - shared media access control
    - Where does it implement? Network Adapter or Network Interface Card(NIC)
- Framing
    - Why ?
    
        The adapter needs to know where it should send bits to. If it transmits data bit-by-bit, each bit would be attached
        with a header providing where it should go, whether or not it has been corrupted and etc. This is obviously 
        inefficient. Transmitting data in frame amortizes this overhead.
        
    - How?
    
        The fundamental problem is how to mark the start and the end of a frame.
    
        - Byte-oriented
            - Byte stuffing
                - use special characters to mark start and end
                - escaping these special characters that appear in data
            - Byte Counting
                - count field: how many bytes in data
        - Bit-oriented
            - Bit stuffing

- Error Detection
    - Causes
        - electrical interference
        - thermal noise
        - hardware errors
        - memory errors
    - Techniques
        - 2-dimensional parity
            - odd parity
            - even parity
        - Internet checksum (TCP)
        - CRC
- Error correction
    - Hamming Code
    - Convolutional Code
    - Turbo Code
    - Reed-Solomon Codes
    - Low-Density Parity Check (LDPC) Codes
    
- Reliable Transmission
    - Mechanism
        - Acknowledgement
        - Timeout
    - Protocol
        - Stop-and-Wait
        - Go-Back-N
        - Selective Acknowledgement(SACK)
        
- Shared Media Access Control

Part4: Network Layer (Host-to-Host)
---
- Functionality
    - Packet Forwarding (or Switching), Fragmentation & Reassembly, Best-effort Delivery
        - Layer2 Switching device: switch
        - Layer3 Switching device: router
    - Routing 
    - Global Addressing
    
- Forwarding (or Switching)
    - Basics
        - Datagrams (Connectionless)
        
            Idea: include in every packet enough information to enable any switch to decide how to get it to its destination
            - Consult forwarding table(if in layer2) or routing table(if in layer3)
        
        - Virtual Circuit (Connection-oriented)    
            - Permanent Virtual Circuit (PVC)
            
                Connection established by the administrator permanently.
            
            - Signaling Virtual Circuit (SVC)
            
                Connection setup -> Communication -> Connection teardown
                
        - Source Routing
        
            Idea: All the information about network topology that is required to switch a packet across the network is
            provided by the source host
            
    - Bridges
        - Simplest Strategy: forward packets to all other output ports
        - Learning Address: no need to forward packets if the sender and the receiver connected at the same port
            - record information (which port does the sender resides on)
            - timeout
    
    - LAN Switches
        - Preventing loops in switched LAN
            - Distributed Spanning Tree algorithm
                - Spanning tree: is a sub-graph of this graph that covers all the vertices but contains no cycles
                - Idea: Each switch/bridge decides the ports over which it is and is not willing to forward frames
                    - reduced to acyclic tree by removing ports
                    - algorithm is dynamic
                - Algorithm:
                    - Rules
                        - Select the bridge with the smallest ID as the root
                        - Each LAN selects the bridge over which it is connected and
                          closet to root as its designated bridge
                        - Each LAN only forward frames to its designated bridge
                    - Tree build
                        - Bridges exchange configuration messages
                            - ID for bridge sending the message
                            - ID for what the sending bridge believes to be the root
                            - distance from sending bridge to root bridge
                        - Each bridge records the current best configurations
                        - Initially, each bridge believes itself root 
                        - When learn not root, stop generating config messages
                        - When learn not designated bridge, stop forwarding config messages
                        - Use root't config messages as heartbreak 
        
        - Switching mechanisms
            - Store-and-Forward
                - incoming frame is completely stored in memory before sending frame out
                - switch latency equals an entire frame time
            - Cut-through 
                - forward the frame before reading the entire packet
                - frames with CRC error will be forwarded (because CRC is at the end of the frame)
                - low switch latency
                - 

- IP Protocol
    - IP Service Model
        - Connectionless (datagram) model for data delivery
        - Best-effort delivery
            - packet loss
            - packets out of order
            - duplicate packet
            - arbitrary length of delay
        - Global Addressing Scheme
        
    - IPv4 Address: 165.91.0.0/16 (network part/host part)
    
    - Classless Inter-domain Routing (CIDR)
        - What?
        
            A method for allocating IP addresses and for IP routing
        
        - Why?
            - slow the growth of routing tables on routers
            - slow the exhaustion of IPv4 Address
        - How?
            - CIDR notation: 165.91.0.0/16 (network part/host part)
            - Longest match rule
        - Subnets
        
            Idea: take a single IP network number and map the IP addresses within that network number to several physical
            networks that are referred to as subnets
    
    - Fragmentation and Reassembly
        - fragment when networks have different MTU
        
    - IP Datagram Forwarding
        - Strategy
            - every datagram contains the destination's IP address
                - if belongs to the same network(network par is same), forward to the host directly
                - if not, forward to some routers
            - forwarding table: network number -> next hop 
                - for subnets, network number is computed by mask the IP address with subnet mask
            - each host has a default router
    
    - Address Resolution Protocol (ARP)
        - What & Why?
        
            A protocol to translate IP addresses into physical network addresses
            
        - How?
            - Table-based: IP-to-Physical table
            - Broadcast request if IP address not in table
            - Timeout 
    
    - Dynamic Host Configuration Protocol (DHCP)
        - What?
            - provide host-specific information to a host from DHCP server
                - client IP address
                - subnet mask
                - default router
                - DNS server
                - 
            - a mechanism to assign IP address to hosts
        - Why? manual configuration is cumbersome
        - How?
            - manual allocation: network admin assigns a fixed address
            - automatic allocation: DHCP server assigns a permanent address
            - dynamic allocation: DHCP assigns address for limited time; Clients can renew the lease
    
    - Internet Control Message Protocol (ICMP)
        - echo
        - error message
            
        
Part5: Transport Layer
---

Part6: Session Layer
---

Part7: Presentation Layer
---
- Why?
    - different Network Byte Order
    - different integer and floating point formats
    - different compilers represent data structures in memory with different padding and end-of-line character


Part8: Application Layer
---