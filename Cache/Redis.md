# Redis

## Transactions

* Reference: https://redis.io/docs/manual/transactions/

* Redis Transactions allow the execution of a group of commands in a single step.
* They are centered around the commands MULTI, EXEC, DISCARD and WATCH.
* Redis Transactions make two important guarantees:
    1. Single isolated operation

    * All the commands in a transaction are serialized and executed sequentially. A request sent by another client will
      never be served in the middle of the execution of a Redis Transaction.

    2. The EXEC command
        * The EXEC command triggers the execution of all the commands in the transaction
            * If a client loses the connection to the server in the context of a transaction before calling the EXEC
              command none of the operations are performed, instead if the EXEC command is called, all the operations
              are performed.
        * When using the append-only file Redis makes sure to use a single write(2) syscall to write the transaction on
          disk.
            * However, if the Redis server crashes or is killed by the system administrator in some hard way it is
              possible that only a partial number of operations are registered. Redis will detect this condition at
              restart, and will exit with an error. Using the redis-check-aof tool it is possible to fix the append-only
              file that will remove the partial transaction so that the server can start again.
            * Using the redis-check-aof tool it is possible to fix the append-only file that will remove the partial
              transaction so that the server can start again.

### Usage

* MULTI
    * A Redis Transaction is entered using the MULTI command. The command always replies with OK. At this point the user
      can issue multiple commands.
    * Instead of executing these commands, all commands will reply with the string QUEUED (sent as a Status Reply from
      the point of view of the Redis protocol).
    * A queued command is simply scheduled for execution when EXEC is called.

* EXEC
    * The EXEC command triggers the execution of all the commands in the transaction.
    * EXEC returns an array of replies, where every element is the reply of a single command in the transaction, in the
      same order the commands were issued.

* DISCARD
    * It flushes the transaction queue and will exit the transaction.

TBD...

## Redis Pub/Sub

* SUBSCRIBE, UNSUBSCRIBE and PUBLISH implement the Publish/Subscribe messaging paradigm
    * Senders (publishers) are not programmed to send their messages to specific receivers (subscribers). Rather,
      published messages are characterized into channels, without knowledge of what (if any) subscribers there may be.
    * Subscribers express interest in one or more channels and only receive messages that are of interest, without
      knowledge of what (if any) publishers there are.
    * This decoupling of publishers and subscribers allows for greater scalability and a more dynamic network topology.

* e.g.,
    ```
        //A client subscribes to channels "channel11" and "ch:00" 
        SUBSCRIBE channel11 ch:00
    ```
    * Messages sent by other clients to these channels will be pushed by Redis to all the subscribed clients.
    * Subscribers receive the messages in the order that the messages are published.
* TBD...

### Delivery semantics

* At-most-once message delivery semantics
* Once the message is sent by the Redis server, there's no chance of it being sent again.
* If the subscriber is unable to handle the message (for example, due to an error or a network disconnect) the message
  is forever lost.
* If your application requires stronger delivery guarantees, you may want to learn about `Redis Streams`.
    * Messages in streams are persisted, and support both `at-most-once` as well as `at-least-once` delivery semantics.

### Format of pushed messages

* TBD...

### Database & Scoping

* Pub/Sub has no relation to the key space. It was made to not interfere with it on any level, including database
  numbers.
    * Publishing on db 10, will be heard by a subscriber on db 1.
    * If you need scoping of some kind, prefix the channels with the name of the environment (test, staging,
      production...).






