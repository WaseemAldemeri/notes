# Designing Data-Intensive Applications

by Martin Kleppmann and Chris Riccomini

<!-- vim-markdown-toc GFM -->

* [Progress](#progress)
* [Chapter 1: Trade-Offs in Data Systems Architecture](#chapter-1-trade-offs-in-data-systems-architecture)
* [Chapter 2: Defining Nonfunctional Requirements](#chapter-2-defining-nonfunctional-requirements)
* [Chapter 3: Data Models and Query Languages](#chapter-3-data-models-and-query-languages)
* [Chapter 4: Storage and Retrieval](#chapter-4-storage-and-retrieval)
* [Chapter 5: Encoding and Evolution](#chapter-5-encoding-and-evolution)
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
- [ ] Chapter 3: Data Models and Query Languages
- [ ] Chapter 4: Storage and Retrieval
- [ ] Chapter 5: Encoding and Evolution
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

## Chapter 4: Storage and Retrieval

> **Takeaway:** 

## Chapter 5: Encoding and Evolution

> **Takeaway:** 

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
