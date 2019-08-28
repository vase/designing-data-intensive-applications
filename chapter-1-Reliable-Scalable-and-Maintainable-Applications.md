# Chapter 1. Reliable, Scalable and Maintainable Applications

## Summary
> This chapter explores some of the fundamental ways of thinking about data-intensive applications. These principles will guide reader through the rest of the book, where technical details are the focus.

An application has to meet various requirements to be useful:
* *Functional requirements* - What the software should do
* *Non-functional requirements*
  * Security
  * Reliability
  * Compliance
  * Scalability
  * Compatibility
  * Maintainability
  * [More](https://en.wikipedia.org/wiki/Non-functional_requirement)

This chapter discusses *Reliability, Scalability and Maintainability* in detail.

**Reliability** means making systems work correctly, even when hardware/software/human faults occur.

**Scalability** means having strategies for keeping performance good, even when load increases. In order to discuss scalability, we first need ways of describing load and performance quantitatively. *Case study: Twitter's home timelines*

**Maintainability** in essence is about making life better for engineering and operation teams who need to work with the system. Two main motives:
* *Good abstractions*: reduce complexity and make the system easier to modify.
* *Good operability*: good visibility into system's health and having effective ways of managing it.
---
Many applications today are *data-intensive*, as opposed to *compute-intensive*.
Raw CPU power is rarely a limiting factor.
Bigger problems are the *amount, complexity and speed of change of data*

Typical data-intensive application standard building block, referred to as *data systems*:
- Databases
- Caches
- Search Indexes
- Stream Processing
- Batch Processing
- Queues etc.

Options under each data system category have characteristics and approaches that leads to difficulties in figuring out which tools and which approaches that meet our requirements. It can be hard to combine tools to do what a single tool cannot do alone.

Therefore, the principles and practicalities of data systems, and how we can use them to build data-intensive applications is very important.

---
## Thinking About Data Systems

### *Why the term data system?*
We typically think of database, queues, caches, etc as being very different categories of tools. Despite the superficial similarity that data systems store data for some time, they have different access patterns, thus different performance characteristics and implementations.

However, many new tools for data storage and processing are optimized for a variety of different use cases, and no longer neatly fit into traditional categories. E.g:
- Redis: datastores that are also used as message queues
- Apache Kafka: message queues with database-like durability guarantees

Secondly, demanding or wide-ranging application requirements means a single tool can no longer meet all of its data processing and storage needs. Solution is to break it into tasks that can be performed efficiently on a single tool, and stitch the tools together using application code.

Figure 1-1: One possible architecture for a data system that combines several components

![Figure 1-1](https://github.com/vasetech/designing-data-intensive-applications/blob/master/assets/figure-1-1.png?raw=true "Architecture with several data systems")

When you combine several tools to provide a service, the service's API hides implementation details from clients. You have essentially created a new, special-purpose data system from smaller, general-purpose components. Your composite data system may provide certain guarantees. E.g: correctly invalidated/updated cache on writes for consistent results

You are now not only an application developer, but also a **data system designer**.

There are many factors that may influence the design of a data system, including skills, legacy dependencies, deadline, organisation policy, regulation, etc.

In this book, we explore **ways of thinking** about the three concerns that are important in most software systems - *Reliability, Scalability and Maintainability*

---
## Reliability
> The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).

##### Terminology
* *Fault*: things that can go wrong, one component of the system deviating from its **spec**
* *Fault-tolerant/Resilient*: systems that anticipate faults and can cope with them
* *Failure*: not the same as fault, is when the system as a whole stops providing the required **service to the user**.

Many critical bugs are actually due to poor error handling. You can ensure the fault-tolerance machinery is continually exercised and tested by deliberately inducing faults.

This book mostly deals with the kinds of faults that can be cured, fault tolerant rather than fault prevention.

### Hardware Faults
*Examples of hardware faults*: hard disks crash, faulty RAM, power grid blackout, someone unplugging the wrong network cable

First response to reduce hardware failure rates is to add redundancy to the individual hardware components. E.g: disks in RAID, servers with double power supplies, hot-swappable CPUs, etc.

This approach cannot completely prevent hardware problems from causing failure, but can often keep a machine running uninterrupted for years. It's sufficient for most applications until recently.

As data volumes and applications' computing demands have increased, more applications have begun using larger number of machines, which proportionally increases the rate of hardware faults.

Hence, there is a move toward systems that can tolerate the loss of entire machines, by using software fault-tolerance techniques in preference in addition to hardware redundancy. It also provides operational advantage where one node can be patched at a time, without downtime of the entire system. (AKA rolling upgrade)

Hardware faults are usually random and independent from each other. There may be weak correlations but otherwise it is unlikely that a large number of hardware components will fail at the same time.

### Software Errors
Systematic error within a system. Is harder to anticipate, and because they are correlated across nodes, they tend to cause many more system failures than uncorrelated hardware faults. E.g: the leap second on June 30, 2012, that caused many applications to hang simultaneously due to a bug in the Linux kernel.

The bugs often lie dormant for a long time until they are triggered by an unusual set of circumstances. In those circumstances, the software makes some assumptions about its environment and the assumptions will eventually stop being true for some reason.

There is no quick solution to systematic faults in software. Lots of small things can help: *carefully thinking about assumptions and interactions in the system; thorough testing; process isolation; allowing processes to crash and restart; measuring, monitoring, and analyzing system behavior in production.*

If a system is expected to provide some guarantee, it can constantly check itself while it is running and raise an alert if a discrepancy is found. E.g: *number of incoming messages and outgoing messages in a message queue must be matching.*

### Human Errors
Humans design, build and keep software systems running.

Humans are known to be unreliable. E.g: *one study of large internet services found that configuration errors by operators were the leading cause of outages, whereas hardware faults (servers or network) played a role in only 10–25% of outages*

Approaches to make our systems reliable, in spite of unreliable humans:
* Design systems in a way that minimizes opportunities for error. E.g: *well-designed abstractions, APIs, and admin interfaces*.
* Decouple the places where people make the most mistakes from the places where they can cause failures. E.g: *non-production sandbox using real data, without affecting real users*.
* Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests
* Allow quick and easy recovery from human errors, to minimize the impact in the case of a failure. E.g: *fast roll back, gradual roll out and tools to recompute data*.
* Set up detailed and clear monitoring, such as performance metrics and error rates. AKA telemetry. Monitoring can show us early warning signals and allow checks for any assumptions or constants are being violated.
* Implement good management practices and training. Complex aspect, out of scope of this book.

### How Important is Reliability?
Reliability is not just for nuclear power stations and air traffic control software—**more mundane applications are also expected to work reliably**. Bugs in applications cause lost productivity, legal risks, lost revenue and damage to reputation.

**Even in “noncritical” applications we have a responsibility to our users.**

We should be very conscious of when we are cutting corners, choosing to sacrifice reliability to reduce development/operational cost.

---
## Scalability
> As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

Even if a system is working reliably today, that doesn't mean it will necessarily work reliably in the future. One common reason for degradation is increased load. E.g: *10,000 concurrent users to 100,000 concurrent users, or from 1 million to 10 million.*

Scalability is not a one-dimensional label we can attach to a system.

Instead of saying "X is scalable" or "Y doesn't scale", consider questions like "If the system grows in a particular way, what are our options for coping with the growth?" and "How can we add computing resources to handle the additional load?"

### Describing Load
First, we need to succinctly describe the current load on the system; only then can we discuss growth questions. Load can be described with a few numbers called **load parameters**. The best choice of parameters depends on the architecture of your system. E.g: *requests per second to a web server, ratio of read to writes in a database, number of concurrent users in chat room, hit rate on a cache*. We have to consider if it's the average case or extreme case that is our concerns.

##### Case study - [Twitter timeline](https://www.infoq.com/presentations/Twitter-Timeline-Scalability/)
Two of Twitter's main operations are:
* *Post tweet*: A user can publish a new message to their followers (4.6k requests/sec on average, over 12k requests/sec at peak).
* *Home timeline*: A user can view tweets posted by the people they follow (300k requests/sec).

Twitter’s scaling challenge is not primarily due to tweet volume, but due to fan-out — follower/following graph. There are broadly two ways of implementing these two operations:
1. Posting new tweet into global collection of tweets, look up for accounts followed by the user, look up for tweets from those accounts and merge them in chronological order.
```sql
SELECT tweets.*, users.* FROM tweets  
  JOIN users   ON tweets.sender_id    = users.id  
  JOIN follows ON follows.followee_id = users.id  
  WHERE follows.follower_id = current_user
```
Figure 1-2: Simple relational schema for implementing a Twitter home timeline<br/>
![Figure 1-2](https://github.com/vasetech/designing-data-intensive-applications/blob/master/assets/figure-1-2.png?raw=true "Simple relational schema for implementing a Twitter home timeline") <br/><br/>
2. Maintain a cache for each user's home timeline. When a user posts a tweet, look up that user's followers and insert the new tweet into each follower's timeline cache.<br/><br/>
Figure 1-3: Twitter's data pipeline for delivering tweets to followers, with load parameters
![Figure 1-3](https://github.com/vasetech/designing-data-intensive-applications/blob/master/assets/figure-1-3.png?raw=true "Twitter's data pipeline for delivering tweets to followers, with load parameters")

In the example of Twitter, the distribution of followers per user is a key load parameter for discussing scalability, since it determines the fan-out load.

### Describing Performance
Once you have described the load on your system, you can investigate what happens when the load increases. You can look at it in two ways:
* When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
* When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

Both questions require **performance numbers**. E.g:
* In a batch processing system such as Hadoop, we usually care about **throughput**
* In online systems, what’s usually more important is the service’s **response time**

##### Case study - Response Time Measurement
 In a system handling a variety of requests, the response time can vary a lot. We therefore need to think of response time not as a single number, but as a distribution of values that you can measure.

 Figure 1-4: Illustrating mean and percentiles: response times for a sample of 100 requests to a service

 ![Figure 1-4](https://github.com/vasetech/designing-data-intensive-applications/blob/master/assets/figure-1-4.png?raw=true "Illustrating mean and percentiles: response times for a sample of 100 requests to a service")

The mean is not a good metric if you want to know your "typical" response time. It doesn't tell you how many users actually experienced that delay.

Usually it's better to use percentiles.

This makes the median a good metric if you want to know how long users typically have to wait: half of user requests are served in less than the median response time, and the other half take longer than the median.

High percentiles of response times, also known as tail latencies, are important because they directly affect users’ experience of the service.

Reducing response times at very high percentiles is difficult because they are easily affected by random events outside of your control, and the benefits are diminishing.


### Approaches for Coping with Load
An architecture that is appropriate for one level of load is unlikely to cope with 10 times that load.

People often talk of a dichotomy between scaling up (vertical scaling, moving to a more powerful machine) and scaling out (horizontal scaling, distributing the load across multiple smaller machines). In reality, good architectures usually involve a pragmatic mixture of approaches.

Some systems are elastic, meaning that they can automatically add computing resources when they detect a load increase (useful if load is highly unpredictable), whereas other systems are scaled manually (fewer operational surprises).

> Common wisdom until recently was to keep your database on a single node (scale up) until scaling cost or high-availability requirements forced you to make it distributed.

The architecture of systems that operate at large scale is usually highly specific to the application. E.g: 100,000 request/s & 1kB/request vs 3 request/s & 2GB/request despite same throughput

> An architecture that scales well for a particular application is built around assumptions of which operations will be common and which will be rare—the load parameters.


---
## Maintainability
> Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.

The majority of the cost of software is not in its initial development, but in its ongoing maintenance—fixing bugs, keeping its systems operational, investigating failures, adapting it to new platforms, modifying it for new use cases, repaying technical debt, and adding new features.

To minimize pain during maintenance and thus avoid creating legacy software ourselves, we should pay attention to three design principles for software systems: *Operability, Simplicity, Evolvability*.

### Operability: Making Life Easy for Operations
> Make it easy for operations teams to keep the system running smoothly.

Operations team are vital to keeping a software system running smoothly. Some of their typical responsibility:
* Monitoring the health of the system and quickly restoring service if it goes into a bad state
* Tracking down the cause of problems, such as system failures or degraded performance
* Keeping software and platforms up to date, including security patches, etc.

Good operability means making routine tasks easy, allowing the operations team to focus their efforts on high-value activities. Data systems can do various things to make routine tasks easy. E.g:
* Providing visibility into the runtime behavior and internals of the system, with good monitoring
* Providing good support for automation and integration with standard tools
* Avoiding dependency on individual machines (allowing machines to be taken down for maintenance while the system as a whole continues running uninterrupted)

### Simplicity: Managing Complexity
> Make it easy for new engineers to understand the system, by removing as much complexity as possible from the system.

As projects get larger, they often become very complex and difficult to understand. The complexity slows down everyone working on the system, increase maintenance cost. Various symptoms of complexity: *explosion of state space, tight coupling of modules, tangled dependencies, inconsistent naming, ad-hoc workarounds, etc*

When complexity makes maintenance hard, budget and schedules are often overrun. There is also a greater risk of introducing bugs when making a change (Hard to reason, hidden assumptions, unintended consequences).

Making a system simpler doesn't necessarily mean reducing its functionality; it can also mean removing *accidental complexity*. Complexity is accidental if it is not inherent in the problem to be solved, but arises only from the implementation.

One of best tool to remove accidental complexity is *abstraction*. A good abstraction can hide a great deal of implementation details behind a clean, simple-to-understand facade. It encourages reuse and lead to higher-quality software, as quality improvements in the abstracted component benefit all applications that use it. E.g: *high-level programming languages hide machine code, CPU registers and syscalls. SQL abstracts complex data structures, concurrent requests and inconsistency management*

However, finding good abstractions is very hard. In the field of distributed systems, although there are many good algorithms, it's much less clear how we should be packaging them into abstractions to keep complexity of the system manageable.


### Evolvability: Making Change Easy
> Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change. Also known as extensibility, modifiability, or plasticity.

System's requirements are likely to be in constant flux: *You learn new facts, unanticipated use cases emerge, business pririty changes, new user requests, regulatory change, scaling etc.*

In terms of organizational processes, *Agile* working pattern provides a framework for adapting to change, via test-driven development (TDD) and refactoring.

The ease with which you can modify a data system, adapting it to changing requirements, is closely linked to its simplicity and abstractions.
