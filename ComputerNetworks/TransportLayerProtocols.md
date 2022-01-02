# Transport Layer Protocols

## Introduction to end-to-end protocols
* A transport protocol is usually expected to provide
    * `Guaranteed` message delivery
    * Delivers messages in the `same order` they were sent
    * Delivers `at most one copy` of each message
    * Supports arbitrarily large messages
    * Supports `synchronization` between the sender and the receiver
    * Allows the receiver to apply `flow control` to the sender
    * Supports multiple application processes on each host

* Typical limitations of the network service (`Unreliable service`, like IP of Internet) on which transport protocol will operate
    * `Drop` messages
    * `Reorder` messages
    * Deliver `duplicate` copies of a given message
    * Limit messages to some `finite size`
    * Deliver messages after an `arbitrarily long delay`

* Challenge for Transport Protocols
    * Develop algorithms that `turn` the `unreliable service` of the underlying network `into` the `service` required by application programs
    * `Unreliable service -> Unreliable service (UDP)`
    * `Unreliable service -> Reliable service (TCP)`

## Simple Demultiplexer protocol (UDP)
* Extends host-to-host delivery service of the underlying network into a `process-to-process` communication service
* Adds a level of `demultiplexing` which allows multiple application processes on each host to share the network

* Format for UDP header
    ```
    ┌─────────┬─────────┐
    │SrcPort  │`DstPort`│
    ├─────────┼─────────┤
    │Checksum │Length   │
    ├─────────┼─────────┤
    │                   │
    │        Data       │
    │                   │
    └───────────────────┘
    ```

* UDP Packet Demultiplexer
    ```
           ┌───[Queue]───[Application Process(Port X)]
           │
    [UDP]──┼───[Queue]───[Application Process(Port Y)]
           │
           └───[Queue]───[Application Process(Port X)]	
    ``` 

## Reliable Byte Stream (TCP)

* In contrast to UDP, Transmission Control Protocol (TCP) offers the following services
  * Reliable
  * Connection oriented
  * Byte-stream service

* Flow control VS Congestion control
  * `Flow control` involves preventing senders from overrunning the capacity of the receivers.
    * Control by receiver

  * `Congestion control` involves preventing too much data from being injected into the network, thereby causing
    routers/switches or links to become overloaded.
    * e.g., Package loss -> might have congestion issue

* End-to-end Issues
  * TCP runs `over the Internet` rather than a point-topoint link
  * `The TCP sliding window algorithm` need to consider:
    * TCP supports `logical connections` between processes that are running on two different computers in the Internet
    * TCP connections are likely to have widely `different RTT`(Round Trip Time) times
    * Packets may get `reordere`d in the Internet

  * TCP needs a mechanism using which each side of a connection will `learn what resources` the other side offers to the
    connection -Flow Control
  * TCP needs a mechanism using which the sending side will `learn the capacity` of the network - Congestion Control

### TCP Segment

* TCP is a `byte-oriented protocol`
* The `sender writes bytes` into a TCP connection and the `receiver reads bytes` out of the TCP connection.
* However, TCP does `not transmit individual bytes` over the Internet
* The source TCP `buffers enough bytes from the sending process` to fill a reasonably sized packet and then sends this
  packet to its peer on the destination host.
* The destination TCP then puts the contents of the packet into a `receive buffer`, and the receiving process reads from
  this buffer.
* The packets exchanged between TCP peers are called `segments`.

### TCP Header

* The `SrcPort` and `DstPort` fields identify the source and destination ports, respectively.
* The `Acknowledgment`, `SequenceNum`, and `AdvertisedWindow` fields are all involved in TCP’s sliding window algorithm.
* Because TCP is a byte-oriented protocol, `each byte of data has a sequence number`; the SequenceNum field contains the
  sequence number for the `first byte of data` carried in that segment.
* The Acknowledgment and AdvertisedWindow fields carry `information about the flow of data` going in the other
  direction.

* The 6-bit Flags field is used to `relay control information` between TCP peers.
* The possible flags include `SYN, FIN, RESET, PUSH, URG`, and `ACK`.
* The `SYN and FIN flags` are used when establishing and terminating a TCP connection, respectively.
* The `ACK flag` is set any time the Acknowledgment field is valid, implying that the receiver should pay attention to
  it.
*
* The `URG flag` signifies that this segment contains urgent data. When this flag is set, the `UrgPtr field` indicates
  where the nonurgent data contained in this segment begins.
* The urgent data is contained `at the front of the segment` body, up to and including a value of UrgPtr bytes into the
  segment.
* The `PUSH flag` signifies that the sender invoked the `push operation`, which indicates to the receiving side of TCP
  that it should notify the receiving process of this fact.

* The `RESET flag` signifies that the receiver has become confused, it received a segment it did not expect to
  receive—and so wants to abort the connection.
* Finally, the `Checksum field` is used in exactly the same way as for UDP—it is computed over the TCP header, the TCP
  data, and the `pseudoheader`, which is made up of the source address, destination address, and length fields from
  the `IP header`.

## References

* [Introduction to Computer Networks by Nen-Fu Huang](https://youtube.com/playlist?list=PLS0SUwlYe8cxktXNovos9xleroaWyb-z5)