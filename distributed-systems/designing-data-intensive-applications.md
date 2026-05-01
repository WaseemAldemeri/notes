# Designing Data-Intensive Applications

by Martin Kleppmann and Chris Riccomini

<!-- vim-markdown-toc GFM -->

* [Progress](#progress)
* [Chapter 1: Trade-Offs in Data Systems Architecture](#chapter-1-trade-offs-in-data-systems-architecture)
* [Chapter 2: Defining Nonfunctional Requirements](#chapter-2-defining-nonfunctional-requirements)
* [Chapter 3: Data Models and Query Languages](#chapter-3-data-models-and-query-languages)
* [Chapter 4: Storage and Retrieval](#chapter-4-storage-and-retrieval)
  * [Data Storage for Analytics](#data-storage-for-analytics)
  * [Multidimensional and Full-Text Indexes](#multidimensional-and-full-text-indexes)
* [Chapter 5: Encoding and Evolution](#chapter-5-encoding-and-evolution)
  * [Dataflow modes — who encodes and who decodes](#dataflow-modes--who-encodes-and-who-decodes)
* [Chapter 6: Replication](#chapter-6-replication)
* [Chapter 7: Sharding](#chapter-7-sharding)
* [Chapter 8: Transactions](#chapter-8-transactions)
* [Chapter 9: The Trouble with Distributed Systems](#chapter-9-the-trouble-with-distributed-systems)
* [Chapter 10: Consistency and Consensus](#chapter-10-consistency-and-consensus)
* [Chapter 11: Batch Processing](#chapter-11-batch-processing)
* [Chapter 12: Stream Processing](#chapter-12-stream-processing)
* [Chapter 13: A Philosophy of Streaming Systems](#chapter-13-a-philosophy-of-streaming-systems)

<!-- vim-markdown-toc -->

## Progress

- [x] Chapter 1: Trade-Offs in Data Systems Architecture
- [x] Chapter 2: Defining Nonfunctional Requirements
- [x] Chapter 3: Data Models and Query Languages
- [x] Chapter 4: Storage and Retrieval
- [x] Chapter 5: Encoding and Evolution
- [ ] Chapter 6: Replication
- [ ] Chapter 7: Sharding
- [ ] Chapter 8: Transactions
- [ ] Chapter 9: The Trouble with Distributed Systems
- [ ] Chapter 10: Consistency and Consensus
- [ ] Chapter 11: Batch Processing
- [ ] Chapter 12: Stream Processing
- [ ] Chapter 13: A Philosophy of Streaming Systems

---

## Chapter 1: Trade-Offs in Data Systems Architecture

> many applications need to do the following:
> - store data so that they, or another application, can find it again later (databases)
> - Remember the result of an expensive operation, to speed up reads (caches)
> - Allow users to search data by keyword or filter it in various ways (search indexes)
> - Handle events and data changes as soon as they occur (stream processing)
> - Periodically crunch a large amount of accumulated data (batch processing)

> - Operational systems consist of the backend services and data infrastructure where data is created—for example, by serving external users. Here, the application code both reads and modifies the data in its databases, based on the actions performed by the users.
> - Analytical systems serve the needs of business analysts and data scientists. They contain a read-only copy of the data from the operational systems, and they are optimized for the types of data processing that are needed for analytics

> Data engineers are the people who know how to integrate the operational and analytical systems and who take responsibility for the organization’s data infrastructure more widely. Analytics engineers model and transform data to make it more useful for the business analysts and data scientists in an organization


> BI: Business intelligence

> . An operational system typically looks up a small number of records by a key (this is called a point query). Records are inserted, updated, or deleted based on the user’s input. Because these applications are interactive, this access pattern became known as **online transaction processing (OLTP)**.

> The reports that result from these types of queries are important for BI, helping management decide what to do next. To differentiate this pattern of using databases from transaction processing, it has been called online analytical processing **(OLAP)**

> A data warehouse, is a separate database that analysts can query to their hearts’ content, without affecting OLTP operations

>The data warehouse contains a read-only copy of the data from all the various OLTP systems in the company. Data is extracted from OLTP databases (using either a periodic data dump or a continuous stream of updates), transformed into an analysis friendly schema, cleaned up, and then loaded into the data warehouse. This process of getting data into the data warehouse is known as **extract–transform–load (ETL)**

> Consequently, organizations face a need to make data available in a form that is suitable for use by data scientists. The answer is a data lake: a centralized data repository that holds a copy of any data that might be useful for analysis, obtained from operational systems via ETL processes. The difference from a data warehouse is that a data lake simply contains files, without imposing any particular file format, data model, or schema. Files in a data lake might be collections of database records, encoded using a file format such as Avro or Parquet, but a data lake can equally well contain text, images, videos, sensor readings, sparse matrices, feature vectors, genome sequences, or any other kind of data. Besides being more flexible, a data lake is also often cheaper than relational data storage, since it can use commoditized file storage such as object stores

> The data lake contains data in the “raw” form produced by the operational systems, without the transformation into a relational data warehouse schema. This approach has the advantage that each consumer of the data can transform the raw data into the form that best suits their needs. It’s sometimes called the sushi principle: “raw data is better”

> Systems of record
>> A system of record, also known as a source of truth, holds the authoritative or canonical version of data. When new data comes in—for example, as user input— it is first written here. Each fact is represented exactly once (the representation is typically normalized

> Derived data systems
>> Data in a derived system is the result of taking existing data from another system and transforming or processing it in some way. If you lose derived data, you can re-create it from the original source. A classic example is a cache: data can be served from the cache if present, but if the cache doesn’t contain what you need, you can fall back to the underlying database. Denormalized values, indexes, materialized views, transformed data representations, and models trained on a dataset also fall into this category.


> Cloud services are particularly valuable if the load on your systems varies a lot over time. If you provision your machines to be able to handle peak load, but those computing resources are idle most of the time, the system becomes less cost-effective. In this situation, cloud services have the advantage that they can make it easier to scale your computing resources up or down in response to changes in demand

> Traditionally, the people managing an organization’s server-side data infrastructure were known as database administrators (DBAs) or system administrators (sysadmins). More recently, many organizations have tried to integrate the roles of software devel‐ opment and operations into teams with a shared responsibility for both backend services and data infrastructure; the DevOps philosophy has guided this trend. Site reliability engineers (SREs) are Google’s implementation of this idea

> A system that involves several machines communicating via a network is called a distributed system. Each of the processes participating in a distributed system is called a node. 

> Troubleshooting a distributed system is often difficult—if the system is slow to respond, how do you figure out where the problem lies? Techniques for diagnosing problems in distributed systems are developed under the heading of observability which involves collecting data about the execution of a system and allowing that data to be queried in ways that allow both high-level metrics and individual events to be analyzed. Tracing tools such as OpenTelemetry, Zipkin, and Jaeger allow you to track which client called which server for which operation and how long each call took.

> The most common way of distributing a system across multiple machines is to divide them into clients and servers and let the clients make requests to the servers. Most commonly, HTTP is used for this communication.The same process may be both a server (handling incoming requests) and a client (making outbound requests to other services).

> This way of building applications has traditionally been called a service-oriented architecture (SOA); more recently, the idea has been refined into a microservices architecture. In a microservices architecture, a service has one well-defined purpose (e.g., in the case of S3, this is file storage); each service exposes an API that can be called by clients via the network, and each service has one team that is responsible for its maintenance. A complex application can thus be decomposed into multiple interacting services, each managed by a separate team. Cloud native systems make heavy use of decomposition into services, but on-premises systems can use a service-oriented approach too

> Serverless, or function as a service (FaaS), is another approach to deploying services, in which the management of the infrastructure is outsourced to a cloud vendor. When using VMs, you have to explicitly choose when to start up or shut down an instance; in contrast, with the serverless model, the cloud provider automatically allo‐ cates and frees hardware resources as needed, based on the incoming requests to your service. Just as cloud storage replaced capacity planning (deciding in advance how many disks to buy) with a metered billing model, the serverless approach is bringing metered billing to code execution: you pay only for the time that your application code is running rather than having to provision resources in advance.

## Chapter 2: Defining Nonfunctional Requirements

> examples of nonfunctional requirements: performance, reliability, scalability, and maintainability

> When one initial request results in several downstream requests being carried out, we use the term fan-out to describe the factor by which the number of requests increases.

> This process of precomputing and updating the results of a query is called materialization, and the timeline cache is an example of a materialized view (a concept we will discuss further in later chapters). The materialized view speeds up reads, but in return we have to do more work on writes.

> Response time
>> The elapsed time from the moment when a user makes a request until they receive the requested answer. The unit of measurement is seconds (or milliseconds, or microseconds).

> Throughput
>> The number of requests per second, or the data volume per second, that the system is processing. For a given allocation of hardware resources, there is a maximum throughput that can be handled. The unit of measurement is “some things per second.”


> If a system is close to overload, with throughput pushed close to the limit, it can sometimes enter a vicious cycle where it becomes less efficient and hence even more overloaded. For example, if a long queue of requests is waiting to be handled, response times may increase so much that clients time out and resend their requests. This causes the rate of requests to increase even further, making the problem worse— a retry storm. Even when the load is reduced again, such a system may remain in an overloaded state until it is rebooted or otherwise reset. This phenomenon is called a metastable failure, and it can cause serious outages in production systems . To avoid retries overloading a service, you can increase and randomize the time between successive retries on the client side (exponential backoff) and tem‐ porarily stop sending requests to a service that has returned errors or timed out recently (by using a circuit breaker  or token bucket algorithm). The server can also detect when it is approaching overload and start proactively rejecting requests (load shedding), or send back responses asking clients to slow down (backpressure). The choice of queueing and load balancing algorithms can also make a difference.


- The response time is what the client sees; it includes all delays incurred anywhere in the system.
- The service time is the duration for which the service is actively processing the client’s request.
- Queueing delays can occur at several points in the flow—for example, after a request is received, it might need to wait until a CPU is available before it can be processed, or a response packet might need to be buffered before it is sent over the network if other tasks on the same machine are sending a lot of data via the outbound network interface.
- Latency is a catchall term for time during which a request is not being actively processed—that is, during which it is latent. In particular, network latency or network delay refers to the time that a request and response spend traveling through the network.

> Queueing delays often account for a large part of the variability in response times. As a server can process only a small number of things in parallel (limited, for example, by its number of CPU cores), it takes only a small number of slow requests to hold up the processing of subsequent requests—an effect known as head-of-line blocking. Even if those subsequent requests have fast service times, the client will see a slow overall response time due to the time waiting for the prior request to complete. The queueing delay is not part of the service time, and for this reason it is important to measure response times on the client side

> Percentiles are often used in service level objectives (SLOs) and service level agreements (SLAs) as ways of defining the expected performance and availability of a service. For example, an SLO may set a target for a service to have a median response time of less than 200 ms and a 99th percentile under 1 second, and a target that at least 99.9% of valid requests result in non-error responses. An SLA is a contract that specifies what happens if the SLO is not met (e.g., customers may be entitled to a refund). That’s the basic idea, at least; in practice, defining good availability metrics for SLOs and SLAs is not straightforward

> Fault
>> A fault occurs when a particular part of a system stops working correctly—for example, if a single hard drive malfunctions, or a single machine crashes, or an external service (that the system depends on) has an outage.

> Failure
>> A failure occurs when the system as a whole stops providing the required service to the user—in other words, when it does not meet the SLO.

>  good general principle for scalability is to break a system into smaller components that can operate largely independently from one another. This is the underlying principle behind microservices (see “Microservices and Serverless” on page 21), sharding (Chapter 7), stream processing (Chapter 12), and shared-nothing architectures


## Chapter 3: Data Models and Query Languages

> Many of the query languages discussed in this chapter (such as SQL, Cypher, SPARQL, and Datalog) are declarative, which means that you specify the pattern of the data you want—what conditions the results must meet and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal. The database system’s query optimizer can decide which indexes and join algorithms to use and in which order to execute various parts of the query. In contrast, with most programming languages (such as Python and Java), you would have to write an algorithm telling the computer which operations to perform in which order.

> A one-to-many relationship is sometimes called one-to-few, since a résumé typically has a small number of positions [11, 12]. If you have a genuinely large number of related items—say, comments on a celebrity’s social media post, of which there could be many thousands—embedding them all in the same document may be too unwieldy, so the relational approach in Figure 3-1 is preferable.


> As a general principle, normalized data is usually faster to write (since there is only one copy) but slower to query (since it requires joins); denormalized data is usually faster to read (fewer joins) but more expensive to write (more copies to update, more disk space used)


> we also have many-to-many relationships (one person may have worked for several organizations, and an organization has several past or present employees). In a relational model, such a relationship is usually represented as an associative table, or join table

> shows an example of a star schema that might be found in the data warehouse of a grocery retailer. At the center of the schema is a so-called fact table (in this example, it is called fact_sales). Each row of the fact table represents an event that occurred at a particular time (here, each row represents a customer’s purchase of a product).
> Some of the columns in the fact table are attributes, such as the price at which the product was sold and the cost of buying it from the supplier (allowing the profit margin to be calculated). Other columns in the fact table are foreign-key references to other tables, called dimension tables. As each row in the fact table represents an event, the dimensions represent the who, what, where, when, how, and why of the event.
> The name star schema comes from the fact that when the table relationships are visualized, the fact table is in the middle, surrounded by its dimension tables
> A variation of this template is the snowflake schema, where dimensions are further broken into subdimensions. For example, there could be separate tables for brands and product categories, and each row in the dim_product table could reference the brand and category as foreign keys, rather than storing them as strings in the dim_product table. Snowflake schemas are more normalized than star schemas, but star schemas are often preferred because they are simpler for analysts to work with

>Some data warehouse schemas take denormalization even further and leave out the dimension tables entirely, folding the information in the dimensions into denormal‐ ized columns in the fact table instead (essentially, precomputing the join between the fact table and the dimension tables). This approach is known as one big table (OBT), and while it requires more storage space, it sometimes enables faster queries

> In the context of analytics, such denormalization is unproblematic, since the data typ‐ ically represents a log of historical data that is not going to change (except maybe for occasionally correcting an error). The issues of data consistency and write overheads that occur with denormalization in OLTP systems are not as pressing in analytics.

> Document databases are sometimes called schemaless, but that’s misleading as the code that reads the data usually assumes some kind of structure—that is, there is an implicit schema, but it is not enforced by the database [19]. A more accurate term is schema-on-read (the structure of the data is implicit and interpreted only when the data is read), in contrast with schema-on-write (the traditional approach of relational databases, where the schema is explicit and the database ensures that all data conforms to it when the data is written)

> But what if many-to-many relationships are very common in your data? The relational model can handle simple cases of many-to-many relationships, but as the connections within your data become more complex, it becomes more natural to start modeling that data as a graph. A graph consists of two kinds of objects: vertices (also known as nodes or entities) and edges (also known as relationships or arcs). Many kinds of data can be modeled as a graph.

> You can think of a graph store as consisting of two relational tables, one for vertices and one for edges, as shown in Example 3-3 (this schema uses the PostgreSQL jsonb datatype to store the properties of each vertex or edge). The head and tail vertices are stored for each edge; if you want the set of incoming or outgoing edges for a vertex, you can query the edges table by head_vertex or tail_vertex, respectively. (page 87)

> graphs are nice for deep nested many to many relationships like frinds of frineds of frineds; SQL performance falls off a cliff. Graphs handle this natively.

> Graphs prioritize the relationship as a first-class citizen, whereas Relational/Document models prioritize the entity.

> know that languages exist to query graph data easier than sql such as Cypher, SPARQL, Datalog

> GraphQL is a query language that, by design, is much more restrictive than the others we have seen in this chapter. It’s intended for OLTP queries; its purpose is to allow client software running on a user’s device (such as a mobile app or a rontend) to request a JSON document with a particular structure, containing the fields necessary for rendering its UI.

> Even though the response to a GraphQL query looks similar to a response from a document database, and even though it has “graph” in its name, GraphQL can be implemented on top of any type of database—relational, document, or graph.

> The idea of using events as the source of truth and expressing every state change as an event is known as event sourcing [65, 66]. The principle of maintaining separate read-optimized representations and deriving them from the write-optimized representation is called command query responsibility segregation (CQRS) [67]. These terms originated in the DDD community, although similar ideas have been around for a long time—e.g., in state machine replication

> Event Sourcing summary: instead of storing only the current state of your data, you treat every state change as an immutable event and append it to a log — that log becomes the source of truth. Current state is just a derived view you rebuild by replaying events, and you can maintain multiple read-optimized projections from the same log (this split between the write-side log and read-side views is CQRS). The benefits are auditability, time-travel/debugging, and the flexibility to derive new representations later; the cost is more complexity and the need to handle schema evolution of old events.

> The DataFrame data model is supported by the R language, the Pandas library for Python, Apache Spark, ArcticDB, Dask, and other systems. DataFrames are a popular tool for data scientists preparing data for training ML models, but they are also widely used for data exploration, statistical data analysis, data visualization, and similar purposes.

## Chapter 4: Storage and Retrieval

> In principle, you could maintain a hash table on disk, but unfortunately it is difficult to make an on-disk hash map perform well. It requires a lot of random access I/O, it’s expensive to grow when it becomes full, and hash collisions require fiddly logic

> In practice, hash tables are not used very often for database indexes. Instead, it is much more common to keep data in a structure that is sorted by key [3]. One example of such a structure is a Sorted Strings Table, or SSTable for short, as shown in Figure 4-2. This file format also stores key-value pairs, but it ensures that they are sorted by key, and each key appears only once in the file.

LSM-tree storage engines (LevelDB, RocksDB, Cassandra) solve the "sorted-on-disk is great for reads but terrible for writes" problem by separating where writes land from where reads search: incoming writes go into an in-memory ordered structure called the *memtable* (a red-black tree, skip list, or trie — all kept sorted cheaply in RAM), and once it grows past a threshold it's flushed sequentially to disk as an immutable SSTable, while a fresh memtable takes over so writes never block. Reads check the memtable first, then walk newest-to-oldest through the SSTables until the key is found, with updates and deletes (tombstones) just being newer entries that shadow older ones. A background compaction process mergesorts overlapping SSTables together — cheap because each input is already sorted — discarding overwritten and deleted values to keep read amplification and disk usage bounded. Durability comes from a separate write-ahead log that every write is appended to before hitting the memtable, so a crash can be recovered by replaying the log; once a memtable is safely flushed, its portion of the log is discarded. The "tree" in LSM-tree is largely historical branding from the 1996 O'Neil paper — modern implementations have no single tree structure, just a tiered hierarchy of sorted files cascading through compaction levels.


> Many databases run as a service that accepts queries over a network, but there are also embedded databases that don’t expose a network API. Instead, they are libraries that run in the same process as your application code, typically reading and writing files on the local disk, and you interact with them through normal function calls. Examples of embedded storage engines include RocksDB, SQLite, LMDB, DuckDB, and KùzuDB 

> o make the database resilient to crashes, it is common for B-tree implementations to include an additional data structure on disk: a write-ahead log (WAL). This is an append-only file to which every B-tree modification must be written before it can be applied to the pages of the tree itself. When the database comes back up after a crash, this log is used to restore the B-tree back to a consistent state [2, 24]. In filesystems, the equivalent mechanism is known as journaling.

> As a rule of thumb, LSM-trees are better suited for write-heavy applications, whereas B-trees are faster for reads

> Range queries are simple and fast on B-trees, as they can make use of the sorted structure of the tree. On LSM storage, range queries can also take advantage of the SSTable sorting, but they need to scan all the segments in parallel and combine the results. Bloom filters don’t help for range queries (since you would need to compute the hash of every possible key within the range, which is impractical), making range queries more expensive than point queries in the LSM approach

Indexes summary: only the primary key is indexed automatically. Every other column — including foreign keys — is a sequential scan unless you add an index. An index is a separate on-disk data structure (typically a B-tree, or a stack of SSTables for LSM engines) that keeps one column sorted and supports `O(log n)` lookups, range scans, and inserts. The cost: extra disk space proportional to the indexed column, plus a write-amplification tax — every `INSERT`/`UPDATE`/`DELETE` has to update every relevant index. So indexes are a read-vs-write trade-off: you pay on writes to make specific reads fast, and the art is picking *which* columns deserve that tax based on actual query patterns. Secondary indexes differ from primary indexes in that their values aren't unique, so the leaf entry is either a list of row pointers, a pointer to the row in a heap file (Postgres), or the primary key requiring a second lookup (InnoDB). Variants like *partial indexes* (index only rows matching a `WHERE` clause) and *covering indexes* (include extra columns so queries never touch the table) widen the trade-off space.

### Data Storage for Analytics

> a data warehouse and a relational OLTP database look similar, because they both have a SQL query interface. However, the internals of the systems can look quite different, because they are optimized for very different query patterns.


> Although fact tables are often over one hundred columns wide, a typical data warehouse query accesses only four or five of them at one time ... a row-oriented storage engine still needs to load all those rows (each consisting of over 100 attributes) from disk into memory, parse them, and filter out those that don't meet the required conditions.

> The idea behind column-oriented (or columnar) storage is simple: instead of storing all the values from one row together, store all the values from each column together instead. If each column is stored separately, a query needs to read and parse only those columns that are used in that query

> we can further reduce the demands on disk throughput and network bandwidth by compressing data ... Often the number of distinct values in a column is small compared to the number of rows ... We can now take a column with n distinct values and turn it into n separate bitmaps


Object storage aside (not in DDIA): **object storage** (S3, GCS, Azure Blob) stores data as discrete blobs in a flat key→bytes namespace, accessed over HTTP. Unlike block storage (raw device, random read/write) or file storage (POSIX directories), objects are immutable in practice — you replace the whole object, not edit bytes in place. Properties that make it the substrate for cloud warehouses: very cheap (~$0.02/GB/mo), effectively infinite capacity, 11 nines durability via automatic replication, massive aggregate throughput, but high per-request latency (tens of ms). That latency profile is fine for "read a 100 MB Parquet file" but useless for OLTP. Immutability is also why analytical formats (Parquet) are write-once and table formats (Iceberg, Delta) layer inserts/deletes/transactions on top by tracking which files make up a table at each version. Mental model: a giant HTTP-addressable KV store where values are large files, optimized for throughput and durability over latency or in-place edits.

Postgres aside (not in DDIA): **TOAST** = "The Oversized-Attribute Storage Technique." Values too big for an 8 KB page (>~2 KB) get compressed and/or moved to a side table, with a pointer in the main row. So `SELECT *` and `SELECT x, y, z` read the same heap pages, but `*` additionally chases TOAST pointers for any large columns — a narrow projection skips that, plus enables index-only scans and cuts network/sort/hash work downstream.

### Multidimensional and Full-Text Indexes

B-trees and LSM-trees are 1D — they sort by a single key. A concatenated index `(lastname, firstname)` works left-to-right only: it can find all Smiths but not all Johns. The moment you need to filter on multiple dimensions *simultaneously*, ordered single-key indexes break down.

**Multidimensional indexes** — for queries like "restaurants in this lat/lon box" or "(date, temperature)" ranges. Two practical options: encode coordinates onto a 1D space-filling curve (Z-order, Hilbert) and use a normal B-tree, or use a real spatial index (R-tree, Bkd-tree). PostGIS = R-trees over Postgres GiST. Reach for these when both dimensions are equally selective.

**Full-text search** = the same idea taken to thousands of dimensions, where each term is a dimension. The data structure is the **inverted index**: term → postings list (IDs of documents containing it). "Docs with X AND Y" = bitwise AND of two postings bitmaps — same trick as columnar bitmap indexes. Lucene (Elasticsearch, Solr) stores postings in SSTable-like sorted files with LSM-style merging; Postgres GIN does the same. **n-gram/trigram** indexes index every length-n substring instead of words, enabling substring and regex search at the cost of size (Postgres `pg_trgm`). Lucene supports typo-tolerant search via finite-state automata + Levenshtein automata.

**Vector embeddings (semantic search)** — embedding models (Word2Vec, BERT, GPT, multimodal) map text/images/audio to a 1,000+ dimensional float vector where semantic neighbors end up geometrically close (cosine similarity / Euclidean distance). R-trees fail in high dimensions, so vector indexes use specialized structures: **Flat** (brute-force, accurate but O(N)), **IVF** (cluster space into centroids, query nearest partitions; tunable via "probes"), **HNSW** (multi-layer proximity graph, descend from sparse top to dense bottom — fastest in practice). Both IVF and HNSW are *approximate*. Implementations: Faiss, pgvector, and every vector DB (Pinecone, Weaviate, Milvus, Chroma). Terminology gotcha: "vectorized processing" (SIMD batches) and "vector embeddings" (semantic floats) are unrelated meanings of "vector."

Practical takeaway: index design follows query shape. One column, range queries → B-tree/LSM. Two-or-three structured dimensions → R-tree or geohash. Keyword search → inverted index. "Find similar meaning" → ANN vector index (HNSW by default). The same primitives — sorted runs, postings lists, bitmaps, hierarchical partitioning — keep recombining.

## Chapter 5: Encoding and Evolution

> With server-side applications you may want to perform a rolling upgrade (also• known as a staged rollout), deploying the new version to a few nodes at a time, monitoring whether it runs smoothly, and gradually working your way through 161 all the nodes.

> Backward compatibility Ensures that newer code can read data written by older code. Forward compatibility Ensures that older code can read data written by newer code

> When you want to write data to a file or send it over the network, you have to encode it as some kind of self-contained sequence of bytes (e.g., a JSON document). Since a pointer wouldn’t make sense to any other process

>  The translation from the in-memory representation to a byte sequence is called encoding (also known as serialization or marshaling), and the reverse is called decoding (aka parsing, deserialization, or unmarshaling).

 IDL (Interface Definition Language): a language-neutral, formal description of data structures and/or service interfaces. An IDL specifies what an interface looks like — its types, fields, methods, and contracts — without committing to any particular implementation language. IDL definitions can be consumed in several ways: generating code (stubs, types, serializers) in target languages, validating messages at runtime, driving documentation, or enabling contract testing between independently developed components. Examples: Protobuf, Thrift, Avro, OpenAPI, GraphQL SDL, WSDL, CORBA IDL.

> Protobufs IDL schema defines the fields of each record and their types, but it does not support other restrictions on the possible values of fields.

> When HTTP is used as the underlying protocol for talking to the service, it is called a web service.

> The most popular service design philosophy is REST, which builds upon the prin‐ ciples of HTTP [31, 32]. REST emphasizes simple data formats, using URLs for identifying resources and using HTTP features for cache control, authentication, and content type negotiation. An API designed according to the principles of REST is called RESTful

> Code that needs to invoke a web service API must know which HTTP endpoint to query, and what data format to send and expect in response. Even if a service adopts RESTful design principles, clients need to somehow find out these details. Service developers often use an IDL to define and document their service’s API endpoints and data models, and to evolve them over time. Other developers can then use the service definition to determine how to query the service. The two most popular service IDLs are OpenAPI (also known as Swagger [33]), used for web services that send and receive JSON, and Protocol Buffers, used for gRPC services.

Encoding formats summary: **Language-specific** (Java Serializable, Python pickle, Ruby Marshal) — convenient but tied to one language, insecure (decoding arbitrary classes = RCE risk), versioning is an afterthought. **JSON/XML/CSV** — human-readable, universal, but ambiguous about types (no int vs float in JSON, no binary strings without Base64), CSV has no schema. Good for cross-org data exchange where format agreement matters more than efficiency. **Binary JSON variants** (MessagePack, CBOR, BSON) — modestly smaller, still must embed field names. **Schema-driven binary** (Protobuf, Thrift, Avro) — most compact, schema doubles as documentation, well-defined evolution rules, code generation for typed languages.

Protobuf vs Avro on schema evolution:
- **Protobuf** identifies fields by **tag numbers** in the schema. Field name changes are free; tag numbers are forever. Adding a field = pick a new tag (old code skips unknown tags using the type annotation — forward compat). Removing a field = retire the tag, never reuse it. New fields must be optional or have a default for backward compat. Datatype widening (int32→int64) works one direction but truncates the other.
- **Avro** has **no tag numbers** — the encoded bytes are just values concatenated in schema order, with no field identifiers or type tags inline. Decoding requires both the **writer's schema** (used to encode) and the **reader's schema** (what the app expects). Avro resolves them: fields matched by name, missing fields filled from reader's defaults, fields the reader doesn't know are skipped. To stay compatible, you can only add/remove fields that have a default value. Field renames use **aliases** in the reader's schema (backward compat only). Null requires an explicit `union { null, T }`.

> ⇒ Avro's killer feature: **dynamically generated schemas**. Because there are no tag numbers to assign by hand, you can auto-generate an Avro schema from, e.g., a relational DB schema and re-export every time the source schema changes — readers match by name. Protobuf needs manual tag assignment, so this doesn't work cleanly.

> Schema evolution allows the same kind of flexibility as schemaless/schema-on-read JSON databases provide ... while also providing better guarantees about your data and better tooling.

### Dataflow modes — who encodes and who decodes

> data outlives code

**Databases** — writer encodes, reader decodes. Need *both* directions: backward compat (future code reads old data — five-year-old rows are still in their original encoding) AND forward compat (during a rolling upgrade, an old node may read data a new node wrote). Big gotcha (Figure 5-1): an old reader that decodes into a typed object, mutates it, and writes it back can **silently drop fields it didn't know about**. Encoders/ORMs need to round-trip unknown fields. LSM compaction and "ALTER TABLE ADD COLUMN with NULL default" let DBs *appear* schema-uniform without rewriting old data — old rows get nulls filled in at read time. Archival snapshots are a free opportunity to re-encode everything in the latest schema (and maybe switch to Parquet for analytics).

**Services (REST/RPC)** — request encoded by client, decoded by server; response goes the other way. Simplifying assumption vs DBs: **servers usually upgrade before clients**, so you need backward compat on requests and forward compat on responses. Across org boundaries (public APIs) you may have to maintain compat indefinitely, often by running multiple API versions side-by-side (URL path version, `Accept` header, etc.).

> The RPC model tries to make a request to a remote network service look the same as calling a function or method, within the same process (this abstraction is called **location transparency**). Although this seems convenient at first, the approach is fundamentally flawed.


**Service discovery / load balancing spectrum**: 

  - Hardcoded IP — works for one box, breaks the moment anything moves.
  - DNS — names instead of IPs, but cached and slow to update.
  - NGINX/HAProxy — a dedicated middle box, smart and fast, but its backend list is static and every request takes an extra hop.
  - Service registry — instances register themselves, clients (or LBs) read the registry; the dynamic-environment problem is solved.
  - Service mesh — registry + per-pod sidecar proxies + central control plane; pushes discovery, security, and observability out of app code into the platform.



**Durable execution / workflows** (Temporal, Restate, Airflow, Dagster) — model a multi-step business process (workflow) as a graph of tasks/activities. The framework logs every RPC and state change to a WAL, so on retry it **replays from the log** and skips already-completed side effects, giving exactly-once semantics across third-party services that can't be wrapped in a DB transaction. Constraints: external APIs must be idempotent (use unique IDs); replay requires deterministic code (no `random()`, no `time.now()` — use the framework's deterministic versions); reordering function calls in a deployed workflow can break in-flight executions, so you deploy new versions side-by-side and let old invocations finish on the old code.

**Event-driven (message brokers, actors)** — sender encodes a message and hands it to a broker (Kafka, RabbitMQ, NATS, SQS, Pub/Sub); broker stores temporarily and delivers to one or more consumers asynchronously. Broker advantages over direct RPC: buffering when consumer is down, automatic redelivery on crash, no service discovery needed, fan-out to multiple subscribers, sender/receiver decoupled. Two delivery patterns: **queue** (one consumer wins each message) vs **topic/pub-sub** (all subscribers get a copy). Brokers are payload-agnostic — same Protobuf/Avro/JSON + schema-registry story applies. Same forward-compat trap as DBs: a consumer that re-publishes messages must preserve unknown fields.

**Distributed actors** (Akka, Erlang/OTP, Orleans) — actor model (single-threaded actors with private state, communicating via async messages) extended across a cluster. Same encode-on-send/decode-on-receive story, but location transparency works *better* here than in RPC because the actor model already assumes messages can be lost — there's no fundamental mismatch between local and remote messaging. Rolling upgrades still need forward/backward compat on the message types.

> **Takeaway:** Rolling upgrades + "data outlives code" mean any non-trivial system has multiple code versions and multiple data-format versions in flight simultaneously. The discipline of forward + backward compatibility — across DBs, RPC, message brokers, and workflow replays — is what lets a team deploy frequently without coordinating a big-bang upgrade. Schema-driven binary formats (Protobuf for hand-curated APIs, Avro when the schema is dynamically generated, e.g. from a DB) make those compat properties explicit and machine-checkable; JSON gives you flexibility but pushes the compat reasoning into your head.

## Chapter 6: Replication

> **Takeaway:** 

## Chapter 7: Sharding

> **Takeaway:** 

## Chapter 8: Transactions

> **Takeaway:** 

## Chapter 9: The Trouble with Distributed Systems

> **Takeaway:** 

## Chapter 10: Consistency and Consensus

> **Takeaway:** 

## Chapter 11: Batch Processing

> **Takeaway:** 

## Chapter 12: Stream Processing

> **Takeaway:** 

## Chapter 13: A Philosophy of Streaming Systems

> **Takeaway:** 
