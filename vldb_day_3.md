### VLDB Day 3

* ToDo
  * "Utility" slide for Big Plateaus of Big Data talk
  * Read "Coordination Avoidance in Database Systems" paper
  * Read "Schema Management for Document Stores" paper
  * Read "Supporting Scalable Analytics with Latency Constraints" paper
  * Read "Gorilla: A Fast, Scalable, In-Memory Time Series Database" paper

* Biggest Takeaways:
  * Great keynote: Big Data Research: Will Industry Solve All the Problems? Magdalena Balazinska, University of Washington
  * Peter Bailis work on Coordination Avoidance is really interesting and important.
  * The Principles of Dataset Versioning talk is interesting, and trying to address a very real problem in many teams
  * Google DataFlow: Lambda Architecture largely a consequence of immaturity in streaming systems

#### Big Plateaus of Big Data on the Big Island, Todd Waters Teradata

* Agree with Stonebraker: #1 thing to be successful is to talk to users
* What is Big?
  * In 1979, 200MB weighed 30 lbs and took up the space of a washing machine
* Real world works in big phase changes, in big leaps. Data grows slowly over the years.
  * Entirely new dataset enabled by new tech at new price point- value has to exceed cost (ROI)
  * Earliest adopters have vision ahead of ROI
* Market Basket Example
  * Retailers were tracking daily summary sales data
  * Visionaries said "we can get value from tracking every item", when sold, what combinations, etc
  * 250k -> 10M records per store per day
* Teradata installing a 100PB system by the end of the year
* Communications customer: 200B incoming network records/day, LTE rollout will generate 500B records/day
* "Todays developer class tools will have to give way to self-service applications"
* "Schema on read is interesting, but you dump some data on users and you are in deep yogurt"
* Anti-Database: How to throw away the RIGHT data?
* CERN spends more compute power throwing away data than storing and analyzing it

#### Big Data Research: Will Industry Solve All the Problems? Magdalena Balazinska, University of Washington

* Tremendous amount of industry activity
* How can academia contribute?
* What goes around comes around: these problems have been worked on for 40 years
* What is next? How can academia contribute if industry has all the workloads?
* There are interesting workloads on campus! Astronomers, scientists, etc
* But what about the tools?
  * Apply open source tools. Most likely these tools will break in interesting ways!
  * Fix them and contribute back to open-source tools
* But industry has more engineers!
  * We don't compete with industry for engineers- we provide them!

#### Coordination Avoidance in Database Systems, Peter Bailis (UC Berkeley)

* Serializable transactions are not as widely deployed as you think...
* Serializability: equivalence to some serial execute
* Very general! But restricts concurrency!
* Serializability requires Coordination
  * Txns cannot make progress independently
* Costs of Coordination between Concurrent Txns
  * Decreased perf
  * Decreased availability during failures
* Serializability => Consistency
* When is coordination strictly necessary?
* When can we safely avoid coordination?
* Semantic concurrency control is back, works, and is needed
* Key idea: Check if constraints can be violated by merging independent operations

#### Principles of Dataset Versioning: Exploring the Recreation/Storage Tradeoff, Souvik Bhattacherjee (University of Maryland (College Park)

* Typical data analysis workflow is akin to people collaborating on source code
* Many collaborative data science projects end up in dataset version management hell
* How can we store thousands of versions of datasets compactly, and access them easily?
* VCS are optimized for code-like data, not data-like-data
* Do we store entire copies of datasets, or just deltas with some sort of reconstruction pipeline?
* How do we minimize retrieval cost within a given storage budget?

#### Schema Management for Document Stores, Lanjun Wang (IBM Research-China)

* Common data management tasks facilitated by schema become very challenging
* What does Schema Management look like for document stores?
* Framework:
  * Schema Extraction and Discovery
  * Schema Repository
  * Schema Management APIs
* Problem 1: How to efficiently discover all distinct schemas
  * Multiple object types in single collection
  * Schemas of same object type vary

#### Supporting Scalable Analytics with Latency Constraints, Boduo Li (University of Massachusetts Amherst)

* What are the key design criteria for big and fast analytics systems?
* 1. Data Parallelism 2. From Batch to Incremental Processing 3. (Near) Real Time Processing

#### The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing, Google

* Motivations
  * Unbounded (infinite sets of growing data)
  * Unordered (can't assume static delay, because of nature of distributed data sources)
    * Verying event-time skew
    * Reasoning about completeness is very difficult
* Sophisticated Windows
  * Fixed
  * Sliding
  * Sessions
* Tensions
  * Correctly
  * Latency
  * Cost
* Data Processing Story
  * Batch: Bounded Data
    * MapReduce
  * Batch: Sessions
* Filtering
* Approximation
* Windowing by processing time
* Windowing by event time (fixed windows)
* Lambda Architecture largely a consequence of immaturity in streaming systems
* DataFlow: single model, single system, strongly consistent
* Natural processing of Bounded & Unbounded Data
  * Unified Model
  * Out of Order Processing
  * Windowing (temporal and custom)
  * Tunable correctness/latency/cost
* Four questions:
  * What are you computing?
  * Where in event time?
  * When in processing time?
  * How do refinements relate?
* What: Transformations
* Where: Windowing
* When: Watermarks + Triggers
* How: Accumulation

#### Gorilla: A Fast, Scalable, In-Memory Time Series Database, Facebook

* Why timeseries DB?
  * Ops data
  * System stats
  * Perf counters
  * Application metrics
* 2 types queries
  * Continuous
    * Dashboards, detectors
  * Exploratory
    * Debugging
* Why in-memory?
  * HBase is high throughput, but also high latency
  * 90th percentile query time was > 2s
  * Users were self-censoring queries
  * Caching doesn't solve the problem
* Motivation
  * > 2B time series
  * > 700M data points per minute
  * 26 hours of retention
  * > 40k queries per second
  * ...16TB for 26 hours
  * 2x in-memory replication
  * Support 2x growth per year