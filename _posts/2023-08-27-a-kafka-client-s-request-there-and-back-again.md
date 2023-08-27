---
layout: post
title: 'A Kafka Client’s Request: There and Back Again'
date: 2023-08-27 02:32 +0200
published: true
---

![Kafka](https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/Apache_kafka_wordtype.svg/640px-Apache_kafka_wordtype.svg.png){: .right .w-25}

This talk is a behind the scene in Kafka of what happens when you push an event. It's important to know what happens because:
- there are a lot of things happening even before the event leaves for the broker
- one must know which parameter and values to investigate when **something goes wrong** or when **performances are not good**.

It mainly goes to keeping an eye on some parameters and values, but it's interesting to understand what's going on under the hood.

### Before leaving the producer

These are a few steps that your messages go through before even leaving your producer. They are all done on the Kafka client. The Kafka client is far more complex than I thought, it's not just an http call.

- ##### Serialize the message
  - during the serialization process, you should keep an eye to `key.serializer` and `value.serializer`.
  - keep in mind that the broker only knows how to deal with _bytes_.

- ##### Partition
  - done either by _key_ or
  - _sticky_, sending a bunch of messages to the same partition
  - make sure things are balanced.

- ##### Batching
  - it's nice to group messages together, in batches, to reduce your network traffic
  - `\batch.size` and `linger.ms` are high marks: when any of these two value is reached, the batch will leave your producer
  - keep also an eye on `buffer.memory`, this mark could be hit before the others if you have large messages.

- ##### Compression
  - it's optional
  - personal note: given the huge advancing in the compression algorithms given by Zstd in term of both speed and compression, I would probably always turn it on.


- ##### Request
It's eventually time to send the message to the broker. A few parameters to keep in mind:
  - `max.request.size`, which gives you the maximum size of the request batches. So if you have large batches the broker will reject them
  - `request.required.acks`, do you want to receive an ack on your requests? All the time? By default it's set to _all_
  - `max.in.flight.requests.per.connection`, connected to the _acks_ parameter. It answers to the question "how many requests can I send at the same time before I need to wait for a ack?"
  - `enable.idempotence`, which makes sure you can send the same request multiple times. Keep in mind also `transaction.id`
  - `request.timeout.ms`

### On the broker

Our messages have arrived on the broker, and they will go through several components/steps.

1. ##### Socket Receive buffer
Initial request landing zone Awaiting processing

2. ##### Network broker
  - Forms the official produce request and adds the request to request queue
  - parameter `num.network.threads` and monitor with `NetworkProcessorAvgIdlePercent`

3. ##### Requests queue
  - where messages await processing by I/O Threads
  - it's important to monitor the queue size and the average time a request waits

4. ##### I/O Threads
  - messages are finally ready to be processed
  - data validation
  - it's the only thing doing some real work <i class="fas fa-hammer fa-sm" style="color: gray;"></i>

5. ##### Page cache and to disk
  - It's expensive to write to disk, so you usually write to disk asynchronously. Most of the time you just keep the in Page cache.
  - You can also configure Tiered Storage, to offload some data to a cheaper storage

6. ##### Purgatory
  - it's when you actually replicate the data to the other brokers
  - hold requests until data has been replicated as per the configuration
  - hierarchical Timing Wheel

7. ##### Response Queue
  - a queue before going to the network thread
  - it's not configurable, but you can monitor it with `ResponseQueueSize` and `ResponseQueueTimeMs`

8. ##### Network Thread
  - places the requests on the send buffer

9. ##### Socket Send Buffer
  - The final interval on the broker
  - parameter `socket.send.buffer.bytes`
  - to be monitored using `ResponseSendTimeMs` 

10. ##### And back to the producer!

The talk was delivered by [<i class="fa-brands fa-twitter fa-sm" style="color: gray;"/>Danica Fine](https://twitter.com/TheDanicaFine), developer advocate at Confluent. Unfortunately it was not recorded, but you can find the [same talk](https://www.youtube.com/watch?v=DkYNfb5-L9o) at other conferences. The slides are [available on Slideshare](https://www.slideshare.net/HostedbyConfluent/a-kafka-clients-request-there-and-back-again-with-danica-fine).
<!-- {% include embed/youtube.html id='DkYNfb5-L9o' %}{: .shadow .w-75} -->
