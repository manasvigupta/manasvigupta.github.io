---
layout: post
title: "Slow Log4j 1.2 performance with Caller location logging"
description: "Kids, don't enable caller logging in Production!"
modified: 2014-04-13 00:02:36 +0530
#category: [blog]
tags: [learning, programming]
#image:
#  feature: texture-feature-06.jpg
#  credit: coffee #17 by Mariantonietta Continenza
#  creditlink: http://www.flickr.com/photos/42897741@N05/3987243989/
comments: true
share: 
---

Quick question .. which of these log statements is more **useful** in debugging an issue?

{% highlight css %}
0    [main] INFO  MyApp - Entering application.
{% endhighlight %}

##### OR

{% highlight css %}
0    [main] INFO  MyApp com.parent.LoggerTest.main(LoggerTest.java:10)  - Entering application.
{% endhighlight %}


Of course, 2nd log statement is more useful, because, it provides **caller information** i.e. line number, method name, class name and file number.

<br/>

### Log4j and caller location information

Log4j 1.2 provides you an easy conversion [pattern] for this -


*  **%C** -  Used to output the fully qualified class name of the caller issuing the logging request
*  **%F** -  Used to output the file name where the logging request was issued.
*  **%L** -  Used to output the line number from where the logging request was issued.
*  **%M** -  Used to output the method name where the logging request was issued.
*  **%l** -  Used to output location information of the caller which generated the logging event.

However, for each case, log4j adds following [warning in javadoc.]

**WARNING**   Generating caller location information is extremely slow and should be avoided unless execution speed is not an issue.
{: .notice}


<br/>
Unfortunately, beyond this warning, it does not explain **why** logging caller info is slow.  To find out, I dug through log4j source code.

---

### My Findings

#### Summary

For **each** log statement using _any of above pattern_, log4j creates **new** instance of [Throwable]  which takes a snapshot of the runtime stack. Next, it parses this stack trace to gather details about line number, method name and class information. 

<br/>
Unfortunately, creating new Throwable is performance intensive as JVM has to pause current thread to gather stack information. 

<br/>
To know more, read this [blog post.]

> The most costly part of exception processing on the JVM is creating the exception, not throwing it. Exception creation involves a native method named **`Throwable.fillInStackTrace()`**, which looks down the stack (before the actual throw) and puts a whole backtrace into the exception object. It’s great for debugging, but a terrible waste if all you want to do is pop a frame or two.


#### Details 

To retrieve caller details, log4j executes in following sequence -

<figure>
    <a href="https://github.com/manasvigupta/manasvigupta.github.io/raw/master/images/log4j_sequence.png"><img src="/images/log4j_sequence.png"></a>
</figure>

<br/>
Below code snippets show how this part actually works.


<br/>
First, **`org.apache.log4j.helpers.PatternParser`** determines if pattern for Location logging  (i.e. _%C, %F, %L, %M, %l_) is present or not. If present, it calls **`LoggingEvent.getLocationInformation()`**, which creates a new Throwable() and instantiates **`LocationInformation`** with it. You can see [full code here - 1].

{% highlight java linenos %}
package org.apache.log4j.spi;

class LoggingEvent {
....

    public LocationInfo getLocationInformation() {
       if(locationInfo == null) {
         locationInfo = new LocationInfo(new Throwable(), fqnOfCategoryClass);
        }
       return locationInfo;
     }
 ...
 }
 
{% endhighlight %}


Next, **`LocationInfo`** class parses stacktrace to retrieve caller information. You can see [full code here - 2].

{% highlight java linenos %}

package org.apache.log4j.spi;

class LocationInfo {
....
....
    public LocationInfo ( Throwable t, String fqnOfCallingClass ) {
        ...
        ...
    
      String s;
      // Protect against multiple access to sw.
      synchronized( sw) {
      /* Manasvi's notes - saves stacktrace from  Throwable into PrintWriter */
       t .printStackTrace(pw);
      /* Manasvi's notes - converts this stacktrace into String */
       s = sw.toString();
        sw.getBuffer().setLength(0);
      }
      ...

      // Given the current structure of the package, the line
      // containing "org.apache.log4j.Category." should be printed just
      // before the caller.

      // This method of searching may not be fastest but its safer
      // than counting the stack depth which is not guaranteed to be
      // constant across JVM implementations.
      ibegin = s.lastIndexOf(fqnOfCallingClass);
      if(ibegin == -1)
        return;

      ibegin = s.indexOf(Layout. LINE_SEP, ibegin);
      if(ibegin == -1)
        return;
      ibegin+= Layout. LINE_SEP_LEN;

      // determine end of line
      iend = s.indexOf(Layout. LINE_SEP, ibegin);
      if(iend == -1)
        return;
        ....
        ....
      }
      // everything between is the requested stack item
      this. fullInfo = s.substring(ibegin, iend);
    }
.....
.....    
{% endhighlight %}



<br/>
Finally, below is a sample Stack trace from my debugging, clearly highlighting call graph of Log4j classes.

{% highlight css linenos%}
java.lang.Throwable
     at org.apache.log4j.spi.LoggingEvent.getLocationInformation(LoggingEvent.java:247)
     at org.apache.log4j.helpers.PatternParser$LocationPatternConverter.convert(PatternParser.java:483)
     at org.apache.log4j.helpers.PatternConverter.format(PatternConverter.java:65)
     at org.apache.log4j.PatternLayout.format(PatternLayout.java:502)
     at org.apache.log4j.WriterAppender.subAppend(WriterAppender.java:302)
     at org.apache.log4j.WriterAppender.append(WriterAppender.java:160)
     at org.apache.log4j.AppenderSkeleton.doAppend(AppenderSkeleton.java:251)
     at org.apache.log4j.helpers.AppenderAttachableImpl.appendLoopOnAppenders(AppenderAttachableImpl.java:66)
     at org.apache.log4j.Category.callAppenders(Category.java:206)
     at org.apache.log4j.Category.forcedLog(Category.java:391)
     at org.apache.log4j.Category.info(Category.java:666)
     at com.parent.LoggerTest.main(LoggerTest.java:10)
{% endhighlight %}



[pattern]:http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html

[warning in javadoc.]:http://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html

[blog post.]:https://blogs.oracle.com/jrose/entry/longjumps_considered_inexpensive

[Throwable]: http://docs.oracle.com/javase/7/docs/api/java/lang/Throwable.html

[full code here - 1]: http://grepcode.com/file/repo1.maven.org/maven2/log4j/log4j/1.2.15/org/apache/log4j/spi/LoggingEvent.java/#LoggingEvent.getLocationInformation%28%29

[full code here - 2]: http://grepcode.com/file/repo1.maven.org/maven2/log4j/log4j/1.2.15/org/apache/log4j/spi/LocationInfo.java/#LocationInfo.%3Cinit%3E%28java.lang.Throwable%2Cjava.lang.String%29
