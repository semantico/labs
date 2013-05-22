---
layout: article
category: articles
title: Message brokers
author: edwinwilson
---

# Message brokers

A message broker is a physical component that handles communication between applications.  
Instead of communicating with each other, applications communicate only with the message broker.  
An application sends a message to the message broker, providing the logical name of the receivers.  
The message broker looks up applications registered under the logical name and then passes the message to them.

Communication between applications involves only the sender, the message broker, and the designated receivers.  
The message broker does not send the message to any other applications.  
(http://msdn.microsoft.com/en-us/library/ff648849.aspx)

Message brokers may support different ways of dispatching messages to their consumers,  
for example by using either a queue or a topic.

The topics and queues are hosted on a message broker,  
messages are sent from sender clients to these queues and topics then dispatched in the appropriate way to the appropriate receiver clients.

### Consumers
A consumer is a client which is capable of receiving messages from a topic or a queue.  
Once a consumer starts receiving messages from a **topic** it is a subscriber.  
When a consumer is to receive messages, it's receive method is called and the consumer retrieves messages (if there are any meant for it) from a queue or from a topic.

### Subscriber
A client capable of subscribing to and receiving messages from a topic.

### Publishers/producer
A publisher is a client capable of sending messages to a queue or topic.

### Topics
A topic receives a message, then immediately dispatches it to all subscribers that are "active" at the time.
A subscriber is said to be active when it's receive method is running.

### Durable subscribers
A durable subscriber will receive all messages from a topic,  
even if the message has been sent to the basic subscribers already and the durable subscriber was inactive at the time.  
When a topic has a durable subscriber, it keeps messages received from the publishers until they have been dispatched to all durable subscribers,  
then disposes of them.

### Queues
A queue keeps a message until one of it's consumers retrieves it, a single message will be received by exactly one consumer.  
If there are no consumers available at the time the message is sent, the message will be kept until a consumer is available that can process it.

### Durable queues
Durable queues keep messages around persistently for any suitable consumer to consume them.  
Durable queues do not need to concern themselves with which consumer is going to consume the messages at some point in the future.  
There is just one copy of a message that any consumer in the future can consume.(http://activemq.apache.org/how-do-durable-queues-and-topics-work.html)

# General message broker functionalities

### Flow control

Flow control is used to limit the flow of data between a client and server,  
or a server and another server in order to prevent the client or server being overwhelmed with data.  
(http://docs.jboss.org/hornetq/2.2.14.Final/user-manual/en/html/flow-control.html)  
Consumer or producer flow control is possible depending on what broker is being used.

### Persistent and non persistent messaging

If you are using persistent delivery, messages are persisted to disk/database so that they will survive a broker restart.
When using non-persistent delivery, if you kill a broker then you will lose all in-transit messages.  
The effect of this difference is that persistent messaging is usually slower than non-persistent delivery.  
(http://activemq.apache.org/what-is-the-difference-between-persistent-and-non-persistent-delivery.html)

### Queue replication

Queue replication's purpose is to provide backup for a queue in case of failure from a broker by replicating a used queue on a primary broker to a backup broker.  
The queue that will be replicated triggers events when messages are enqueued or dequeued.  
A queue with the purpose of being a backup queue is created on the backup broker.  
The events are sent to the backup broker and allow it to update the backup queue.  
In the event of the primary broker not responding, clients can failover to the backup broker.

### Message store and forward
Store and forward of messages works in a cluster of brokers.  
Messages sent to a broker are passed on from broker to broker inside the cluster until there is a consumer available to consume them.

### Message TTL
A TTL defining how long a message is kept on a queue or topic before it is deleted can be set.

### Destination TTL
A TTL defining how long to wait before a inactive queue or topic is deleted.

### Message filtering consumers

A consumer can be set to ignore certain messages by using message filtering.

# ActiveMQ

ActiveMQ is a popular open-source message broker supported by the Apache foundation.  
It is written in and closely linked with Java and fully implements JMS.  
It is widely used and supported.

### Embedded Java deployment and configuration

ActiveMQ is entirely exploitable by using Java code and configuration files.

### Networks of brokers

It is possible with ActiveMQ to configure broker networks.  
These networks allow you to manage the number of clients per broker and to survive the failure of a particular broker  
(a client can fail over to a broker of the same network in case of failure of it's own broker).  
They enable support of distributed queues and topics across a network  
(a consumer on a broker from a network can receive messages from a queue or topic from a different broker on the same network).  
Messages will only be forwarded to brokers which have consumers attached to them.

### Master/slave brokers

The problem with running lots of stand alone brokers or brokers in a network is that messages are owned by a single physical broker at any point in time.  
If that broker goes down, you have to wait for it to be restarted before the message can be delivered.  
(If you are using non-persistent messaging and a broker goes down you generally lose your message).

The idea behind MasterSlave is that messages are replicated to a slave broker so that even if you have catastrophic hardware failure of the master's machine,  
file system or data centre, you get immediate failover to the slave with no message loss.  
(http://activemq.apache.org/clustering.html)

ActiveMQ supports :

* Shared File System Master Slave : with a shared file system
* JDBC Master Slave : when working with a single database
* Replicated LevelDB Store : uses Apache ZooKeeper

### Sending large files

Sending big files with ActiveMQ uses BlobMessages(ActiveMQ Java API) which allows to send big messages directly from a file or a stream.
The process for receiving large messages is exactly the same as for receiving ordinary messages.

### Supported protocols

* AMQP
* OpenWire
* REST
* STOMP
* XMPP
* WS Notification

* SSL is supported

### Embedded or standalone

#### Standalone

* Configuration : .XML, .properties, shell command line 
* Installation :
* Unix, Windows : download and extract archives
* OSX : HomeBrew installer

#### Embedded

* Configuration : all the standalone possibilities, fully configurable through Java code
* Installation : ActiveMQ api
* No feature limitations compared to standalone


# Apollo

Apollo seems to be a much faster and more reliable broker than ActiveMQ.  
It is based on ActiveMQ, but it has been developed in Scala and the threading sections have been rethought.  
Whereas, it has less features, for example no broker networks.  
But it has new exclusive features like the REST management API, message sequences, runtime configuration updates.  
Also, it supports less protocols, STOMP, AMQP, MQTT and OpenWire are the only one are the only ones supported at the moment.  
Most of the functionalities that can be found in ActiveMQ should be implemented in Apollo in the future.

# Apache QPid

The Apache QPid message broker exclusively supports the AMQP messaging protocol.
Brokers are available as a Java version and a C++ version.

Clients are available in Java (JMS 1.1), C++, C# .Net, WCF Adapter, Python and Ruby.

### Features

Both brokers have the following features : 
* Queues
* Topics
* Support of durable consumers
* Persistent messaging
* Transactional messaging
* Queue replication

The C++ version adds to these the following exclusive features : 
* Broker clustering
* Broker networks (federation)
* Unix command line management tools
* Runtime Unix command line configuration

* Apache QPid does not enable embedded running of brokers

### Drawbacks

* No flow control in networks
* Redundant paths in networks are not automatically managed

### Apache QPid's broker networking

Under the name federation in their documentation, has the same behaviour as an ActiveMQ broker network.

# Benchmarking : ActiveMQ, RabbitMQ, Apollo, HornetQ with STOMP protocol

Numbers from http://hiramchirino.com/stomp-benchmark/ubuntu-2600k/index.html

### Benchmarking with different consumer/producer loads on one queue or topic

##### Queue
##### Non persistent
Apollo and ActiveMQ, better consumer rates, Apollo and RabbitMQ, better producer rates.

Consumer rates example : non persistent, 5 consumers, 5 producers, 1 queue.

* Apollo     : 262,878 msg/second

* ActiveMQ  : 132,133 msg/second

* HornetQ   : 0 msg/second

* RabbitMQ  : 6,126 msg/second

Producer rates example : non persistent, 5 consumers, 5 producers, 1 queue.

* Apollo	  : 236,899 msg/second

* ActiveMQ  : 124,822 msg/second

* HornetQ   : 0 msg/second

* RabbitMQ  : 0 msg/second

##### Persistent
RabbitMQ better producer and consumer rates when used with 1 producer.
Apollo, faster with 5 and 10 producers. (all tested with 1, 5 and 10 consumers)

Consumer rates example : persistent, 5 consumers, 1 producer, 1 queue.

* Apollo   : 2,161 msg/second

* ActiveMQ : 847 msg/second

* HornetQ : 0 msg/second

* RabbitMQ : 5562 msg/second

Producer rates example : persistent, 5 consumers, 1 producer, 1 queue.

* Apollo : 2,110 msg/second

* ActiveMQ : 844 msg/second

* HornetQ : 0 msg/second

* RabbitMQ : 5,533 msg/second

Consumer rates example : persistent, 5 consumers, 10 producers, 1 queue.

* Apollo : 44,138 msg/second

* ActiveM : 4887 msg/second

* HornetQ : 0 msg/second

* RabbitMQ : 23,726 msg/second

Producer rates example : persistent, 5 consumers, 10 producers, 1 queue.

* Apollo : 43,927 msg/second

* ActiveMQ : 3376 msg/second

* HornetQ : 0 msg/second

* RabbitMQ : 23,286 msg/second

##### Topic
##### Non Persistent
Apollo has much better consumer and producer rates than the rest, ActiveMQ second.  
(e.g. : 1 topic, 10 consumers, 10 producers : Apollo 957,677 msg/second, ActiveMQ 353,283 msg/second, RabbitMQ 20997 msg/second, HornetQ 0msg/second)
##### Persistent
Apollo and ActiveMQ ahead of RabbitMQ and HornetMQ with increasing gap when number of producers is higher. ActiveMQ nearly as fast as Apollo.

---

### Benchmarking with different message sizes

#### Queue
##### Non persistent
Apollo and ActiveMQ better consumer and producer rates with with 20b and 1k messages.
RabbitMQ faster with 256k messages.

Consumer rates example : non persistent, 10 consumers, 10 producers, 10 queue, 1k message.

* Apollo : 320,588 msg/second

* ActiveMQ : 97,568 msg/second

* HornetQ : 208 msg/second

* RabbitMQ : 30,456 msg/second

Consumer rates example : non persistent, 10 consumers, 10 producers, 10 queue, 256k message.

* Apollo : 1,214 msg/second

* ActiveMQ : 1,245 msg/second

* HornetQ : 8,912 msg/second

* RabbitMQ : 1,684 msg/second

Producer rates example : non persistent, 10 consumers, 10 producers, 10 queue, 1k message.

* Apollo : 324,062 msg/second

* ActiveMQ : 97,112 msg/second

* HornetQ : 2,004 msg/second

* RabbitMQ : 0 msg/second

Producer rates example : non persistent, 10 consumers, 10 producers, 10 queue, 256k message.

* Apollo : 1,212 msg/second

* ActiveMQ : 1,240 msg/second

* HornetQ : 0.0 msg/second

* RabbitMQ : 1,683 msg/second

##### Persistent
Same as non persistent.

#### Topic
##### Non persistent
Apollo much faster particulalrly with high number of producers, consumers and topics, with 20b, 1k messages, ActiveMQ comes 2nd, then RabbitMQ.  
With 256k messages, RabbitMQ is faster than Apollo, and ActiveMQ, while thir results are comparable.
##### Persistent
Same as non persistent.

---

### Request/reply scenarios

In this scenario producers send a request message with a 20 byte body to a queue the consumers are subscribed to  
and then waits for response on a queue unique to each producer.  
The request message has the `reply-to` header set to the queue which the producer is will wait for the response on.  
Consumers are receive the request message and send a response message without a body to the `reply-to` queue specified on the request.  
The 99.9th percentile latency is measured using the time elapsed from when the producer sends the request message to when the producer receives the response message. 

#### Queue
##### Non persistent
Request Rates (requests/second) : RabbitMQ, better with 1 producer, Apollo and ActiveMQ better with better with 10 and 100 producers.

99.9th latency : ActiveMq only one to show any latency.

###### Persistent
Same as non persistent.
