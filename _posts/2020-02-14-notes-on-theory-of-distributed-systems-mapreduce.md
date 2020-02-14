---
layout: post
title: "Notes on theory of Distributed Systems - MapReduce"
description: "My notes from MIT 6.824 - distributed systems course - Lecture 1 which covers MapReduce, a classic paper from Google"
modified: 2020-02-12 12:41:33 +0530
permalink: /notes-on-theory-of-distributed-systems-mapreduce/
#category: []
tags: [distributed-systems, mapreduce]
image:
  feature: 
  credit: 
  creditlink: 
comments: true
share: 
---

[MapReduce: Simplified Data Processing on Large Clusters]:https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf


## Background
So I am going through MIT 6.824 - distributed systems course, and, lec 1 covers MapReduce.
<br/>

These are my notes from Google's classic paper - [MapReduce: Simplified Data Processing on Large Clusters].


## Introduction
MapReduce is a programming model and an associated implementation **for processing and generating large data sets.**
<br/>

Users specify -
*   a **map function** that processes a key/value pair to **generate** a set of intermediate key/value pairs, and
*   a **reduce function** that **merges** all intermediate values associated with the same intermediate key.

Many real world tasks are expressible in this model, as shown in the paper.

<br/>
  
__**Advantage**__
1.  The **run-time system takes care** of the details of partitioning the input data, scheduling the program’s execution across a set of machines, handling machine failures, and managing the required inter-machine communication.
2.  This allows **programmers** without any experience with parallel and distributed systems to **easily utilize the resources of a large distributed system..**

**" a new abstraction that allows us to express the simple computations we were trying to perform but hides the messy details of parallelization, fault-tolerance, data distribution and load balancing in a library."**

<br/>
inspired by the map and reduce primitives present in Lisp and many other functional languages.

## Programming Model
**map (k1,v1) → list(k2,v2)**

{% highlight java %}
    map(String key, String value):
        // key: document name
        // value: document contents
        for each word w in value:
             EmitIntermediate(w, "1"); 
{% endhighlight %}

**reduce (k2,list(v2)) → list(v2)**

{% highlight java %}
       reduce(String key, Iterator values):
    		    // key: a word
    		    // values: a list of counts
    		    int result = 0;
    		    for each v in values:
    		         result += ParseInt(v);
    		    Emit(AsString(result));
{% endhighlight %}

In addition, the user writes code to fill in a **mapreduce specification object with the names of the input and output files**, and optional tuning parameters. The user then invokes the MapReduce function, passing it the specification object.

## Applications

 - **Distributed Grep**

  - **Count of URL Access Frequency:** The map function processes logs of web page requests and outputs <URL, 1>. The reduce function adds together all values for the same URL and emits a <URL, total count> pair. 
 
  - **Reverse Web-Link Graph:** The map function outputs h target, source i pairs for each link to a target URL found in a page named source. The reduce function concatenates the list of all source URLs associated with a given target URL and emits the pair: <target, list(source)>

- **Term-Vector per Host:** A term vector summarizes the most important words that occur in a document or a set of documents as a list of <word, frequency> pairs. The map function emits a <hostname, term vector> pair for each input document (where the hostname is extracted from the URL of the document). The reduce function is passed all per-document term vectors for a given host. It adds these term vectors together, throwing away infrequent terms, and then emits a final <hostname, term vector> pair.
    
- **Inverted Index:** The map function parses each document, and emits a sequence of hword, document IDi pairs. The reduce function accepts all pairs for a given word, sorts the corresponding document IDs and emits a hword, list(document ID)i pair. The set of all output pairs forms a simple inverted index. It is easy to augment this computation to keep track of word positions.
  
![](https://www.evernote.com/shard/s7/res/1949e8d9-ff89-4fdb-aa8d-29e3e68825b7.jpg?search=mapreduce)

  
## Implementation

An implementation targeted to the computing environment in wide use at Google: **large clusters of commodity PCs connected together with switched Ethernet.**

### 1. Execution Overview
 1. The Map invocations are distributed across multiple machines by automatically **partitioning the input data into a set of M splits. **The input splits can be processed in parallel by different machines.
 2. Reduce invocations are distributed by **partitioning the intermediate key space into R pieces using a partitioning function** (e.g., hash(key) mod R). The number of partitions (R) and the partitioning function are specified by the user. 

**Map phase**
1.  The MapReduce library in the user program first **splits the input files into M pieces of typically 16 megabytes to 64 megabytes** (MB) per piece (controllable by the user via an optional parameter). It then starts up many copies of the program on a cluster of machines.
2.  One of the copies of the program is special – the master. The rest are workers that are assigned work by the master. **There are M map tasks and R reduce tasks to assign.** The master picks idle workers and assigns each one a map task or a reduce task.
3.  A worker who is assigned a **map task reads the contents of the corresponding input split.** It parses key/value pairs out of the input data and passes each pair to the user-defined Map function. The **intermediate key/value pairs** produced by the Map function **are buffered in memory.**

**Shuffle phase**

4.  **Periodically, the buffered pairs are written to local disk, partitioned into \*\*\*R regions\*\*\* by the partitioning function.**
5. **The locations of these buffered pairs on the local disk are passed back to the master, who is responsible for forwarding these locations to the reduce workers.**
6.  When a reduce worker is notified by the master about these locations, it uses remote procedure calls to read the buffered data from the local disks of the map workers. When a reduce worker has read all intermediate data, **it sorts it by the intermediate keys so that all occurrences of the same key are grouped together.** The sorting is needed because typically many different keys map to the same reduce task. **If the amount of intermediate data is too large to fit in memory, an external sort is used.**

**Reduce phase**
6.  The reduce worker iterates over the sorted intermediate data and for each unique intermediate key encountered, it passes the key and the corresponding set of intermediate values to the user’s Reduce function. The **output of the Reduce function is appended to a final output file** for this reduce partition.

**Finally**
7.  When all map tasks and reduce tasks have been completed, the master wakes up the user program. At this point, the MapReduce call in the user program returns back to the user code.  

> **reduce -> creates one file**
<br/>
> **map -> creates R files (one per reduce task)**

9. After successful completion, the output of the mapreduce execution is available in the R output files (one per reduce task, with file names as specified by the user).**

10. Typically, users do not need to combine these R output files into one file – they often pass these files as input to another MapReduce call, or use them from another distributed application that is able to deal with input that is partitioned into multiple files. 

###  2. Master Data Structures
The master keeps several data structures. For each map task and reduce task -
- it stores the **state (idle, in-progress, or completed)**, and
- the **identity of the worker machine** (for non-idle tasks).
 - for each completed map task, the master stores the **locations and sizes of the R intermediate file regions produced by the map task**. Updates to this location and size information are received as map tasks are completed. The information is pushed incrementally to workers that have in-progress reduce tasks.

### 3. Fault Tolerance
**Worker Failure**
 - The master pings every worker periodically. If no response is received from a worker in a certain amount of time, the master marks the worker as failed.  
 - **Any map tasks completed by the worker are reset back to their initial idle state,** and therefore become eligible for scheduling on other workers. 

> - Completed map tasks are re-executed on a failure **because their output is stored on the local disk(s) of the failed machine and is  therefore inaccessible.**
> - Completed reduce tasks **do not need to be re-executed** since their output is stored in a global file system.
> -  When a map task is executed first by worker A and then later executed by worker B (because A failed), all workers executing reduce tasks are notified of the re-execution. Any reduce task that has not already read the data from worker A will read the data from worker B.
- Similarly, any map task or reduce task in progress on a failed worker is also reset to idle and becomes eligible for rescheduling.
    
**Master Failure**
- **It is easy to make the master write periodic checkpoints of the master data structures described above. If the master task dies, a new copy can be started from the last checkpointed state.** 
- However, given that there is only a single master, its failure is unlikely; therefore our current implementation aborts the MapReduce computation if the master fails. Clients can check for this condition and retry the MapReduce operation if they desire.

- When the user-supplied **map and reduce operators are deterministic functions of their input values**, our distributed implementation produces the same output as would have been produced **by a non-faulting sequential execution** of the entire program.
 
- We **rely on atomic commits of map and reduce task** outputs to achieve this property.

- Each in-progress task writes its output to private temporary files. **A reduce task produces one such file, and a map task produces R such files (one per reduce task).**
    
- When a map task completes, the worker sends a message to the master and includes the names of the R temporary files in the message. Master records the names of R files in a master data structure.
    
- **When a reduce task completes, the reduce worker atomically renames its temporary output file to the final output file.** If the same reduce task is executed on multiple machines, multiple rename calls will be executed for the same final output file. We rely on the atomic rename operation provided by the underlying file system to guarantee that the final file system state contains just the data produced by one execution of the reduce task.

### 4. Locality
- Conserve network bandwidth by taking advantage of the fact that the **input data (managed by GFS \[8\]) is stored on the local disks of the machines that make up our cluster.**
- GFS divides each file into 64 MB blocks, and **stores several copies of each block (typically 3 copies)** on different machines.
- The **MapReduce master** takes the location information of the input files into account and **attempts to schedule a map task on a machine that contains a replica of the corresponding input data.** Failing that, it attempts to schedule a map task near a replica of that task’s input data (e.g., on a worker machine that is on the same network switch as the machine containing the data).

### 5. Task Granularity
- subdivide the map phase into M pieces and the reduce phase into R pieces.
- Ideally, **M and R should be much larger than the number of worker machines.**
    * Having each worker perform many different tasks improves dynamic load balancing,
    * and also speeds up recovery when a worker fails
 - Bounds on M and R
- **master must make O(M + R) scheduling decisions and keeps O(M ∗ R) state in memory**
 - **R is often constrained by users because the output of each reduce task ends up in a separate output file**
- In practice, we tend to choose M so that each individual task is roughly 16 MB to 64 MB of input data (so that the locality optimization described above is most effective), and we make R a small multiple of the number of worker machines we expect to use. We often perform MapReduce computations with M = 200, 000 and R = 5, 000, using 2,000 worker machines.

###  6. Backup Tasks
One of the common causes that lengthens the total time taken for a MapReduce operation is a **“straggler”: a machine that takes an unusually long time to complete one of the last few map or reduce tasks in the computation.**

__Reasons__
- a machine with a bad disk may experience frequent correctable errors that slow its read performance from 30 MB/s to 1 MB/s
- The cluster scheduling system may have scheduled other tasks on the machine, causing it to execute the MapReduce code more slowly due to competition for CPU, memory, local disk, or network bandwidth.
    
__Resolution__
- When a MapReduce operation is close to completion, the **master schedules backup executions of the remaining in-progress tasks.** **The task is marked as completed whenever either the primary or the backup execution completes.** We have tuned this mechanism so that it typically increases the computational resources used by the operation by no more than a few percent. We have found that this significantly reduces the time to complete large MapReduce operations.
    
### 7. Refinements/Extensions
__a. Partitioning Function__

The users of MapReduce specify the number of reduce tasks/output files that they desire (R). Data gets partitioned across these tasks using a partitioning function on the intermediate key. **A default partitioning function is provided that uses hashing (e.g. “hash(key) mod R”).**

**In some cases, however, it is useful to partition data by some other function of the key. For example, sometimes the output keys are URLs, and we want all entries for a single host to end up in the same output file. To support situations like this, the user of the MapReduce library can provide a special partitioning function. For example, using “hash(Hostname(urlkey)) mod R” as the partitioning function causes all URLs from the same host to end up in the same output file.

__b. Ordering Guarantees__

**We guarantee that within a given partition, the intermediate key/value pairs are processed in increasing key order.**  This ordering guarantee makes it easy to generate a sorted output file per partition, which is useful when the output file format needs to support efficient random access lookups by key, or users of the output find it convenient to have the data sorted.

__c. Combiner Function__

We allow the user to specify an optional Combiner function that does **partial merging of data** before it is sent over the network to Reduce function.

e.g. Since word frequencies tend to follow a Zipf distribution, each map task will produce hundreds or thousands of records of the form . All of these counts will be sent over the network to a single reduce task and then added together by the Reduce function to produce one number.

**The Combiner function is executed on each machine that performs a map task**. Typically the **same code is used to implement both the combiner and the reduce functions.**

 The only difference between a reduce function and a combiner function is how the MapReduce library handles the output of the function.
- The output of a reduce function is written to the final output file.
- The output of a **combiner function is written to an intermediate file that will be sent to a reduce task.**
    
**Partial combining significantly speeds up certain classes of MapReduce operations.**

__d. Input and Output Types__
The MapReduce library provides support for reading input data in several different formats.
 1. “text” - each line as a key/value pair 
 2. reader interface - add support for a new input type 
 3. easy to define a reader that **reads records from a database, or from data structures mapped in memory**

__e. Side Effects__
- users of MapReduce have found it convenient to **produce auxiliary files as additional outputs from their map and/or reduce operators.**
- **rely on the application writer to make such side-effects atomic and idempotent.**
    
__f. Skipping Bad Records__
- Certain records can crash Map & Reduce functions
- **prevent** a MapReduce operation from completing.

Can fix bug but sometimes not feasible.

Solution - provide an optional mode of execution where the **MapReduce library detects which records cause deterministic crashes and skips these records** in order to make forward progress.

Each worker process installs a signal handler that catches segmentation violations and bus errors. Before invoking a user Map or Reduce operation, the MapReduce library stores the sequence number of the argument in a global variable. If the user code generates a signal, the signal handler sends a “last gasp” UDP packet that contains the sequence number to the MapReduce master. When the master has seen more than one failure on a particular record, it indicates that the record should be skipped when it issues the next re-execution of the corresponding Map or Reduce task.

__g. Local Execution__

To help facilitate debugging, profiling, and small-scale testing, we have developed an alternative implementation of the MapReduce library that sequentially executes all of the work for a MapReduce operation on the local machine.

### 8. Status Information
The master runs an internal HTTP server and exports a set of status pages for human consumption, which shows
the progress of the computation, such as - 

 1. how many tasks have been completed,
  2.  how many are in progress,
  3.  bytes of input,
  4.  bytes of intermediate data,
  5.  bytes of output,
  6.  processing rates, etc.
  7.  The pages also contain links to the standard error and standard output files generated by each task

### 9. Counters
The MapReduce library provides a counter facility to count occurrences of various events. To use this facility, user code creates a named counter object and then increments the counter appropriately in the Map and/or Reduce function.

    Counter\* uppercase;
    uppercase = GetCounter("uppercase");
    
    map(String name, String contents):
         for each word w in contents:
             if (IsCapitalized(w)):
                 uppercase->Increment();
             EmitIntermediate(w, "1"); 

 - **The counter values from individual worker machines are periodically propagated to the master (piggybacked on the ping response). The master aggregates the counter values from successful map and reduce tasks and returns them to the user code when the MapReduce operation is completed.**
 
- Some counter values are automatically maintained by the MapReduce library, such as the number of input key/value pairs processed and the number of output key/value pairs produced.  

 __Usage__
- Users have found the counter facility useful for sanity checking the behavior of MapReduce operations. For example, in some MapReduce operations, the user code may want to ensure that the number of output pairs produced exactly equals the number of input pairs processed,
- or that the fraction of German documents processed is within some tolerable fraction of the total number of documents processed.

## Experience
It has been used across a wide range of domains within Google, including:
- large-scale machine learning problems,
- clustering problems for the Google News and Froogle products,
- extraction of data used to produce reports of popular queries (e.g. Google Zeitgeist),
- extraction of properties of web pages for new experiments and products (e.g. extraction of geographical locations from a large corpus of web pages for localized search), and
- large-scale graph computations.
  

## Large-Scale Indexing
One of our most significant uses of MapReduce to date has been a **complete rewrite of the production indexing system** that produces the data structures used for the Google web search service. The indexing system takes as input a large set of documents that have been retrieved by our crawling system, stored as a set of GFS files. The raw contents for these documents are more than 20 terabytes of data. The indexing process runs as a sequence of five to ten MapReduce operations.

Using MapReduce (instead of the ad-hoc distributed passes in the prior version of the indexing system) has provided  **several benefits:**
- **code is simpler, smaller, and easier to understand** - The indexing code is simpler, smaller, and easier to understand, because the code that deals with fault tolerance, distribution and parallelization is hidden within the MapReduce library.
    
- **good performance** - The performance of the MapReduce library is good enough that we can keep conceptually unrelated computations separate, instead of mixing them together to avoid extra passes over the data. This makes it easy to change the indexing process.
    
- **easy to operate** - The indexing process has become much easier to operate, because most of the problems caused by machine failures, slow machines, and networking hiccups are dealt with automatically by the MapReduce library without operator intervention.
    
- **horizontally scalable** - Furthermore, it is easy to improve the performance of the indexing process by adding new machines to the indexing cluster.
    
## Related Work
- Our locality optimization draws its inspiration from techniques such as active disks \[12, 15\], where computation is pushed into processing elements that are close to local disks, to reduce the amount of data sent across I/O subsystems or the network. We run on commodity processors to which a small number of disks are directly connected instead of running directly on disk controller processors, but the general approach is similar.
- Our backup task mechanism is similar to the eager scheduling mechanism employed in the Charlotte System \[3\].
    
## Lessons learnt for Google
**We have learned several things from this work.**
1.  First, **restricting the programming model** makes it easy to parallelize and distribute computations and to make such computations fault-tolerant.
2.  Second, **network bandwidth is a scarce resource.** A number of optimizations in our system are therefore targeted at reducing the amount of data sent across the network: the locality optimization allows us to read data from local disks, and writing a single copy of the intermediate data to local disk saves network bandwidth.
3.  Third, **redundant execution can be used to reduce the impact of slow machines**, and to handle machine failures and data loss.
