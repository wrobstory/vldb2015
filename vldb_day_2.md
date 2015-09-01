### VLDB Day 2

* ToDo

* Biggest Takeaways:
  * Oracle is very good at Enterprise Names
  * We've clearly reached the point where SSD/RAM bandwidth have completely outpaced CPU compute bandwidth
  *

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
