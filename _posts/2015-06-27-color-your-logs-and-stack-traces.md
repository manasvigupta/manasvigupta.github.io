---
layout: post
title: "Color Your Logs And Stack Traces"
description: "How to color your logs using tail and AWK on command line"
modified: 2015-06-27 00:02:36 +0530
#category: [blog]
tags: [learning, programming]
#image:
#  feature: texture-feature-06.jpg
#  credit: coffee #17 by Mariantonietta Continenza
#  creditlink: http://www.flickr.com/photos/42897741@N05/3987243989/
comments: true
share: 
---

Logs are not the only way we monitor our systems, but they play an important role, both for troubleshooting purposes and to check the general health of an application.

<br/>
One of the problems we had was the readability of logs. When you know what you’re looking for in a log, it is easy to search for an exception. But when you don’t know what is wrong, going through gigabytes of logs in white text on a black background can be very time consuming and onerous.

<br/>

My solution is to add colors to our logs to facilitate the readability. Wouldn’t it be great if every level (i.e., info, warning, error) message had a different color? What if we could have different colors in our stack trace to highlight different exceptions?

Before jumping into the details of the implementation, let’s first understand how the coloring works in Unix. An easy way to understand it is to type the following in a Unix terminal:

{% highlight css %}
echo -e "\033[31m  Exception log  \033[39m"
{% endhighlight %}

The result is:

Exception log

What happened?

<br/>
First, it’s important to know that every color has a corresponding ANSI color code:

{% highlight css %}
Black = 30
Red = 31
Green = 32
Yellow = 33
…
Default = 39
{% endhighlight %}

<br/>
For the color codes to take effect, they need to be preceded by the “start” escape character, \033[, and be followed by the “end” escape character, m.

<br/>
Therefore, to display a message in red, we would type the following:

{% highlight css %}
\033[31m
{% endhighlight %}

<br/>
By using this information, I created following AWK script to highlight logs in console while we "tail" our logs.

{% highlight css %}
tail -f async.log | awk '

  BEGIN { RED="\033[0;31m"; 
          BLACK="\033[39m"; 
          YELLOW="\033[33m"; 
          LIGHT_GREEN="\033[1;32m"
  }

  /WARN/ {print YELLOW $0; next}
  /INFO/ {print LIGHT_GREEN $0; next}
  /ERROR/ {p=1} p && /INFO|WARN|DEBUG/ {p=0};p {print RED $0; next}
  // {print}
'
{% endhighlight %}


Enjoy !


