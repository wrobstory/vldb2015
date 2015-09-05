### VLDB Day 4

* ToDo
  * Read "Building a Replicated Logging System with Apache Kafka" paper
  * Share "Building a Replicated Logging System with Apache Kafka" preso
  * I <3 Logs book by Jay Kreps
  * Look at Samza paper from CIDR 2015
  * Maybe ref CTE paper w/ respect to Redshift perf on CTEs
  * Look at DBx1000, github.com/yxymit/DBx1000
  * Read HStore papers
  * Read "Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores"
  * Must Read: "E-Store: Fine-Grained Elastic Partitioning for Distributed Transaction Processing Systems"
  * Sigmod 2015 "Squall: Fine-Grained Live Reconfiguration for Paritioned Main Memory Databases"
  * Read "Rethinking serializable multiversion concurrency control"


* Biggest Takeaways:
  * LinkedIn using Kafka as replication layer under Espresso DB!
  * Graph partitioning is used in Facebook is used for Data Partitioning
  * FB has 300PB of Hive data
  * Databases allocate timestamps for Timestamp Ordering with the `atomic add` instruction for speed.
  * Hardware Counter can get 1B ts/s
  * E-Store is going to be the Next Big DB Thing. I bet Stonebraker's next company is this auto-partitioning.
  * ADDICT: Advanced Instruction Chasing for Transactions is actually optimizing txn instructions across cores to take advantage of L1/2/3

#### Building a Replicated Logging System with Apache Kafka, Guozhang Wang (LinkedIn)

* Pre-Kafka: LinkedIn was all point-to-point pipelines, with complex producer/consumer graph
* All of LinkedIn messaging infra is managed by 5 data engineers and 2 ops engineers
* Tricky part: achieving consensus for Log Replication between brokers
* Primary-backup Replication, Conventional Quorum Commits
* 2F + 1 Replicas is a LOT in production
* Soln: COnfigurable In-Sync-Replica Commits
* Membership Mgmt:
  * Detect broker fialure via ZK
  * Leader failure => elect new leader from ISR
  * Leader and ISR persisted in ZK
* LinkedIn using Kafka as replication layer under Espresso DB

#### Optimization of Common Table Expressions in MPP Database Systems, Amr El-Helw (Pivotal Inc.)

* WITH clause, temp tables for one query, widely used
* Two reasons: query readability, performance gain??
* Challenge 1: Deadlock hazard
* Challenge 2: To inline or not to inline?
* Challenge 3: Contexualized optimization

#### One Trillion Edges: Graph Processing at Facebook-Scale, Avery Ching (Facebook)

* Three major use cases for graph processing in FB:
  * Ranking Features
  * Recommendations
  * Data Partitioning
* Graph partitioning is used in Facebook is used for Data Partitioning
* Industry Graphs are 70x larger than benchmarks
* Requirements
  * Efficient iterative computing model
  * Easy to program and debug graph-based API
  * Scale to 1B+ nodes and hundreds of billions of edges
  * Easy interop with Hive
  * Multi-tenant
* FB has 300PB of Hive data
* Using Apache Giraph
* Being very careful about Java objects, falling back to primitive arrays and bytearrays instead of boxed reps

#### Staring into the Abyss: An Evaluation of Concurrency Control with One Thousand Cores, Xiangyao Yu (MIT)

* Era of single-core CPU speedup is over, cores going up, DBMSs not ready
* Will DBMS on future architectures (1000+ cores) scale to this level of parallelism?
* "All classic concurrency control algorithms fail to scale to 1000 cores"
* New database: DBx1000
  * Support all seven classic concurrency control algorithms
* Algos:
  * Two Phase Locking
  * Timestamp Ordering
    * Timestamp
    * MVCC
    * OCC
  * Partitioning
* Solutions to Timestamp Allocation
  * Batch Timestamp Allocation
  * Hardware Counter
  * Distributed Clock
    * Must be syncronized. Clocks are hard.

#### E-Store: Fine-Grained Elastic Partitioning for Distributed Transaction Processing Systems, Rebecca Taft (MIT)

* Problem: Workload Skew
  * Skew: 40-60% of NYSE volume is on 40 stocks
  * Load Spikes: Interesting company news
  * Hockey Stick Effect: viral article
* Skew matters a lot: High skew increases latency 10x, decreases throughput 4x
* Shared nothing systems very susceptible
* Solutions:
  * Provision resources for peak load
  * Limit load on system
  * Enable system to elastically scale
* Two-Tieried Partitioning
  * Individual hot tuples mapped explicitly to partitions
  * large blocks of colder tuples
* Most existing systems partition data at coarse granularity
* EStore: extends H-Store wit automatic, adaptive, two-tiered elastic partitioning
* EStore is optimized for stored procedures, not SQL. Provide procedure name & params.

#### Rethinking serializable multiversion concurrency control, Jose Faleiro (Yale University)

* Single vs. Multiversion systems. The latters buys us extra concurrency, right?
* In Practice: Not necessarily true.
* "Solution": use conservative isolation techniques similar to single version systems
* Bohm: separate concurrency control from transaction execution
* Txn entire logic must be submitted in one piece