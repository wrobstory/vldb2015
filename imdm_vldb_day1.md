### In Memory Database Management Workshop

http://imdm.ws/2015/program/

#### ToDo
- Look at Bitweaving
- Read WideTable paper
- Postgres-R replication
- Get An Analysis of In-Memory Database Durability Techniques presentation
- Read Partitioned Bit-Packed Vectors for In-Memory-Column-Stores

#### Recommended Presentations
  * SLIK: Scalable Low-Latency Indexes for a Key-Value Store, Ankita Kejriwal (Stanford)
  * Near Memory Databases and Other Lessons, Ryan Betts, CTO VoltDB

#### Biggest Takeaways:
  * Oracle and big vendors are pushing db-specific hardware, but its a much bigger hill to climb for academic/oss databases
  * Ankita Kejriwal knows her shit. One of the most impressive Q&As I've seen at a conference.
  * Probably worth taking a serious look at VoltDB as streaming engine. Some huge advantages.
  * There's probably some really interesting stuff that can be built on partitioned bit vectors, with some tweaking for bit-packing.

#### Keynote: Towards Hardware-Software Co-Design for Data Processing: A Plea and a Proposal, Jignesh Patel

* Vicious cycle: leapfrog between hardware and algorithms. Caches, TLB, Main Memory, then algorithms (hash tables in L2, etc) chase the hardware.
* Hash join algo started simple, adapted with architecture
* Now have come full circle: simple hash join very competitive!
* 2008: do we redesign hash joins for SSDs? Not really. Change a few constants. Blocked i/o matters for SSDs. Traditional join algorithms work fine.
* Story so far: Rapid hardware changes, database software plays catch-up
* Being more bandwidth friendly (instead of chasing latency) is the next thing to lean on
* Latency bound apps are doomed. Storage latency not improving fast compared to bandwidth or density.
* Luckily, in many cases we can convert latency-bound applications to throughput applications (e.g. batching transactions)
* We can stream data into the processor WAYYYYY faster than we can process it. 37.8B cycles/s + 68 GB/s = 0.56 bytes/cycle
* *Look at bitweaving*, Sigmod 2013 *WideTable: Extending Bitweaving to Joins* Convert joins to simple scans
* WideTable: Denormalize! data. Columnar Storage, BitWeaving, Dictionary Encoding (Li and Patel, VLDB 2014)
* Aim for bandwidth-friendly methods! Pretty clear that streaming individual tuples is not the future.
* Spread out buffer pool over main memory and SSD
* Databases have mechanisms to shape access patterns, lean on sequential access

#### Cost-based Memory Partitioning and Management in Memcached, Damiano Carra (University of Verona), Pietro Michiardi (Eurecom)

* Memcached: Key-value store, common component of web architectures (caching layer), all data kept in-memory (RAM)
* Memchached divides objects in classes depending on size (small, medium, large)
* Memory divided into blocks called "slabs", default slab is 1 MB. Slab contains variable number of objects.
* Slabs are assigned to classes upon request
* After many requests, available memory (divided into slabs) is full, what happens with new request?
* Memcaches uses LRU eviction *within* the class
* Challenges: how is memory divided among the classes? Static vs. dynamic assignment
* Given a trace and an eviction policty, can compute miss-ratio curves (MRC)
* Goal -> minimize the miss ratio
* Can compute optimal slab assignment based on shape of MRC
* Need new interface: the interface `set(k,v)` implies that all objects have same cost or weight
* But some objects can be difficult to obtain, because they are larger, or it takes more time to retrieve them, etc
* New interface! `set(k,v,c)`, where the application can assign a cost to the object!
* Slab Allocation Schema: Define observation epoch, e.g. when system has experienced M misses
* During each epoch, collect (for each class): number of weighted misses, number of weighted requests
* High number misses -> class is suffering
* Low number misses -> class can lose a slab
* Trace driven experiment with traces collected by major CDN operator
* Interesting finding: cost is *not* correlated with object size

#### Query Optimization Time: The New Bottleneck in Real-time Analytics Rajkumar Sen, Jack Chen, Nika Jimsheleishvilli (MemSQL)

* Real time Data Analytics Today
* Answer analytics queries in "real time", aka within a second or seconds. Data mostly in 100TBs range, queries could be ad-hoc
* Example 1: Financial Service, distributed join order, out join to inner join rewrite
* Ad-hoc queries from dashboards *always* require optimization
* Queries that are *not* ad-hoc may also require optimization, e.g. first invocation of query, if data stats have changed significantly, if query parameters differ
* Why is optimization time important?
  * It cannot afford to be bottleneck in real-time analytics
  * Very small time budget (<100 ms) for query optimization
  * Optimizer should be able to produce efficient execution plans with near-optimimal runtime perf
* MemSQL query optimizer: modular and flexible, built from scratch using C++ lambda functions
* Three components:
  * Rewriter
  * Enumerator
  * Planner
* Rewriter:
  * Applied query rewrites based on heuristics/cost
  * Costs rewrites by calling Enumerator
  * Interleaves mutually beneficial rewrites

#### HyRise-R Acale-out and Hot-Standby through Lazy Master Replication for Enterprise Applications

* Scale up vs Scale out: Hyrise leans on scale-out
* Theoretical replication models: eager vs. lazy, group vs. master
* Implementations:
  * Postgres-R - Eager group replication based on shadow copies
  * ScyPer - Lazy master replication with row layout for master node
* Hyrise
  * Storage engine focused on main-memory processing, hybrid storage layout
  * Dictionary and bit-vector compression
  * Main/delta architecture with merge process
  * Hybrid row and column layouts
  * Supports vertical and horizontal partitioning
* Transactional workload -> Master node
* Reads -> Cluster nodes
* Logs written to file system and sent to cluster interface, then interface sends log info to replicas
* Frequency configurable, based on number of calls, buffer size, time since last transmission
* Heartbeat protocol for failover?

#### Write Amplification: An Analysis of In-Memory Database Durability Techniques, Jaemyung Kim, Kenneth Salem, Khuzaima Daudjee (University of Waterloo)

* Durability matters
* Even for IMDB, i/o is unavoidable (ACID, Durability)
* Goal of Write Amplification Model
  * Quantify/compare i/o efficient of persistent storage management
  * Provide insight into nature of update-in-place vs copy-on-write
* Architectural Diversity in IMDB SM. Two classes:
  * Update-In-Place
  * Copy-On-Write
* Update-In-Place (UIP)
   * conventional page based
   * random writes for checkpointing
   * device sensitive (HDD vs SSD)
* Copy-On-Write (COW)
  * logging only (log-structured) db
  * log-structured memory *and* disk (RAMCloud)

#### SLIK: Scalable Low-Latency Indexes for a Key-Value Store, Ankita Kejriwal (Stanford)

* Hypothesis: K/V store can support highly consistent secondary indexes
* SLIK: Scalable Low-latency Indexes for Key-value Store
* Enables multiple seconary keys for each object
* Allows lookups and range queries on these keys

#### Near Memory Databases and Other Lessons, Ryan Betts, CTO VoltDB

* What happens when you try to productize some papers?
* Lessons and observations learned trying to build a Stonebraker database
* Research -> Code, Code -> Product
* H-Store: High Performance, Distributed Transaction Processing System
* Think of in-memory DB as "near-memory" DB. Gotta be durable.
* 3 Pillars of H-Store
  * Partition Data
  * Execute serially at a partition
  * Don't *ever* block transactions
* Several assumptions in the research
  * OLTP systems fit in main memory
    * Customers willing to separate OLTP workloads from OLAP
    * Only true at scale- most people prefer simple consolidated system from multiple systems
    * True...mostly.
    * Scale on a few axes- devices, users
    * Number of use-cases where data access skews hot/cold
      * IMDB vs. Cache + K/V
      * Customers prefer tiered architecture over large DRAM cluster
  * Recovery can be accomplished by copying missing state from other database replicas.
    * Data absolutely has to be made durable *somewhere*
    * VoltDB uses "snapshots" and "command logging"
    * Snapshots are checkpoints, full copy via copy-on-write
    * Users can initiate snapshot on demand, so some users are aggregating logs, then snapshotting to CSV for downstream
    * Command logging is essentially write-ahead log. But *logical log*, not log of tuples. Commands.
    * Command logging is a slightly more predictible size to write to disk than a tuple
    * Unclear if this will survive, or more traditional logging will win in the end with VoltDB
  * An alternate active-active architecture appears to make sense. In this case, all replicas are "active" and the transaction is performed syncronously on all replicas.
    * Interesting choice, core aspect of system
    * Parallel execution, syncronous replication
    * Syncronous replication huge win. This allows VoltDB to parallelize read workloads.
    * This havs an important tradeoff: all commands have to be deterministic
      * VoltDB planner flags non-deterministic queries
    * Clocks are hard. Timeline coordination/sync was too painful.
    * Had to rewrite txn syncing: Partition sequencers
    * They wrote leader election/txn ordering before Raft, but very similar
    * They lean on many pieces of Zookeeper
  * Single threaded execution: long running commands do not exist in OLTP. This is controversial.
    * Some things do introduce long running txns.
    * Single biggest source of long-running commands are operator errors. Fat fingered queries.
    * Can cancel reads, set per-read timeout.
  * Any future database system must be designed from the ground-up to run on clusters
    * Customers can be stupid with clusters.
    * On more than one occasion, failed node results in customer deleting the entire cluster
    * Huge lack of monitoring. Debugging is hard.
    * Partitioning is a complex design decision.
    * Explaining a dynamically partitioning cluster is too hard a sell for customers.
    * Not convinced the future is all clusters, all the time
    * The future might be shared-memory across many CPUs
* Commercial lessons
  * New DBs are largely for new problems
  * In memory DBs differentiate by performance
  * Many high-throughput applications are not request/response.
  * VoltDB has introduced set of importers to take that from the customer. Subscribe to queue.
  * One size does not fit all. Play nice with others. Think about integration with other systems.
    * Many customers have multiple layers of storage and engines
    * VoltDB has built exporters to send messages downstream. Exporters send to new queues downstream after tnx commits.
* Looking Ahead
  * Internal PaaS effect
    * Googles and Facebooks can try new stuff internally *very* quickly
    * Companies like VoltDB have much more work to do packaging, working with SaaS, etc
    * See enterprise trying to replicate architectures built by Googles, Facebooks, etc, but doing so poorly
    * A lot of Hadoop use-cases require analysts to code. That's why SQL on Hadoop has been so important.
    * Pace of innovation vastly outpaces pace of education of vendors and customers
    * A lot of people try to solve problems by going to the Apache menu

#### Selection on Modern CPUs, Steffen Zeuch, Johann-Christoph Freytag (Humboldt Universität zu Berlin)

* How to improve branch prediction?
* Branch history buffer saves last outcomes for each branch to improve prediction
* Ivy, Sandy, Haswell processors recognize up to 72 different outcomes

#### NVC-Hashmap: A Persistent and Concurrent Hashmap For Non-Volatile Memories, David Schwalb, Markus Dreseler, Matthias Uflacker, Hasso Plattner (Hasso Plattner Institute)

* NVRAM allows for easy persistency and (almost) instant recovery of In-Memory Database
* Data structures need to be adapted for NVRAM to fulfill ACID criteria
* Lock-free data structures can be easily adapted
* What is NVRAM? Non-Volatile RAM, does not lose contents on power loss, battery backed or new microchip tech
  * Intel announced 3D XPoint
* NVRAM directly attached to CPU memory controller, just like DRAM. Both can run side-by-side.
* Still in dev. Can't buy it yet. Slower than  DRAM, way faster than Flash and especially HDD.
* Moving Hyrise to NVRAM - what needs to be persisted?
* Data structure challenges
  * Volatile CPU cache hold data not on NVRAM
  * Incomplete updates need to be rolled back
  * NVRAM memory dynamically allocated
  * Pointers to NVRAM objects need to be valid after a restart
* Challenge: Persisting Changes
  * Volatile CPU caches hold data not on NVRAM
  * If system crashed between Cache and NVRAM, we lose part of committed transaction
* Non-Volatile Hashmaps
  * Based on split-ordered lists. Instead of hash buckets, you have pointers to one big linked list

#### Partitioned Bit-Packed Vectors for In-Memory-Column-Stores, Martin Faust, Pedro Flemming, David Schwalb, Hasso Plattner (Hasso Plattner Institute)

* Compressed append-only structure for the attribute vector
* Ad-hoc analytics: fast col scans, high compression, bandwidth
* Transaction processing: tuple reconstruction, insert/update perf, Latency
* How do we handle both workloads without compromise or replication?
* All sorts of mixed-model DBs, persisting data in both row and column formats for different workloads
* The compromise in the merge process, in which we have a main and delta partition, and async merge them.
* This is the "tuple mover" concept from the C-Store paper
  * Majority of data in heavily compressed main partition
  * Main partition still array-addressable for tuple reconstruction
  * Delta optimizes for inserts
  * Updates/deletes via invalidation of old tuples
* So: how would we build a system without a delta merge?
* Partitioned bit-packed vector
  * Generalize "main delta partition per table" concept to "x-partitions per column"
* Compression benefit: Gaussian dist of distinct values

