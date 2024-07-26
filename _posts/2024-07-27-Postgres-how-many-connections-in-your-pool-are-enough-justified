# [Postgres] How many connections in your pool are enough/justfied?

- We will explore:
    - What's the cost of a connection?
        - A OS native process is an expensive construct. It requires execution of significant logistical instructions and data structures while creating as well as well scheduling for execution.
    - What vectors influences the decission of figuring out how many connections(bellpark) would be optimal?
    - How to come up with the right number of connections?
        - The connections are mostly hard conigured via env variable or some other configuration injection mechanism during the startup of the application/service, meaning  we are supposed to figure out the adequate number of connections before deployment.

- The vector which principally influences the number of connections decission are(in declining order):
    - Available CPU cores
    - Available RAM
    - Your DISK I/O Speed
    - Network Bandwidth

These vectors influences the decission in terms of bottlenecks they intorduces.

- a cpu core can execute only one instruction per execution cycle, hence one process at a time
    - you might wonder, we should have connections as a multiple of number of cores available, good but wait.
- cpu reads information from cache-lines(L1 - fast but expensive: both cost and required real estate) present on its own chip die, but the capacity is so low that it can hold data barely enough for instructions queued for execution
    - it does so because
        1. speed at which electrons(read electricity) can travel in circuitry is really a bottleneck especially when you make them do back and forth for billions of times per second
        2. the RAM uses different(slow but cheap and higher capacity) technique
- for well distributed(against data) queries needing data beyond what's already there in in-memory cache-buffer, disk is a major bottleneck
    - but hey disk is cheap and answers where would data go if someone develops OCD with state of power plug?

> Fun fact:
Practical read speed of Great Enterprise grade SSDs is below 10GB per second that is 10 bytes per nano second.

Databases are often queried randomly and hence disk reads are random.

Practical random read speed, for a h**igh-performance PCIe 5.0 SSD like  PM1743,** hovers around 2.5 million I/O per second

Postgres writes rows in pages of size 8kb each

To. read a row, postgres needs to access a full page even if it’s 1/8 of the page

If a row has like ~1kb long content(1 8-byte long timestamp, 10 32bit integers for various foreign keys i.e. 40 bytes, one 100 words article i.e. 500 16bit unicode chars i.e. 1000 bytes )

It would require 1 I/O operation to access the page containing the row. Targetted access is only possible if we have our query criteria indexed before hand
> 

If you query requires loading one row with a text field with 100 ascii characters and 5 32-bit integers, we are talking about 100bytes + 160bytes plus some metadata about who is who and where, roughly translating into let's say half KB or  ~512 bytes

Factoring the electrons traveling from your CPU to disk controller via central bus, and then to disk, plus getting parked in queue in case someone else is having little work-place romance with the disk, we are talking about micro seconds of latency(1000 nano second is 1 micro second, don't confuse with milli second). 1 Ghz processor core can theoretically run 1 billion instructions per second.

So, what to do with idle sitting CPU core when rest of the system is flirting with disk?

Operating System vendors say put the process who triggered disk IO into sleep mode till we get data and disk controller raises an intrupt(yeah just like that cool person in your morning standup who has to say something when you are in middle of delivering the punchline you reserved for task completion)

So what to do when the process is snorring? Obviously let's ask the CPU core to run instruction of some other process.(All the logistics related to coming up with this decission and executing it gracefully is called task scheduling in OS lingo)

**Assuming you are with me and we can safely declare that okay, we can do more, we can plan to cut the queue and get compute part of our next query sorted as CPU is just idle waiting for disk to return requested data page(s).**

**What it takes to cut the queue?**

Establish a new connection with Postgres and submit the next query using this connection.

So, for each query we intend to cut the queue we would need a database connection.

**But for how many queries we can cut the queue? Or simply how many connections we can have?**

The answer lies in the context. What kind of queries you are running?

Are you trying to retrieve the master informaton which is limited by quantity and rarely changes? Given Postgres smartly caches repeatedly access data in RAM, RAM is your bottleneck.

Are you trying to update the tables? Disk is your bottleneck

Are you trying to insert data? Again Disk is your bottleneck, though to cut the overhead, postgres doesn't insert the data in tables directly, and instead parks the raw-data in a fast to persist log file and parks the strutured raw data in pages which it writes in one go after certain threshold is met.

Are you running DDL queries frequently? Might require lock on full table and a lot of IO

Does your queries translate into lock contentions? CPU is your bottleneck, semaphoric checks are done till the lock is available on the data covered by your query.

**This information is great but I am still not sure how to come up with right number.**

Okay, so here’re few approaches:

1. Binary Discovery approach
    1. You model your production query execution behaviour
    2. you take a wide range of connection count(2*number of database server cpu cores - 100*number of cpu cores)
    3.  you stress test the system with connection pool count defined by repetead division of connection range in half till we end up with the maximum throughput. Start with let's say 4-400: then explore 202, 101, 52, 26, 39…..
2. gentleman approach would be:
    1. identify applicable bottlenecks i.e. network, disk, cache, memory buffer size etc
    2. Identifying the 20% queries which run 80% of the time.
    3. Let's say you are a News website with 80% people not moving beyond the first news on your landing page.
    4. For sake of example, we aren’t using application or protocol level caching.
    5. Let's say your landing page requires quering home_page_select_news_personalised_by_demography table. Retrieves roughly 40 rows each containing 6 columns of size 2 KB
    6. the news article table has each row of roughly 16-KB(each article is roughly 1500 words long and is personalised for each user i.e. 7.5k 16-bit unicode chars → 15 kbytes)
    7. You have 10000 visitors max per second at any day.
    8. So, we are talking about 10000 * *40 * 2* = 800 Mega bytes of data movement per second, and 80% * *10000 ** 16Kb = 120.8 Megabytes
    9. You are having a state of the art SSD with random read throughput of 2000 KIOPS(Kilo Input/Output Operations) and 10 GBPS sequential read capacity
        1. A home page news bulletins read query would require atleast 40 IOPS and 40*8kb of sequential reads
        2. SSD is access over SATA Bus or PCI lane, and there’s a ~20 micro-second latency between CPU’s instruction to read data from disk and the instruction reaches to disk controller.
        3. 40 IOPS would require 20 micro seconds (2 million IOPS per second is 2 IOPS per micro second)
        4. 320KB sequential reads(remember we remarked a row is stored in a page and a page is 8kb long) would need 32 micro seconds
    10. We shouldn’t forget the network part
        1. Let’s say we have a 10 Gigabit connection with 0.1ms ping latency
        2. Let’s say our sql query and parameters collectively are of 1 KB or ~8 kilo bits
        3. 10 Gbps is 10 kilo bits per micro-second, factoring in ping latency, protocol overhead, it would still take ~100-200 micro-seconds
        4. Similarly for collecting the data we have bandwidth latency of 32 micro-seconds, total latency would be 150-250 micro-seconds
    11. This means that your **CPU has nothing to do for 400+72 = 472 micro-seconds** every time you run an sql query
        
        We have total latency of: ~2.5ms
        
    12. Let’s guesstimate how much time(in micro seconds) it takes to run an sql query end to end:
        1. Send Query Over Network: Network | ~200
        2. Query parsing and syntax checking: CPU+RAM| Use from cache in case of prepared statements | ~1
        3. Admin work(Authentication/authorization): CPU+RAM | ~50
        4. Query analysis and rewriting: CPU+RAM | Use from cache in case of prepared statements | ~1
        5. Query planning and optimization: CPU+RAM |  Depends on `plan_cache_mode` if set force_generic_plan ~10 else if auto ~50 else if force_custom_plan ~100
        6. Executor initialization: CPU+RAM | ~50
        7. Index selection and access: CPU+RAM | Assuming 20% of the indexes needs to be loaded from disk | ~100
        8. Data retrieval:
            1. Buffer cache check: CPU+RAM | Let’s say available 10% of the time | ~10
            2. Disk I/O (if data not in cache): DISK | 90% of time |
                1. Disk ~72*0.9  → ~ 63 when disk is idle
                2. If connection processes are competing for disk access, the latency would atleast quadruple
                3. so it would be ~250 micro-seconds
                4. Kernel↔User space interaction—sys call etc, buffer sharing —> ~20
        9. Cast/Filter/Union data as per query: CPU+RAM | ~50
        10. Collect result over network: NETWORK | ~200
        11. Application not releasing the connection during intermediate steps b/w sql queries| IDLE | ~5-50
            1. application should better release the connection if it’s doing some rpc or network call like http etc
    13. Guestimate totals to 1000 micro-seconds, round off to 1.5ms(assuming I missed something)
    14. While executing a query, CPU is idle for >750micro-seconds or >50% of the time(just waiting for the IO to happen)
    15. Now we know how much time it takes to execute an query on an average, 1ms
    16. Theoretically,  ~650 queries can be processed per second per connection
    17. A quad core CPU, can have 4 connection procceses parallely running, so net throughput is ~2400 queries per second
    
    > Did you know? When a process is waiting for IO, the OS will deschedule it and will schedule the next process in the queue.
    > 
    
    Did you notice, the disk is idle while CPU is busy doing logic work? Almost 80% of the time.
    
    We can be more productive by treating the machine as a machine.
    
    We can add atleast one more connection for each core, resulting in >85% increase in CPU and Disk productivity.
    
    Our throughput should also double.
    
    To come up with ideal number of connection count, I have created this simple equation.
    
    `(x+1)(a) + b*(1.02^x) = 100`
    
    `a = max(p,d)`
    
    `p` is the average cpu utilisation during your stress test with `k` connections where `k` is count of physical cores of cpu(0-100 scale)
    
    `d` is the average % utilisation of disk KIOPS(Kilo Input Output Operations) throughput
    
    `b` is Operating System overhead of managing an active process(Use a number between 4-8)
    
    `1.02` is assumption that overhead compounds by 1% with every new process scheduled
    
    In our case `a` would be 50(just put the number without the %)
    
    `(x+1)*50 + 5*(1.02^x) = 100`
    
    > Put it in google search
    > 
    
    `log(5) + xlog(1.02) = log(100 - (x+1)*50)`
    
    > or solve this :p
    > 
    
    You would get x = 0.898
    
    Total connections apt for you is:
    
    `floor(kx + k`)
    
    For k = 4:
    
    `floor(~3.6 + 4)` is 7
    
    However, 7 connections would be able to execute ~4500 queries only.
    
    > But that doesn’t help in above example, right? We need ~20k queries per second.
    > 
    
    I get it but increasing the number of connections is not the answer. We are better off increasing the cache buffer size(remember the vacant RAM is utilised by OS to cache disk pages so don’t try to occupy it all).
    
    This should reduce the expensive Kernel↔User space interaction, decrease disk IOPS utilisation. Should reduce CPU utilization as well. How much? Depends on available ram.
    
    In descrete language, the gain is directly proportional to the percentage of actively used data that can be kept in cache buffer in RAM while not impacting the OS optimisations/cache using free RAM.
    
    If 90% data is loaded on cache buffer, for 90% of the queries we would save atleast 20% of query execution time and 100+ micro-seconds in CPU time per query (saved from kernel↔user space interaction, os scheduling/descheduling etc)
    
    Overall, our average query latency would go down by 0.9*350(disk latency+os overhead) = 300+ micro-seconds
    
    This should translate into 30%+ improvement in throughput per connection.
    
    Overall you would get 5800+ queries per second
    
    Your disk is idle now, but CPU is fully occupied in mostly productive work.
    
    You could utilise the fancy disk better by scaling your CPU.
    
    Every additional core would bring nearly 30% improvement in throughput.
    
    An octa core CPU would get you 10K+ queries per second.
    
    For further scale, it would be recommended to use partioning/sharding in an horizontally scaled apparatus.
    
    > Do you know? Smaller number of active processes results in efficient and productive utilization of CPU L1, L2, L3 cache.
    > 
    
    > What if we had a slower disk with just 1000 KIOPS and 2 GBPS  sequential read speed?
    > 
    
    Disk latency would be: 40+160 = 200*4 = 800 micro seconds when 4 cpus are competing for disk 
    
    If you can’t have more RAM, it would be justfied to have more connection to get non linear increment in throughput, but don’t forget that disk is already a bottleneck. Queing more I/O activity won’t help a lot.
    
    Only value we can have is warmed up queries ready at pre disk fetch stage.
    
    We can modify the equation and just put in the cpu usage with 1 connection per core. With increased latency, cpu usage is bound to be less than 700/2100 ⇒ 30%
    
    For `(x+1)*30+5*(1.02^x) = 100`, x is 2.16
    
    Total connections be 12 on a quad core CPU.
    
    If network is a bottleneck(maybe because database and application are in two separate regions), it might make sense to identify network latency. Monitor cpu usage, and increase connection count accordingly.
    
    If network latency more than doubles, as in becomes 1100 micro-seconds, the 700/2800 would give us ~25% as cpu usage
    
    For `(x+1)*25+5*(1.02^x) = 100`, x is 2.79
    
    Total connections be `floor(4*2.79+4)`  = 15 on a quad core CPU.
    


> Let's connect over linkedin if you got feedback or you want to brainstorm further on this topic
> https://www.linkedin.com/in/ravinderpayal/
