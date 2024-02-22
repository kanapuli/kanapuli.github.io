+++
title = 'Problems With Mirrored Queues'
date = 2024-02-22T17:49:36+01:00
draft = false
tags = ['rabbitmq']
+++

Let's talk about classic mirrored queues in Rabbitmq.

### To start with, what is even a Queue ?

A queue is an ordered collection of messages. Messages can be either enqueued or dequeued from a queue. Usually a queue is tied to a single node and stores the messages. 

But if high availability of the messages is necessary, then a different approach should be taken. Mirrored Queues. Yes, as the name suggests, the queues have to mirrored into different copies in order to keep it highly available. These mirrors have to placed in different brokers so that a failure of one broker won't result in a data loss.

Among these redundant copies of the same queue, one copy will act as the Master queue and the other copies are called Mirror queues.

### How is data synchronised between the Master queue and the Mirrors ?

Rabbitmq uses the **Chain Replication Algorithm** to synchronise data between the masters and the mirrors. 

When a message is published to the master, the master queue forwards the message to its next mirror in the chain. The mirror processes the message and forwards the message to the next mirror in the chain. This continues until the last mirror in the chain notifies the master that the operation is fully replicated. Thus a ring is formed between the master queue and the mirrors.

![Chain replication](/chain-replication.png)

#### Problem 1: Network Calls
Let's talk about our first problem in Mirrored queues. During some edge cases when produced message loss, this algorithm is changed so that the publisher channel process would additionally send the message directly to the mirror in addition to publishing to the master. This means that the message has to be forwarded twice in the ring and this is going to make a lot of network calls between the brokers.

***Note: Usually the messages are produced to and consumed from the Queue that acts as the master.***

![Chain replication double send](/double-send-chain-replication.png)

#### Problem 2: Synchronisation

The second problem with mirrored queues arises due to synchronisation. The mirrors can always leave from the chain replication algorithm link due to server restarts or a node failure or due to a network partition. 

When the mirror has been out of communication and then join the ring back, they always start from a clean slate. The joining mirror always wipes it data first and starts synchronisation with the queue master. This mirroring happens fast for smaller queues but could take some minutes, hours or even days to mirror queues with large number of messages. Most importantly, **the mirroring process is a stop the world process.** Producers cannot publish and consumers cannot consume the message unless the mirroring completes.


![Stop the world sync](/stop-the-world-sync.png)

#### Problem 3: Redundancy

The third issue with mirrored queues is caused by redundancy. When using mirrored queues, each queue needs a mirror in another broker. For example, if there are 3 brokers and each one has a unique master queue and a mirror of another master queue from a different broker. If one broker fails, Rabbitmq promotes the mirror of the lost master queue to become the new master queue and creates a mirror for it in the only other working broker. This is good, but when the third broker rejoins the cluster, it will be mostly empty, leading to an unbalanced cluster.

#### Problem 4: Network Partition

The fourth problem is how mirrored queues handle about the network partitions. When a network paritition occurs, it is going to split the cluster into two halves and we shall end up with one or more mirro that lose communication with the master queue. Here there is a trade off between consistency and availability.

If one chooses consistency over availability, then the `cluster_partition_handling` setting has to be set to `pause_minority` where the minority brokers in the split cluster will be paused from all its operations. On the majority side, the queues continues to operate with reduced redundancy. 

![](/pause-minority.png)

If we want continued availability, we can set the `cluster_partition_handling` to either `ignore` or `auto-heal` mode. This will allow the mirror in the splitted cluster to be promoted to master, meaning we will end up with a state where there are masters for a queue on both halves of the partition. Clients can publish to and consume from both these masters. Unfortunately, when the cluster is healed, one of these master is chosen as victim and restarted, leading to message loss in that queue. 

![](/auto-heal.png)

##### Finally... 

These are the problems with mirrored queues in Rabbitmq and can cause serious troubles when working in an high throughput messaging system. A better replication algorithm is needed and Rabbitmq has come up with Quorum queues. Let's talk about quorum queues in another post.
