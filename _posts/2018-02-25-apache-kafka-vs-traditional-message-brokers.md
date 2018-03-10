---
layout: post
title: "Apache Kafka vs Traditional Message brokers"
description: "Summary of key differences between Apache Kafka and Traditional message brokers (e.g JMS, ActiveMQ)"
modified: 2018-02-25 20:41:33 +0530
permalink: /apache-kafka-vs-traditional-message-brokers/
#category: []
tags: [apache-kafka]
#image:
#  feature: unsplash-image-9.jpg
#  credit: Unsplash
#  creditlink: https://unsplash.com
comments: true
share: 
---

In this post, we are going to look at some key differences between ***Apache Kafka*** and Traditional message brokers ***(e.g  JMS, ActiveMQ)***.

### 1. Design
__Apache Kafka :__ ***client-centric***, with the client taking over many of the functions of a traditional broker, such as fair distribution of related messages to consumers, in return for an extremely fast and scalable broker.

<br/>
__Traditonal message brokers :__ ***broker-centric***, where the broker is responsible for the distribution of messages, and clients only have to worry about sending and receiving messages.


### 2. Scalability

__Apache Kafka :__ ***horizontally scalable*** by adding more partitions
<br/>
__Traditonal message brokers :__ not possible to scale horizontally

### 3. Performance

__Apache Kafka :__ does not slow down with addition of new consumers
<br/>
__Traditonal message brokers :__ both queue and topic performance degrades as the number of consumers rise on a destination

### 4. Publish options

__Apache Kafka :__ unifies both publish-subscribe and point-to-point messaging under a single destination type— __topic__
<br/><br/>
__Traditonal message brokers :__ provides both ***publish-subscribe*** and ***point-to-point messaging*** for publishing messages

### 5. Journal

__Apache Kafka :__ each topic has a journal and there can be multiple topics.
<br/>
__Traditonal message brokers :__ messages from all queues are stored in the same journal

### 6. Message delivery failures

__Apache Kafka :__ client is responsible for replaying failed messages
<br/>
__Traditonal message brokers :__ broker is responsible for re-delivery of messages

### 7. Message structure

__Apache Kafka :__ message is a key-value pair. Payload of the message is sent as the value. Key, on the other hand, is used primarily for partitioning purposes and should contain a business-specific key in order to place related messages on the same partition.
<br/><br/>
__Traditonal message brokers :__ message consists of metadata (headers and properties) and body (payload)


### 8. Message integrity

__Apache Kafka :__ includes checksums to detect corruption of messages in storage and has a comprehensive set of security features
<br/><br/>
__Traditonal message brokers :__ no such option provided out-of-the box


### 9. Message retention

__Apache Kafka :__ messages are ***not*** deleted on the broker once consumed.
<br/><br/>
__Traditonal message brokers :__ broker would either delete a successfully processed message or re-deliver an unprocessed or failed one (assuming transactions were in play)

### 10. Replay of messages

__Apache Kafka :__ since Kafka retains messages, it is possible for clients to retrieve any message, providing option for re-processing of all messages.
<br/><br/>
__Traditonal message brokers :__ not possible, since, broker does not re-deliver messages that have been processed successfully by client


