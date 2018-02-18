---
layout: post
title: "Getting started on contributing to Apache Kafka (Part 1): Setup tools and source code"
description: "In this blog post we will setup tools necessary to run Apache Kafka from source"
modified: 2018-02-18 20:41:33 +0530
permalink: /getting-started-on-contributing-to-apache-kafka-part1-setup-tools-and-source-code/
#category: []
tags: []
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

In this post, we will be doing these activities -
1. setup necessary software/tools
2. download apache kafka source code
3. run/debug kafka from source

For this setup, I am assuming that you are using Ubuntu 16.04 or hihger.

### Install necessary software

#### * Java 8
{% highlight text %}
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
java -version
{% endhighlight %}

Refer for more details - [Java8 installation]

#### * Gradle
We will install Gradle using SDK Man.

{% highlight text %}
curl -s "https://get.sdkman.io" | bash
sdk install gradle
gradle -version
{% endhighlight %}


#### * Git
{% highlight text %}
sudo add-apt-repository ppa:git-core/ppa
sudo apt-get update
sudo apt-get install git
git --version
{% endhighlight %}

#### * IntelliJ IDEA
Refer to [IntelliJ IDEA for Ubuntu 16.04] for installation.

{% highlight text %}
sudo snap install intellij-idea-community --classic --edge

intellij-idea-community
{% endhighlight %}

Make sure you have installed following plugins - Scala, Gradle, JUnit

### Setup Apache Kafka source code
* Create a new github account, if you don't have one already.
* Fork [Apache Kafka github] project
* Using git, clone this project in your local m/c

`git clone https://github.com/<your github id>/kafka.git`

### Build Kafka source
Execute following commands in terminal:

{% highlight text %}
cd kafka
gradle
./gradlew jar
{% endhighlight %}

### Import Kafka projects into Intellij IDEA
* Start IntelliJ IDEA using cmd - `intellij-idea-community`
* Click on "Import Project" and browse to Kafka source folder
* Make sure you import by selecting - "Import using external model > Gradle"
* Wait for import to complete
* Now, build project
* Verify that build is successful (you may see warning which can be ignored)


### Debug and Run Kafka in IntelliJ IDEA
Add steps to run

[Java8 installation]:http://tipsonubuntu.com/2016/07/31/install-oracle-java-8-9-ubuntu-16-04-linux-mint-18/
[Scala-IDE]:http://scala-ide.org
[Apache Kafka github]:https://github.com/apache/kafka
[IntelliJ IDEA for Ubuntu 16.04]:https://blog.jetbrains.com/idea/2017/11/install-intellij-idea-with-snaps/

