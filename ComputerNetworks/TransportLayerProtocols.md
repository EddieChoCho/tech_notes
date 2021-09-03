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

## References
* [Introduction to Computer Networks by Nen-Fu Huang](https://youtube.com/playlist?list=PLS0SUwlYe8cxktXNovos9xleroaWyb-z5)