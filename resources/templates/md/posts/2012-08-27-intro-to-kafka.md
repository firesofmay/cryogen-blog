{:title "Intro to kafka"
 :layout :post
 :tags  ["kafka"]}

### What is Kafka?
Kafka provides a fast, distributed, highly scalable, highly available, publish-subscribe messaging system. It solves communicatin and integration between components of large software systems.

Distributed Messaging System
(Other messaging systems are RabbitMQ, ActiveMQ etc)

By distributed we mean it'll scale out, it'll work on multiple servers.

Getting data from one system to another is a challenging problem
It can easily become a mess one everything is trying to talk directly to everything else. You need someone to coordinate that flow of data across systems.

Kakfa solves just that problem. Having Producers write data to Kafka, and Having Consumers read data from Kafka.


### Unique about Kafka?

* Large number of Consumers
	* Scales really well with really large number of consumers.
* Ad-hoc consumers
	* Kafka doesn't need to know you need data from Kafka.
	* Not a performance issue.
* Batch consumers
	* Can have a consumer waking up say every hour, getting that hours worth of data and processing that.
	* This was a performance issue before Kafka.
	* Great for batch systems like Hadoop.
* Automatic recovery from broker failures.
	* If one of the brokers go down, data is not lost.
	* There maybe tiny delay in that scenario though.

### What do we do with Kafka?
* Messaging - Communication between apps
	* Using to it to notify one to another.
* Website Activity Tracking (Clicks, Searches)
	* A small kafka cluster can support large number of events per second.
* Metrics collection - instead of writing to logs
	* If we would like to store and analyze different kinds of data about machines, cpu memory, collect data and keep it in kafka, and other systems can analyze it.
* Audit
* Source & target of Stream Processing.

### What Kafka Doesn't do?
* It's not and end-user solution. You need to write code to use it.
	* Non developer cannot use this. Not an equivalent of log stash for example.
* Not a drop-in JMS (Java Messaging Service) replacement.
	* Example cannot just replace ActiveMQs with Kafka as it has different APIs/Semantics	.
* It doesn't have many ready-made producers/consumers.
* No data transformations.
* No encryption/auth/authentication (yet).
	
### Misc
* Kafka is written in Scala and runs on the JVM.
* Can store many TB of messages
* Messages live for a certain time.

### Kafka has

* Groups
* Partitions

In Kafka a server is called a broker

Producers sends the message to broker to a given topic
Consumers connects to broker and tell them what topic they want to listen to and what group they belong to.

Each Producer/Broker/Consumer are easy to add/remove from to the system. And every other component gets notified accordingly.

To coordinate among each other Kafka uses Zookeeper.
Say when a new broker is started up, it'll write its presence to Zookeeper, so that both producers and subscribers will get call backs that new broker is available. Similarly when broker goed down. Similarly for producers/consumers.

### Why is Kafka Fast?

* No message committing. In other similar systems, consumer has to commit that it has received the message, In Kafka thats not the case.
* When broker recieves the message from producer, all it does is appends it to a log file. 
* No state on Broker. Broker does not store which user were getting which messages. Hence makes it fast
* Kafka appends file upstream (Producer) & writes file downstream (Consumer).
* Offset is the King.

### Kafka Basics

* Messages are organized into **topics**
	* When you send a message you send to a topic
	* When you are reading a message you need to say what topic to read from.
	* When you design your system you need to design it around differnt topics of interest for different consumers.
* **Producers** push messages
* **Consumers** pull messages
	* Consumers subscribe to topic and they get the messages.
* Kafka runs in a cluster. Nodes are called **brokers**.
* A single topic can have multiple partitions.
	* Some topics can have lot of data so you want to partition it across multiple machines. And then you can put each partition on a seperate machine.
	* Multiple Consumers can also be configured to read from a particular topic and each of those consumers can read from a particular partition. Helps in high throughput.
	* Inside each partition you'll see ids. Those ids are called offset.
	* When consumer consumes messages you say I want to consume messages from this offset.
	* Say you a consumer broke connection, you reconnect and say what offset you want to continue from. That way messages are not lost and at the same time brokers don't have to maintain any kind of state making them super fast.
	* Producer partitions data and adds it in partitions, hence order is maintained withing a partition.
	* Order of messages are maintained only withing a partition, not across partitions.
	* A specific message is uniquely identified by topic+partition+offset
	* Partition is a log
	* You write to it sequentially.
	* Multiple consumers can read from multiple points of the log.
	* Data is stored for a certain amount of time say a week.
	* If a consumer hasn't read that data withing that period that data is lost else it can catch up within that given time.

	
CONTINUE FROM VIDEO 2 AT 3mins
	
References:

* [Apache Kafka - distributed publish-subscribe messaging system - Morten Kjetland](https://vimeo.com/62298867)
* [Introduction to Apache Kafka (Paid)](http://shop.oreilly.com/product/0636920038603.do)
* Kafka Slides - http://web.stanford.edu/class/ee380/Abstracts/141112-slides.pdf
* http://kafka.apache.org/08/design.html
* https://www.hakkalabs.co/articles/apache-kafka-top-12-tutorials-on-performance-more
* http://www.happiestminds.com/sites/default/files/article/ANALYTICS_ON_BIG_FAST_DATA_USING_A_REALTIME_STREAM_DATA_PROCESSING_ARCHITECTURE.pdf
* http://www.quora.com/Whats-the-best-way-to-start-learning-Kafka