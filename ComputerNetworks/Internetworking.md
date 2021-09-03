 # Internetworking
 
 ## Introduction
 * What is IP ?
    * IP stands for Internet Protocol
    * Key tool used today to build scalable, heterogeneous internetworks
    * It runs on all the nodes in a collection of networks
    * Defines the infrastructure that allows these nodes and networks to function as a single logical internetwork
    
 ### IP Service Model
 * Packet Delivery Model
    * `Connectionless model` for data delivery
    * `Best-effort delivery` (unreliable service)
        * packets are lost
        * packets are delivered out of order
        * duplicate copies of a packet are delivered
        * packets can be delayed for a long time
* Global Addressing Scheme
    * Provides a way to identify all hosts in the network

### How Layer 3 Routers Work ?
* Layer 3 router uses store and forward scheme to forward incoming IP packets (datagrams).
    * IP Address Lookup (Forwarding Table constructed by routing protocols, such as RIP, OSPF, BGP, etc)
    * IP/MAC mapping table

# References
* [Introduction to Computer Networks by Nen-Fu Huang](https://youtube.com/playlist?list=PLS0SUwlYe8cxktXNovos9xleroaWyb-z5)