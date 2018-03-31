---
layout: post
title: "Getting started on contributing to Apache Kafka (Part 1): Setup tools and source code"
description: "In this blog post we will setup tools necessary to run Apache Kafka from source code"
modified: 2018-02-18 20:41:33 +0530
permalink: /getting-started-on-contributing-to-apache-kafka-part1-setup-tools-and-source-code/
#category: []
tags: [apache-kafka]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

If you always wanted to contribute to Apache Kafka, but, did not know where to begin, then, you have come to the right place.

<br/>
In this two-part series, I will walk you through the whole process (based on my learnings after I made contribution to Apache Kafka).

<br/>
**Part 1** - Download source code & install tools required to compile, run & debug Kafka source code.
<br/>
**Part 2 (upcoming blog post)** - Identify starter bugs, fix these & verify. Also, we will look at Kafka code review & submission process.

<br/>
So, lets begin with our journey.

<br/>
In this post, we will be execute following steps - 
1. setup necessary software/tools
2. download apache kafka source code
3. run/debug kafka from source

For this setup, I am assuming that you are using **Ubuntu 16.04** or higher.

### 1. Install necessary software

#### Java 8
{% highlight text %}
> sudo add-apt-repository ppa:webupd8team/java
> sudo apt-get update
> sudo apt-get install oracle-java8-installer
> java -version
{% endhighlight %}

Refer for more details - [Java8 installation]

#### Gradle
We will install Gradle using SDK Man.

{% highlight text %}
> curl -s "https://get.sdkman.io" | bash
> sdk install gradle
> gradle -version
{% endhighlight %}


#### Git
{% highlight text %}
> sudo add-apt-repository ppa:git-core/ppa
> sudo apt-get update
> sudo apt-get install git
> git --version
{% endhighlight %}

#### IntelliJ IDEA (Community edition)
Refer to [IntelliJ IDEA for Ubuntu 16.04] for installation.

{% highlight text %}
> sudo snap install intellij-idea-community --classic --edge
> intellij-idea-community
{% endhighlight %}

Also make sure you have installed following intellij plugins - **Scala, Gradle, JUnit**

### 2. Download Kafka source code
1. Create a new github account, if you don't have one already.
2. Fork [Apache Kafka github] project
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
4. Now, build project. Verify that build is successful (warning can be ignored)


### 5. Debug/Run Kafka in IntelliJ IDEA
Add steps to run

[Java8 installation]:http://tipsonubuntu.com/2016/07/31/install-oracle-java-8-9-ubuntu-16-04-linux-mint-18/
[Scala-IDE]:http://scala-ide.org
[Apache Kafka github]:https://github.com/apache/kafka
[IntelliJ IDEA for Ubuntu 16.04]:https://blog.jetbrains.com/idea/2017/11/install-intellij-idea-with-snaps/

