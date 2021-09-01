# Foundation

## Applications

### Application Protocol
* URL: Uniform resource locater
* HTTP: Hyper Text Transfer Protocol
* TCP: Transmission Control Protocol
* 17 messages for one URL request
    * 6 to find the IP (Internet Protocol) address
    * 3 for connection establishment of TCP
    * 4 for HTTP request and acknowledgement
        * Request: I got your request and I will send the data
        * Reply: Here is the data you requested; I got the data
    * 4 messages for tearing down TCP connection

## Network Connectivity
* Important terminologies
    * Link
    * Nodes
    * Point-to-point
    * Multiple access
    * Switched Network
    * Circuit Switched
    * Packet Switched
    * Packet, message
    * Store-and-forward
    * Hosts
    * Switches
    * Spanning tree
    * internetwork
    * Router/gateway
    * Host-to-host connectivity
    * Address
    * Routing
    * Unicast/broadcast/multicast
    * LAN (Local Area Networks)
    * MAN (Metropolitan Area Networks)
    * WAN (Wide Area Networks)

### How datagrams are delivered in an Internet?

#### Cost-Effective Resource Sharing
* Resource: links and nodes
    * How to share a link?
        * `Multiplexing`
            * Multiplexing multiple logical flows over a single physical link
        * `De-multiplexing`
* `FDM: Frequency Division Multiplexing`
* `Synchronous Time-division Multiplexing (TDM)`
    * Time slots/data transmitted in predetermined slots

* `Statistical Multiplexing`
    * Data is transmitted based on demand of each flow.
    * What is a `flow`?
    * Packets vs. Messages
    * FIFO, Round-Robin, Priorities (Quality-of-Service (QoS))
    * Congested ?
    
    * A switch multiplexing packets from multiple sources onto one shared link

#### Logical Channels
* Logical Channels: Application-to-Application communication path or a pipe

#### Network Reliability
* Network should `hide the errors`
* Bits are lost
    * Bit errors (1 to a 0, and vice versa)
    * Burst errors – several consecutive errors
* Packets are lost (Congestion)
* Links and Node failures
* Messages are delayed
* Messages are delivered out-of-order
* Third parties eavesdrop

## Network Architecture
* Example of a layered network system:
    * Application Programs
    * Process-to-process Channels
    * Host-to-Host Connectivity
    * Hardware

* Protocols
    * `Protocol` defines the `interfaces` between
        * the layers in the same system and with
        * the layers of peer system
    * Building blocks of a `network architecture`
    * Each protocol object has two different interfaces
        * `Service interface`: operations on this protocol
            * e.g., Inside a host, the higher-level object operates the protocol through it's service interface. 
        * `Peer-to-peer interface`: messages exchanged with peer
            * e.g., 2 hosts communicate through the peer-to-peer interface of the protocol.

    * `Protocol Specification`: pseudo-code, state transition diagram, message format
    * `Interoperable`: when two or more protocols that implement the specification accurately
    * IETF: Internet Engineering Task Force
        * Define Internet standard protocols
 
* Protocol Architecture
    * Example of a protocol architecture
    * nodes are the protocols and links the “depends-on” relation

    ```
    [FTP] [HTTP] [DNS]
     └──┬─────┘   │
        │         │ 
       [TCP]    [UDP]
         └──┬─────┘
            │
           [IP]
            │
         [Internet]
    ``` 
    * Encapsulation: High-level messages are encapsulated inside of low-level messages

## Network Performance
* `Bandwidth`
    * Width of the frequency band
    * Number of bits per second that can be transmitted over a communication link
  
* 1 Mbps: 1 x 10^6 bits/second 
* 1 x 10^-6 seconds to transmit each bit or imagine that a timeline, now each bit occupies 1 micro second space.
* On a 2 Mbps link the width is 0.5 micro second.
* Smaller the width more will be transmission per unit time.
* Bits transmitted at a particular bandwidth can be regarded as having some width

* `Latency` = Propagation time + transmission time + queuing time
* `Propagation time` = distance/speed of light
* `Transmission time` = size/bandwidth
    * One bit transmission => propagation is important (Propagation time >> transmission time)
    * Large bytes transmission => bandwidth is important (Transmission time >> propagation time)
    
### Delay X Bandwidth
* The channel between a pair of processes can be viewed as a pipe
    * `Latency (delay)`: length of the pipe
    * `Bandwidth`: width of the pipe
* Delay x Bandwidth means how many data can be stored in the pipe
* For example, delay of 80 ms and bandwidth of 100 Mbps
    * 80 x 10-3 seconds x 100 x 106 bits/second
     * 8 x 106 bits = 8 M bits = 1 MB data.

* Relative importance of bandwidth and latency depends on application
    * `For large file transfer, bandwidth is critical`
    * `For small messages (HTTP, NFS, etc.), latency is critical`
    * Variance in latency (jitter) can also affect some applications (e.g., audio/video conferencing)

* If the sender keeps the pipe full, `delay x bandwidth` is the number of bits the sender must transmit before the first bit arrives at the receiver.
* Takes another one-way latency to receive a response from the receiver
* The sender will not fully utilize the network if the sender does not fill the pipe
    * send a whole `delay × bandwidth`(keep the pipe full) product’s worth of data before it stops to wait for a signal

### Throughput
* `Infinite bandwidth`
    * RTT (Round Trip Time) dominates
    * `Throughput` = TransferSize / TransferTime
    * TransferTime = RTT + TransferSize/Bandwidth
    * 1-MB file to 1-Gbps link looks like a 1-KB packet to 1-Mbps link

# References
* [Introduction to Computer Networks by Nen-Fu Huang](https://youtube.com/playlist?list=PLS0SUwlYe8cxktXNovos9xleroaWyb-z5)
