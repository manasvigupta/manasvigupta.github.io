---
layout: post
title: "Getting started with contributing to Apache Kafka (Part 1): Build and run Kafka from source code"
description: "In this blog post we will setup tools necessary to run Apache Kafka from source code"
modified: 2018-03-31 20:41:33 +0530
permalink: /getting-started-with-contributing-to-apache-kafka-part1-build-and-run-kafka-from-source-code/
#category: []
tags: [apache-kafka]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

If you always wanted to contribute to Apache Kafka, but, didn't know where to begin, then, you have come to the right place.

<br/>
In this two-part series, I will guide you through this process (based on my learnings in the process of contributing to Apache Kafka).

<br/>
**Part 1 (current post)** - Install tools needed to run Kafka from source code.
<br/><br/>
**Part 2 (upcoming blog post)** - Identify starter (i.e. beginner) bugs in Kafka, fix locally & test. Also, we will look at Kafka code review & code submission process.

<br/>
So, lets begin with our journey.

<br/>
In this post, we will be executing following steps - 
1. setup necessary software/tools
2. download apache kafka source code
3. run/debug kafka from source

For this setup, I am assuming that you are using **Ubuntu 16.04** or higher.

### 1. Install necessary software

#### Java 8
Execute following commands in terminal.
{% highlight text %}
> sudo add-apt-repository ppa:webupd8team/java
> sudo apt-get update
> sudo apt-get install oracle-java8-installer
> java -version
{% endhighlight %}

#### Gradle
Execute following commands in terminal to __install Gradle using SDK Man__:

{% highlight text %}
> curl -s "https://get.sdkman.io" | bash
> sdk install gradle
> gradle -version
{% endhighlight %}


#### Git
Execute following commands in terminal:
{% highlight text %}
> sudo add-apt-repository ppa:git-core/ppa
> sudo apt-get update
> sudo apt-get install git
> git --version
{% endhighlight %}

#### IntelliJ IDEA (Community edition)
Refer to [IntelliJ IDEA for Ubuntu 16.04] for detailed installation steps.

{% highlight text %}
> sudo snap install intellij-idea-community --classic --edge
> intellij-idea-community
{% endhighlight %}

Also make sure you have installed following Intellij plugins - **Scala, Gradle, JUnit**

### 2. Download Kafka source code
1. Create a new github account, if you don't have one already
2. Fork [Apache Kafka] github project
3. Using git, clone your forked project in your local m/c

{% highlight text %}
> git clone https://github.com/<your github id>/kafka.git
{% endhighlight %}


### 3. Build Kafka source code
Execute following commands in terminal:

{% highlight text %}
> cd kafka
> gradle
> ./gradlew jar
{% endhighlight %}

### 4. Import Kafka projects into Intellij IDEA
1. Start IntelliJ IDEA using - `> intellij-idea-community`
2. Click on **Import Project** and browse to Kafka source folder
3. Make sure you import by selecting - **Import using external model > Gradle**. Wait for import to complete.
4. Now, build project and __verify that build is successful__ (warnings can be ignored)

### 5. Run Zookeeper
Since Kafka depends on Zookeeper, we need to run it first. In a new terminal, execute following commands:

{% highlight text %}
> cd kafka
> bin/zookeeper-server-start.sh config/zookeeper.properties
{% endhighlight %}


### 6. Debug/Run Kafka in IntelliJ IDEA
1. In Intellij, click on **Run > Edit Configurations > + (i.e. Add New Configuration) > Application**
2. Enter values as per below screen grab & save configuration.

<figure>
    <a href="https://github.com/manasvigupta/manasvigupta.github.io/raw/master/images/intellij_run_config.png"><img src="/images/intellij_run_config.png"></a>
</figure>

3. Next, click on **Run > Run "Kafka"**
4. Intellij console will show log messages indicating that Kafka is running.
5. Also, in terminal where Zookeeper is running, you will see logs indicating that Kafka has connected to Zookeeper.

### 7. Testing Kafka setup
Create a new topic named "test" with a single partition and only one replica:

{% highlight text %}
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
{% endhighlight %}

We can now see that topic if we run the list topic command:
{% highlight text %}
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
{% endhighlight %}

Run the producer and then type a few messages into the console to send to the server.
{% highlight text %}
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
{% endhighlight %}

Kafka also has a command line consumer that will dump out messages to standard output.
{% highlight text %}
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
{% endhighlight %}

If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

### 8. la fin
That's it. You will now be able to run Kafka from source & debug it at your convinience.

<br/>
In __part 2__, we will learn to identify starter (i.e. beginner) bugs in Kafka, fix locally & test. Also, we will look at Kafka code review & change submission process.


[Apache Kafka]:https://github.com/apache/kafka
[IntelliJ IDEA for Ubuntu 16.04]:https://blog.jetbrains.com/idea/2017/11/install-intellij-idea-with-snaps/

