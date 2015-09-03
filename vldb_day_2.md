### VLDB Day 2

* ToDo
  * Look at Data Feed Management Systems (DFMS): Bistro (Sigmod 2011), AsteriskDB (EDBT 2015)
  * Must Read:  Schema-Agnostic Indexing with Azure DocumentDB paper

* Biggest Takeaways:
  * Oracle is very good at Enterprise Names
  * We've clearly reached the point where SSD/RAM bandwidth have completely outpaced CPU compute
  * LinkedIn built Yet Another Pipeline Thing
  * Do the Spark folks understand that there is a lot of operational/cognitive overhead to running the JVM?

#### Engineering Database Hardware and Software Together, Juan Loaiza Oracle

* Integrating at system level: software + cluster hardware
* Integrating down to the *microprocessor* level
* Oracle Exadata Machine: optimized compute, networking, storage hardware
* Exadata hardware: Compute, Networking, Storage (PCIe flash)
* Data warehousing: achieve throughput of Flash
* Throughput of Flash *much* higher than network, so push some compute to storage
* Oracle Exadata: 1/2 deployments OLTP, 1/2 OLAP/Analytics. Definitely some mixed workloads.
* Some say you need specialized product, Oracle disagrees (naturally)
* Oracle says it's hard to move data between products. Literally every startup disagrees.
* Oracle thinks specialized algos can allow for single product
* Oracle's next big focus: In-memory Database
* Wanted "Real Time Analytics" on top of existing OLTP data
* Oracle Dual Format: They're writing row data to columns on txn commit. Basically keeping two copies of data.
* Columns let them lean on SIMD for vectorized processing
* Oracle Next Big Thing: SPARC M7 processor
* Database-optimized microprocessor, some algorithms right on the chip
* Essentially have "where-clause engines", specialized compute for vectorized instructions
* "Decompressed engines", decompress-specific compute
* "Database Accelerator", "DAX": Pipelined streaming engine on the chip itself
* Essentially optimizing the chip itself for database-specific algorithms
  * Bloom filter
  * Predicate eval
  * Bit-vector filtering
  * Run Length Expansion
* Has SRAM buffer to hold dictionaries and lookup tables

#### DBs and hardware: the beginning and sequel of a beautiful friendship, Anastasia Ailamaki

* Recurring headaches:
  * Non-uniform communication
  * Underutilization of resources
  * One size fits None
* OLTP: bound by access latency to memory
* OLAP: similar, but bounding factor is memory bandwidth
* Adaptive partitioning: dynamically allocate based on workload
* OLTP: communication islands, intra-core/L1/L2/L3, inter-socket
* OLAP: partitioning across sockets to parallelize access to data
  * 1 hot table: throughput goes up with # partitions
  * 3 hot tables: can oversaturate bandwidth with more partitions
  * Fewer partitions -> higher throughput by avoiding bandwidth saturation
* Underutilization of resources: processor is idle most of the time!
* Vitamin: instead of building the system to be cache conscious, chase locality in L1
* One DMBS fits none
* Just-in-time databases: access only useful raw data, no copying or altering data, no vendor lock-in, custom (per-query) engine

#### Industrial 1

#### FIT to monitor feed quality, AT&T Labs

* People don't take data quality into account when making decisions
* Make tools that help people make fast decisions given data quality issues
* Data Feeds: hundreds of feeds that populate databases, data lakes
  * 1000s of files of varying size, format, content
* Data Feed Management Systems (DFMS): Bistro (Sigmod 2011), AsteriskDB (EDBT 2015)
* Multiple feed publishers -> DFMS -> multiple subscribers
* DFMS is perfect platform for data quality module
* Why a Data Feed/Data Quality Module?
  * Early error detection can help with quick remediation
  * Many errors in downstream DBs due to errors in upstream feeds
  * Alert upstream publishers they have problems sooner
  * Lightens DQ burden on individual subscriber systems
* DQ module inspects feeds for data quality issues
  * Missing files, too few/many files, too small/big files
  * Feeds are stalled, files are late
  * Messages are mangled
* What is FIT?
  * Feed Inspection Tool
  * Data quality tool to continuously monitor data feeds
  * Written in...R?
* FIT compares current time partition with historical data, checks:
  * Timeliness: was the data delayed?
  * Completeness: did we receive all the data?
  * Anomalies: is the data consistent with expectations?
  * Other DQ metrics, such as mangled schemas, etc
* Use case: Data Lake
  * Feed logs for high volume feeds
  * Pipeline:
    * Data extraction (JSON, etc)
    * Data aggregation frequency is flexible: hourly, daily
    * Models: non-parametric, continuously adapting, multi-scale
    * Multiple alerting frequencies

#### ConfSeer: Leveraging Customer Support Knowledge Bases for Automated Misconfiguration Detection, Microsoft

* Motivation: Huge Impact of Misconfigurations
* Goal: Configuration-Daignosis-As-A-Service
* Goal 1: Automated detection of configuration errors from the software config files
* Goal 2: Recommend technical solution (e.g. KB article) to solve the misconfig
* Challenge 1: KB articles are free-form natural language text
* Challange 2: Automatically detect math constraints from free-form text. Ex: "Is NumDBCopies < 3?"
* Key challenges:
  * KB articles natural language
  * Identifying mathematical constrants from text
  * Huge range of parameter values- what to choose?
* ConfSeer: Automate mapping of misconfigs to their solutions in KB articles
* NLP, IR, and Learning
* 80-97.5% accuracy

#### Gobblin: Unifying Data Ingestion for Hadoop, LinkedIn

* Data Ingestion Challenges in LinkedIn
  * Many data sources (Kafka, databases, logs, S3, google analytics).
  * Many data types (structured, unstructured, batches, single-messages)
  * Operational pain
* All data ingestion goes through Gobblin
* Requirements
  * Rich Source Integration
    * Integrated with as many data sources as possible
  * Centralized State Mgmt for different pipelines
  * Multi-platform and Scalability Support
  * Extendable
  * Self-service
  * Operable
* Architecture
  * Constructs for Building Ingestion Flows
    * Source
    * Extractor
    * Converter
    * Quality Checker
    * Writer
    * Publisher
  * WorkUnit/Task
  * Execution Runtime
  * Deployment Modes
    * Standalone
    * Hadoop MR
    * Yarn
* Case Study: Kafka Ingestion
  * KafkaAvroSource ->
  * WorkUnits (Topic/Partition) ->
  * KafkaAvroExtractor ->
  * Avro ->
  * KafkaConverter ->
  * TimePartitionedAvroWriter ->
  * AuditCountQualityChecker ->
  * TimePartitionedDataPublisher ->
* Case Study: Database Ingestion
  * JDBCSource ->
  * WorkUnits (Time Ranges) ->
  * JDBCExtractor ->
  * Row ->
  * ToAvroConverter ->
  * SnapshotAvroWriter ->
  * SchemaCompatibility & Count Quality Checker ->
  * SnapshotDataPublisher
* Case Study: Filtering Sensitive Data
  * Source ->
  * WorkUnit ->
  * Extractor ->
  * Converter and Quality Checker ->
  * Fork and Branching ->
  * HasSensitiveData? ->
  * No? -> Writer
  * Yes? -> SensitiveDataConverter -> Writer
* On the fly Data Quality Checking
  * Per record or task checking
  * Policy driven
  * Composable
    * Schema compat
    * Audit check
    * Sensitive fields
    * Required fields
    * Unique key
* State and Metadata Mgmt
  * Stores runtime metadata, e.g. checkpoints (watermarks)
  * Default impl: serialized job/task states into files, one per run
  * Allows other impls that conform to the interface to be plugged in
* Metrics/Events and Alerting
  * Metrics for ingestion progress
  * Events for major milestones
  * Various built-in metric/event reporters
* Running Modes
  * Standalone: single JVM, tasks in threadpool
  * Gobblin on Map-Reduce
  * Yarn

#### Schema-Agnostic Indexing with Azure DocumentDB, Microsoft

* DocumentDB: fully managed, multi-tenant, geo-distributed document database service on Azure
* Three core capabilities
  * Built from the ground up with resource governance
    * Provisioned throughput, performance isolation, OPEX efficiency
  * Well defined consistency levels with predictable performance
    * Strong
    * Bounded Staleness
    * Session
    * Eventual
  * Database engine built for JSOn & JavaScript
    * Automatic indexing of JSON, rich (SQL & JavaScript query)
    * JavaScript language integrated transactions and query directly inside the database
* Schema agnostic indexing
  * JSON document as tree
* Index is a union of all the document trees
* Queries operate on trees
