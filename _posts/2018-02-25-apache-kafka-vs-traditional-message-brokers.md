---
layout: post
title: "Apache Kafka vs Traditional Message brokers"
description: "In this post we will explore differences between Apache Kafka and traditional message brokers"
modified: 2018-02-25 20:41:33 +0530
permalink: /apache-kafka-vs-traditional-message-brokers/
#category: []
tags: [apache-kafka]
#image:
#  feature: texture-feature-06.jpg
#  credit: coffee #17 by Mariantonietta Continenza
#  creditlink: http://www.flickr.com/photos/42897741@N05/3987243989/
image:
  feature: unsplash-image-9.jpg
  credit: Unsplash
  creditlink: https://unsplash.com
comments: true
share: 
---

Here are the key differences between Apache Kafka and traditional message brokers - 

<style>
table{
    border-collapse: collapse;
    border-spacing: 0;
    border:2px solid;
}
th{
    border:2px solid;
}
td{
    border:2px solid;
}
</style>

|  | Apache Kafka | Traditional Message Brokers |
|:--------:|:--------:|:-------:|
|----
| ***Design (client vs broker)*** | Kafka is __client-centric__, with the client taking over many of the functions of a traditional broker, such as fair distribution of related messages to consumers, in return for an extremely fast and scalable broker.  | The JMS model is very __broker-centric__, where the broker is responsible for the distribution of messages, and clients only have to worry about sending and receiving messages.   |
|----
| ***Retention and replay of messages*** | Possible   | Not possible   |
|----
| ***Scalability*** | Horizontally scalable (by adding more partitions)   | Not possible    |
|----
| ***Performance*** | Does not slow down with addition of consumers   | Both queue and topic performance degrades as the number of consumers rise on a destination  |
|----
| ***Publish options*** | Kafka unifies both publish-subscribe and point-to-point messaging under a single destination type— __topic__  | Publish-subscribe and point-to-point messaging are independent features  |
|----
| ***Journal*** | Each topic has a journal. There can be multiple topics.   | Messages from all queues are stored in the same journal   |
|----
| ***Message delivery failure*** | Client is responsible for replaying failed messages | Broker is responsible for re-delivery of messages   |
|----
| ***Message structure*** | Message is a key-value pair. The payload of the message is sent as the value. The key, on the other hand, is used primarily for partitioning purposes and should contain a business-specific key in order to place related messages on the same partition. | Message consists of metadata (headers and properties) and body (payload) |
|----
| ***Message integrity*** | Kafka includes checksums to detect corruption of messages in storage and has a comprehensive set of security features | ????? |
|----
| ***Message retention*** | Messages are not deleted on the broker once consumed, and the responsibility for working out what happens on failure lies with the consuming code itself | Broker would either delete a successfully processed message or re-deliver an unprocessed or failed one(assuming transactions were in play) |



