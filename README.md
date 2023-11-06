# systemdesign_alex
This is learning path for general system design from alex xu

<br><br><br>

# Chapter 1: Scale from zero to millions of users


<br><br><br>

## which db to use?
1. relational db (RDBMS)
    - mysql
    - postgresql
    - oracle
    - sql server
    - db2
    - sqlite
    - ...

2. non-relational db:
    - your application requires super-low latency
    - your data are unstructured, or you do not have any relational data
    - you only need to serialize and deserialize data(JSON, XML, YAML, etc)
    - you need to store a massive amount of data

<br><br><br>

## Vetical scaling vs Horizontal scaling

1. Vetical scaling
    - scale up, add more resources(CPU, RAM) to a single server
    - when traffic is low is fine
    - hard limit, impossible to add unlimited CPU and RAM
    - doesn't have failover and redundancy, one server down, the whole system down

2. Horizontal scaling
    - scale out, add more servers to the system
    - more desirable for large scale applications

<br><br><br>

## Load Balancer `this is for the  web tier`
- users connect to the public IP of the load balancer directly
- when a large amount of traffic comes in to the system, the load balancer will distribute the traffic to different servers
- For better security, private IPs are used for communication between servers.
    - A private IP is an IP address reachable only between servers in the same network; however, it is unreachable over the internet.
- after a load balancer and a second web server are added, we successfully solved no failover issue and improved the availability of the web tier.

<br><br><br>

## Database replication `data tier`

1. master slave replication
    - master for `insert`, `update`, `delete`
    - slave for `read`
    - Most applications require a much higher ratio of reads to writes; thus, the number of slave databases in a system is usually larger than the number of master databases.

2. advantages
    - better performance (read scalability)
    - reliability(disaster recovery)
    - high availability

3. what if one of the databases goes offline
    - only one slave goes down, read will direct to master temporarily, and start a new slave db for replacement
    - if master goes down, promote a slave to master, new slave create to replace old one.
        - new master might not update to date
            - run a script to update
            - multi-master
            - curcular replication

<br><br><br>

## Cache
1. temporary storage area in memory, so that subsequent requests are served more quickly.
    - result of expensive responses
    - frequently accessed data

2. Every time a new web page loads, one or more database calls are executed to fetch data. The application performance is greatly affected by calling the database repeatedly. The cache can mitigate this problem.


## Cache Tiers
1. The cache tier is a temporary data store layer, much faster than the database. 
    - The benefits of having a separate cache tier include better system performance, 
    - ability to reduce database workloads, 
    - and the ability to scale the cache tier independently.

2. read-through cache.
    - After receiving a request, a web server first checks if the cache has the available response. 
        - If it has, it sends data back to the client. 
        - If not, it queries the database, stores the response in cache, and sends it back to the client.

## Considerations for using cache
1. Decide when to use cache.
    - Consider using cache when data is read frequently but modified infrequently.

2. Expiration policy.
    - [ref](https://dev.acquia.com/blog/how-choose-right-cache-expiry-lifetime)
    - It is advisable not to make the expiration date too short as this will cause the system to reload data from the database too frequently. 
    - Meanwhile, it is advisable not to make the expiration date too long as the data can become stale.

3. Consistency
    - Inconsistency can happen because data-modifying operations on the data store and cache are not in a single transaction. When scaling across multiple regions, maintaining consistency between the data store and cache is challenging. 

4. mitigating failures:
    - multiple cache servers across different data centers are recommended to avoid SPOF

5. Eviction Policy
    - Once the cache is full, any requests to add items to the cache might cause existing items to be removed. 
        - LRU
        - LFU
        - FIFO

<br><br><br>

## Content delivery network (CDN)
1. A CDN is a network of geographically dispersed servers used to deliver static content. 
    - CDN servers cache static content like 
        - images, 
        - videos, 
        - CSS, 
        - JavaScript files, etc.

2. CDN in high-level:
    - when a user visits a website, a CDN server closest to the user will deliver static content. 
    - Intuitively, the further users are from CDN servers, the slower the website loads.

3. CDN workflow:
    1. user-A try to get image.png
    2. CDN has not, request from original server
    3. origin return imgage.png with HTTP header TTL describes how long the image is cached
    4. return to user-A
    5. user-B try to get image.png
    6. within TTL, returned from CDN directly

## Considerations of using a CDN
1. Cost
2. Setting an appropriate cache expiry
3. CDN fallback
4. Invalidating files

1. Static assets (JS, CSS, images, etc.,) are no longer served by web servers. They are fetched from the CDN for better performance.
2. The database load is lightened by caching data.

<br><br><br>

## Stateless web tier
1. Now it is time to consider scaling the web tier horizontally. 
    - For this, we need to move state (for instance user session data) out of the web tier.
    - A good practice is to store session data in the persistent storage such as relational database or NoSQL
    - Each web server in the cluster can access state data from databases. This is called `stateless web tier`

## Stateful architecture
1. each data stored at specific server, the user must routing to the same.
    - Adding or removing servers is much more difficult with this approach. It is also challenging to handle server failures.

## Stateless architecture
1. In this stateless architecture, HTTP requests from users can be sent to any web servers, which fetch state data from a shared data store.
2. State data is stored in a shared data store and kept out of web servers.
3. A stateless system is simpler, more robust, and scalable.

<br><br><br>

## Data centers

1.  technical challenges must be resolved to achieve multi-data center setup:
    - Traffic redirection
    - Data synchronization
        - [Netflix](https://netflixtechblog.com/active-active-for-multi-regional-resiliency-c47719f6685b)  
    - Test and deployment
    

<br><br><br>

## Message queue
1. A message queue is a durable component, stored in memory, that supports asynchronous communication.
    - It serves as a buffer and distributes asynchronous requests
    - Pub/Sub
2. Decoupling makes the message queue a preferred architecture for building a scalable and reliable application. 


## Logging, metrics, automation
1. Logging
    - Monitoring error logs is important because it helps to identify errors and problems in the system.

2. Metrics
    - Collecting different types of metrics help us to gain business insights and understand the health status of the system.
        - Host level metrics
        - Aggregated level metrics
        - Key business metrics

3. automation
    - CI/CD
    - automating your build, test, deploy process, etc. could improve developer productivity significantly.

## Adding message queues and different tools



<br><br><br>


## Database scaling

### Vertical scaling
1. drawbacks:
    - You can add more CPU, RAM, etc. to your database server, but there are hardware limits. If you have a large user base, a single server is not enough.
    - Greater risk of single point of failures.
    - The overall cost of vertical scaling is high. Powerful servers are much more expensive.

### Horizontal scaling
1. AKA, sharding, is the practice of adding more servers.
2. Sharding separates large databases into smaller, more easily managed parts called shards. `Each shard shares the same schema, though the actual data on each shard is unique to the shard.`
3.  User data is allocated to a database server based on user IDs.

4. most important factor is the choice of the `sharding key`
    - to choose a key that can evenly distributed data.

5. sharding introduce new challenges to the system:
    - Resharding data
    - Celebrity problem
        - Imagine data for Katy Perry, Justin Bieber, and Lady Gaga all end up on the same shard. 
        - For social applications, that shard will be overwhelmed with read operations. 
        - To solve this problem, we may need to allocate a shard for each celebrity. Each shard might even require further partition.
    - Join and de-normalization
        - Once a database has been sharded across multiple servers, it is hard to perform join operations across database shards 

<br><br><br>

## Millions of users and beyond
1. More fine-tuning and new strategies are needed to scale beyond millions of users.
    -  you might need to optimize your system and decouple the system to even smaller services.

2. so far we learned:
    - Keep web tier stateless
    - Build redundancy at every tier
    - Cache data as much as you can
    - Support multiple data centers
    - Host static assets in CDN
    - Scale your data tier by sharding
    - Split tiers into individual services
    - Monitor your system and use automation tools




<br><br><br><br><br><br>


# Chapter 2: Back-of-envelope estimation

<br><br><br>

## Power of two
- 1 byte = 8bits
- 2^10 = 1 Thousand     = 1 Kilobyte = 1 KB
- 2^20 = 1 Million      = 1 Megabyte = 1 MB
- 2^30 = 1 Billion      = 1 Gigabyte = 1 GB
- 2^40 = 1 Trillion     = 1 Terabyte = 1 TB
- 2^50 = 1 Quadrillion  = 1 Petabyte = 1 PB

<br><br><br>

## Latency numbers every programmer should know

- By analyzing the numbers in below figure
    1. memory is fast but the disk is slow
    2. avoid disk seeks if possible
    3. simple compression algorithms are fast
    4. compress data before sending it over the internet if possible
    5. data centers are usually in different regions, and it takes time to send data between them
    - ![imgs](./imgs/Xnip2023-11-06_11-34-15.jpg)


<br><br><br>

## Availability numbers
- High availability is the ability of a system to be continuously operational for a desirably long period of time.

- `SLA`(service level agreement). This is an agreement between you (the service provider) and your customer, and this agreement formally defines the level of uptime your service will deliver.

| Availability % | Downtime per day | Downtime per year |
| -------------- | ---------------- | ----------------- |
| 99%            | 14.40 mins       | 3.65 days         |
| 99.9%          | 1.44 mins        | 8.77 hours        |
| 99.99%         | 8.64 s           | 52.60 mins        |
| 99.999%        | 864.00 ms        | 5.26 mins         |
| 99.9999%       | 86.40 ms         | 31.65 s           |


<br><br><br>

## Example: Estimate Twitter QPS and storage requirements

1. Assumptions
    - 300m monthly active users
    - 50% of users use Twitter daily
    - Users post 2 tweets per day on average
    - 10% of tweets contain media
    - data is stored for 5 years

2. Estimations
    - Query per second(QPS) estimate
        - Daily active users(DAU) = 300m * 50% = 150m
        - Tweets QPS = 150m * 2 tweets / 24 hour / 3600 seconds = ∼3500
        - Peek QPS = 2 * QPS = ∼7000
    
    - media storage estimate:
        - Average tweet size:
            - tweet_id  64 bytes
            - text      140 bytes
            - media     1 MB
        - Media storage: 150m * 2 tweet * 10% * 1 MB = ∼30 TB per day
        - 5-year media storage: 30 TB * 365 * 5 year = ∼55 PB

    - text storage estimate:
        - text storage: 150m * 2 tweet * 90% * 140 bytes = 37800,000,000 bytes = ∼35 GB per day
        - 5-year media storage: 35 GB * 365 * 5 year = ∼ 62 TB

<br><br><br>

## Tips

1. Back-of-the-envelope estimation is all about the process
    - Rouding and Approximation. Precision is not expected.  “99987 / 9.1” >  “100,000 / 10”
    - Write down your assumptions. for your reference later
    - Label your units. 5MB, 5KB instead of 5
    - Commonly asked back-of-envelope estimations. QPS, peak QPS, storage, cache, # of servers, etc.

<br><br><br><br><br><br>

# Chapter 3: A framework for system design interviews

## A 4-step process for effective system design interview

<br><br><br>

### Step 1 - understand the problem and establish design scope
- What specific features are we going to build? 
- How many users does the product have?
- How fast does the company anticipate to scale up? What are the anticipated scales in 3 months, 6 months, and a year?
- What is the company’s technology stack? What existing services you might leverage to simplify the design?
- what is traffic volume


<br><br><br>

### Step 2 -  Propose high-level design and get buy-in

- come up with an initial blueprint for the design.
- draw box diagrams with key components on the whiteboard or paper
- do back-of-the-envelope calcuolations to evaluate if your blurprint fits the scale constraints

<br><br><br>

### Step 3 - Design deep dive
- Agreed on the overall goals and feature scope
- Sketched out a high-level blurprint for the overall design
- Obtained feedback from your interviewer on the high-level design
- Had some initial ideas about areas to focus on in deep dive based on her feedback

<br><br><br>

### Step 4 - Wrap up
- the interviewer might want you to identify the system bottlenecks and discuss potential improvements
- it could be useful to give the interviewer a recap of your design.
- Error cases (server failure, network loss, etc)
- Opeartion issues are worth mentioning
- How to handle the next scale curve is also an interesting topic
- Propose other refinements you need if you have more time


<br><br><br>

## Summarizations

<br><br><br>

### Dos
- always ask for clarification
- understand the requirements of the problem
- there is neither the right answer nor the best answer
- let the interviewer know what your are thinking
- suggest multiple approaches if possible
- once you agree with your interviewer on the blueprint, go into details on each component
- Bounce ideas off the interviewer
- Never give up

<br><br><br>

### Don'ts
- dont be unprepared for typical interview  questions
- dont jump into a solution without clarifying the requirements and assumptions
- dont go into too much details on a single component in the beginning
- if you get stuck, dont hesistate to ask for hints
- again, communicate
- dont think your interview is done once you give the design.

<br><br><br>

## Time allocation on each step
- Step 1 Understand the problem and establish design once : 3 - 10 mins
- Step 2 Propose high-level design and get buy-in : 10 - 15 mins
- Step 3 Design deep dive : 10 - 25 mins
- Step 4 Wrap : 3 - 5 mins





