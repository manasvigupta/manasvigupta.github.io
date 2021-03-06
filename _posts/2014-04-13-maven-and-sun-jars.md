---
layout: post
title: "Add Sun jars to your Maven repository"
description: "How to add Sun jars to your maven repository"
modified: 2014-04-13 20:46:05 +0530
permalink: /add-sun-jars-to-your-maven-repo/
#category: []
tags: [programming, learning]
#image:
#  feature: texture-feature-04.jpg
#  credit: Texture Lovers
#  creditlink: http://texturelovers.com
comments: true
share: 
---

If you have used log4j 1.2 on your project via Maven, you may have see missng artifact error for following Sun jars - `com.sun.jmx:jmxri`, `com.sun.jdmk:jmxtools` and `javax.jms:jms`.


<br/>
Common solution to this problem is to exclude these dependencies, as suggested on [Stackoverflow].

{% highlight xml %}
<!-- update your pom.xml to exclude sun jars -->
<dependency>
<groupId>log4j</groupId>
<artifactId>log4j</artifactId>
<version>1.2.15</version>
<exclusions>
    <exclusion>
        <groupId>com.sun.jmx</groupId>
        <artifactId>jmxri</artifactId>
    </exclusion>
    <exclusion>
        <groupId>com.sun.jdmk</groupId>
        <artifactId>jmxtools</artifactId>
    </exclusion>
    <exclusion>
            <groupId>javax.jms</groupId>
            <artifactId>jms</artifactId>
    </exclusion>
</exclusions>
</dependency>
{% endhighlight %}

<br/>
But, what if you actually need these jars available locally in maven repository ? For e.g.  I needed Sun jars for compiling and debugging log4j source code. You can read more about it [here].


### Handling missing `com.sun.jmx:jmxri`, `com.sun.jdmk:jmxtools`

Download these jars from Oracle site and install manually in m2 repository using following commands -

{% highlight java %}
mvn install:install-file 
            -Dfile=C:\Users\magupta\Downloads\jmxtools.jar 
            -DgroupId=com.sun.jdmk 
            -DartifactId=jmxtools 
            -Dversion=1.2.1 
            -Dpackaging=jar
            
mvn install:install-file 
            -Dfile=C:\Users\magupta\Downloads\jmxtools.jar 
            -DgroupId=com.sun.jmx 
            -DartifactId=jmxri 
            -Dversion=1.2.1 
            -Dpackaging=jar
            
{% endhighlight %}

### Handling missing `javax.jms:jms`




[Stackoverflow]: http://stackoverflow.com/questions/9047949/missing-artifact-com-sun-jdmkjmxtoolsjar1-2-1
[here]: {% post_url 2014-04-14-do-not-enable-caller-location-logging-with-log4j %}


