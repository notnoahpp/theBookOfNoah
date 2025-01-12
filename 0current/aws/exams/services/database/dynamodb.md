# dynamodb

- fully managed nosql (document/key-value) db providing durability high availability and autoscaling
- designed for OLTP with known query patterns

## my thoughts

- getting dynamodb right is all about the designing the data model to lower costs
  - you'll end up with more tables than you would prefer, smaller items than you prefer, and longer partition keys than you prefer
  - colocate hot data to a table thats equally distributed across partitions
    - cold data should be deleted via TTL or moved to s3
  - read the best practices articles, it takes considerable effort to beat the AWS cost structure

## links

- [AAA best practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [accelerator (DAX)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DAX.html)
- [change data capture](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/streamsmain.html)
- [client and server side encryption](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/client-server-side.html)
- [data model partitioning](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.Partitions.html)
- [data protection](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/data-protection.html)
- [direct kms materials provider](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/direct-kms-provider.html)
- [encryption client: how it works](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/how-it-works.html)
- [encryption client: intro](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/what-is-ddb-encrypt.html)
- [error handling](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ErrorHandling.html)
- [pagination](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)
- [query (guide)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html)
- [query (ref)](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html)
- [Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html)
- [streams and lambda triggers](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.html)
- [streams and lambdas (tut)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.Lambda.Tutorial.html)
- [streams](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html)
- [tables](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithTables.html)
- [limits](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Limits.html)
- [working with large attributes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-use-s3-too.html)
- [setting up dynamodb local](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)

### blogs

- [adaptive capacity for uneven data access patterns](https://aws.amazon.com/blogs/database/how-amazon-dynamodb-adaptive-capacity-accommodates-uneven-data-access-patterns-or-why-what-you-know-about-dynamodb-might-be-outdated/)

### api ref

- [api retries](http://docs.aws.amazon.com/general/latest/gr/api-retries.html)
- [batch get item](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_BatchGetItem.html)
- [batch read writem](http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_BatchWriteItem.html)

## best practices

- determine your consistency pattern for each READ request
  - the default is eventually consistent, but you can choose strongly consistency
  - strongly consistency costs more than eventually
- design your data model for requests that are evenly distributed across partitions
  - updating a single attribute in an item requires rewriting the entire item
  - the recommended item size is under 4kb
- read through the architecture section below, then read through it again
- monitoring and troubleshooting: watch your metrics
  - autoscaling takes time to get right, ensure your target utilization matches the spikes in your request patterns
  - always include aws error codes in your cloudwatch logs
  - enable cloudtrail so that control-plane operations (table create/update/etc) are tracked for later analysis
  - cloudwatch is required to monitor table performance
    - set alarms for tracking when specified metrics fall outside acceptable ranges
- global tables
  - provide extremely low latency to global clients
- indexes
  - sparse indexes are key
  - global secondary indexes
    - useful for quick lookups and temporary tables
  - local seoncary indexes
    - use them sparingly
    - choose projections carefully and only those atributes you request frequently
- Managing item expiration with TTL
  - expire old items to keep your storage cost and RCU consumption low (and its free)
    - this is often more cost effective than paying for the WCU to delete an item
  - if implementing this feature ensure your reads check each item's timestamp
- Using streams
  - if implementing item TTL, you can watch the stream data and copy items into cold storage (e.g. s3)
- Using DAX
  - decreases the amount of RCUs required for a table, and smooths out spikey/inbalanced read loads
  - reduces dynamodb's response time from single-digit millisecond to sub-millisecond
  - use cases
    - reduce response times of eventually-consistent read workloads
    - reduce orpational and application complexity througha managed service with the same dynamdob API
    - increase throughput for read-heavy/bursty workloads
- table, item and item attributes
  - focus on matching your data model to dynamodb best practices
    - small item size may mean many tables to help control throughput costs
    - in general avoid large items, and large item attributes at all costs, no matter how much decomposition is required
- design for uniform workloads
  - its all about choosing the correct partition key, which controls how data is distributed across partitions
  - the total throughput provisioned for a table is dividied equally across partitions
    - thus if your only R/W to a single partition, the throughput allocated to remaining partitions are unused
    - to accomodate the weird constraints applied to your data model by dynamodb
      - add a random number / calculated value to each partition key when writing data
  - hot and cold data

### anti patterns

## features

- JSON document / key-value data structures
- event driven programming
- fully managing sharding enables horizontal scaling
- bursts: an adaptive capacity to borrow read throughput from less active keys to more demanding keys
- global tables for fully managed multi-region & multi-master deployments
- auto scaling based on target read/write utilization
- DAX: the dynamodb accelerator: integrated cache with dynamodb compatible api

### pricing

## terms

- consistency: ability to read data understanding that all prior writes will be reflected in the results returned
- scatter gather technique: funny name for a regular-ole thread-safe data chunking algorithm
  - the base thread determines the number of chunks primary key for each chunk
  - threads are created to read/write chunks to/from dynamodb
- read-modify-write: aka optimistic concurrency control; application-level design pattern;
  - whenever you want to update an item but ONLY if it hasnt changed in a particular way since you last read it
    - requires that all processes knows to increment the version number for specific items
  - read: an application retrieves the item and store its version number in memory
    - prepare the update and increment the version number (this will be the new version you'll check against in the future write)
  - write: execute the update with a conditional expression that fails if the version number has changed since you last read it
    - indicates some other process has changed the data your interested in
- back-fill: porting data from one place to another, often used in db migrations to port data between two timestamps

## basics

### architecture

#### instance

- dynamodb replicates (copies) of your data in fault-independent zones within a region
  - replication usually occurs within 1 second
  - availability SLA is four 9s
- request throughput: you specify the requirements and dynamodb will manage the rest ensuring your limits are met
  - read and writes are managed separately
  - updating a single attribute in an item requires rewriting the entire item, so a balance must be met
  - a single item cannot be read greater than 3k RCU, or written at more than 1k WCU
  - eventually consistent reads are levied at half the cost of strongly consistent reads
- streams: similar to kensis streams and compatible with the kinesis client library
  - an ordered flow of information about changes to a table
  - all writes are recorded in the stream like a changelog
    - you can specify the level of detail recorded
  - streams are durable and kept for up to 24 hours
  - shards are created with the stream as the data grows

#### data

- tables: the core data structure
- items: i.e. records in a table
- attributes: i.e. columns in an item
  - primary key: must be unique across all items; either partition key or partition + sort
    - partition key: is always required
    - sort key: if provided, its required in all items and makes the primary key a composite key (partition + sort)
    - FYI:
      - the partition and the sort key (if provided) must be string, number or binary
      - to use maps/lists as part of a primary key, you must expose a copy of the entry directly as an attribute
  - each item can be up to 400kb; the larger the item the higher probability of `hot` activity
- secondary indexes: either local or global; enable queries on attributes other than the tables primary key
  - in general
    - can have 5 local + 5 globals per table
    - allow you to query data based on attributes other than the tables primary key
    - consumption of throughput is based on secondary index for scanning
    - sparse indexes: an attribute used as an secondary index but is only contained in a subset of base table items
      - thus sparsely indexing the base table and optimizes the througput consumption
    - each index requires an additional write, thus incuring additional WCU costs
  - local secondary index: LSI; index must be local to a partiion key;
    - e.g. base table (pkey = name, sortkey = id, attr = date)
    - e.g. LSI (pkey = name, sortkey = date, attr = id)
    - often used for sorting on a different attribute of the base table
    - requirements
      - have a max partition size of 10gb
      - can only be defined when the base table is created, and cannot be deleted
      - must use the same partition key defined in the base table
      - cannot have its own provisioned throughput
  - global secondary index: GSI;
    - generally recommended > LSI unless you need strong consistency
    - temporary indexes that can use totally different partition & sort keys
    - logically its a replication of the base table with an entirely different primary key (partition and/or sort)
    - e.g. take a base table, and define a completely different primary key over the same data
      - then when your done with the GSI, delete it
    - GSI back pressure: when the GSI write throughput is too low and causes throttling to your base table during writes
      - Reads are independent to the base table; this fact can be used to isolate heavy reads to GSI (e.g. for scanning)
    - requirements
      - do not provide strong consistency like LSIs
      - are not subject to the size limitation of LSIs
      - can be created and deleted dynamically unlike LSIs
      - do not require unique primary keys
      - have their own provisioned throughput managed separately from the table
      - only supports eventual consistency
      - only return attributes that are projected into the index

##### data types

- key value model
  - string, number, boolean, binary (base64 encoded), null, and unordered sets of the aforementioned
- json model
  - unordered maps (i.e. object) and unordered lists (i.e. array) of any JSON data type
  - a single item can be a json document
  - or each item in a JSON can be attributes of a json document

### api

- in general
  - batchBLAH is more efficient than multiple non batches
  - deleteBLAH costs the same WCU as creation
  - scan should be avoided unless required, and then effectively filtered as to not read the entire table
    - even when filtered, you are stilled charged for the total amount retrieved before filtering
    - it always scans every item in the table inorder to build the resultset
    - less efficient than scan
  - query can only be used on tables with partitioned composite keys
    - fully indexed and very fast (relative to scan)
  - remember by default reads are eventually consistent, unless strong consistency is requested on each READ request
- writes
  - putItem: upsert single item
  - updateItem: upsert item attributes
  - batchWriteItem: upsert multiple items
  - deleteItem: single item; costs the same amount of WCU as putItem
- reads
  - getItem: single item
  - batchGetItem: multiple items
  - query: retrieve items matching sort key expression for specific partition
  - scan: retrieve items across all partitions in table
- client behavior and configuration
  - error handling:
    - 500 errors can be retried, e.g. Provisioned THroughput exceeded (throttling)
    - 400 errors need to be resolved on the client: e.g. required params missing
    - batch operations
      - operate as loops around get/put/delete items
      - individual requests in the loop which do not complete are returned as such
        - you can implement your own retry loop with exponential-backoff
  - tuning retries
    - set a max number of retries, times and strategies for exponential back-off & jitter

## considerations

### configuration

- request throughput: read and write capacity per second
  - must be specified when you create a table; AWS provisions resources to ensure your settings can be met
    - you set a min, max and a target utilization (in percent)
  - RCU: read capacity unit; 1 RCU = 1 item (4kb/less) per second
  - WCU: write capacity unit; 1 WCU = 1 (1kb/less) write per second
- autoscaling: adjusts provisioned throughput in response to actual traffic patterns
  - is enabled by default and can be configured separately for the table and GSI
  - the target utilization setting is what drives a smooth reaction in autoscaling to match your target request throughput
- Global tables
  - tables can be spread across multiple regions
  - has a higher costs but if enables the SLA becomes 5 nines
  - multi master, and conflicts are resolved by a last-write-wins mechanism
  - doesnt support cross-region strong consistency
  - generally require autoscaling or big enough write capacity to carry all global writes and accommodate the replicated traffic
  - routing mechanisms
    - geo-routing: send global clients to whichever global endpoint is closest
- item TTL
  - expire items after some time

### IAM

- see [the markdown file](../securityIdentityCompliance/iam.md)

### Dynamodb Accelerator (DAX)

- api-compatible cache for dynamodb tables via a separate endpoint
- highly-available cluster accessbile only in a VPC
- write-through cache: items and updates written to cache are eventually consistent on the next read
  - strongly consistent reads are not cached

### backup / restore

- backups: neither type consumes any read/write capacity
  - on demand: created when you request it
  - PITR: point in time recovery
    - 35-day rolling window of recoverable table data down to the second
- restore
  - always made to a new table, after which you can delete the previous one
  - most restores complete in less than 10 hrs
  - partitioned data is restored in parallel

### table design

- partition key
  - should result in an event distribution of item data and traffic across the hash space
    - i.e. pick a partition key and that strongly unique (high cardinality) across all items
      - i.e. pick a key that is unique across the most items, dont pick US.STATE when you can pick US.CITY
  - should reduce the amount of `hot` keys
    - i.e. dont use US.STATE if 90% of your users are in California
    - read the adaptive capacity blog post
- rolling tables: where a new table is created for the current period, e.g. monthly/quarterly/etc
  - older tables will only see traffic when the application specifically requests them
  - enables setting a lower provisioned capacities as tables age and eventualy moved to cold storage/dropped
- attribute size
  - reading/writing large objec ts results in hot activity localized toa single partition
  - prefer to keep item size between 1 and 4kb
    - storing large json blobs in s3 and indexing their location in dynamodb
    - storing large json objects as binary attributes, and keeping the smaller metadata attributes separately
    - decompose json objects into smaller pieces that can be stored as separate items
      - this occurs at the application level using the `scatter-gather` technique
- manage concurrency with teh read-modify-write pattern
- one-to-many tables: e.g.
  - avoid large string/number sets as item attributes, and prefer storing each set element in a separate table as items
  - helps reduce throughput because you wont need to fetch the entire set every time you fetch the item
    - now your base table can contain a reference to the many table
- varied access patterns
  - generally each item in a table should have attributes that are read with similar access patterns
    - if each item has large attributes that dont meet this requirment, split those attributers into a new table
      - e.g. instead of one large USER table, have user.preferenes, and user.blah, and user.bloop
    - its all about controlling item throughput
