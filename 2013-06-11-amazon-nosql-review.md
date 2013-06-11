Amazon NoSQL Solutions
============================================

Amazon provides following [NoSQL](https://en.wikipedia.org/wiki/NoSQL) storage options:

* [SimpleDB] (http://aws.amazon.com/simpledb/) - Amazon SimpleDB is a highly
  available and flexible non-relational data store that offloads the work of
  database administration. Developers simply store and query data items via web
  services requests and Amazon SimpleDB does the rest.
* [DynamoDB] (http://aws.amazon.com/dynamodb/) - Amazon DynamoDB is a
  fully-managed, high performance, NoSQL database service that is easy to set
  up, operate, and scale.


Amazon's review for big data solutions
--------------------------------------------
Amazon's review [Big Data on AWS](http://aws.amazon.com/big-data/) mentions
only DynamoDB and Elastic Map Reduce (based on Hadoop) as tools for big data
management.  As one of major DynamoDB benefits it is highlighted that it uses
solid state drives, but this option is also available for other technologies:

    Solid state, at your service:
    NoSQL data stores benefit greatly from the speed of solid state drives.
    DynamoDB uses them by default, but if you are using alternatives from the AWS
    Marketplace, such as Cassandra or MongoDB, accelerate your access with
    on-demand access to terabytes of solid state storage, with the High I/O
    instance class. Learn more about the options with EC2 instance types (http://aws.amazon.com/ec2/instance-types).

[Elastic Map Reduce] (http://aws.amazon.com/elasticmapreduce/) is a computing
service which can be used in conjunction with storage services to perform
operations on large datasets (indexing, data mining, log file analysis, etc).
Amazon Elastic MapReduce is a web service that enables businesses,
researchers, data analysts, and developers to easily and cost-effectively
process vast amounts of data.  Amazon Elastic Compute Cloud (EC2) - Amazon
Elastic Compute Cloud delivers scalable, pay-as-you-go compute capacity in
the cloud.

[Hadoop] (http://hadoop.apache.org/)
The Apache™ Hadoop® project develops open-source software for reliable,
scalable, distributed computing.  The Apache Hadoop software library is a
framework that allows for the distributed processing of large data sets across
clusters of computers using simple programming models. It is designed to scale
up from single servers to thousands of machines, each offering local
computation and storage. Rather than rely on hardware to deliver
high-availability, the library itself is designed to detect and handle failures
at the application layer, so delivering a highly-available service on top of a
cluster of computers, each of which may be prone to failures.

Q: When would I use Amazon RDS vs. Amazon EC2 Relational Database AMIs vs.
Amazon SimpleDB vs. Amazon DynamoDB? (http://aws.amazon.com/rds/faqs/#4)

    Amazon Web Services provides a number of
    database alternatives for developers. Amazon RDS enables you to run a fully
    featured relational database while offloading database administration; Amazon
    SimpleDB provides simple index and query capabilities with seamless
    scalability; Amazon DynamoDB is a fully managed NoSQL database service that
    offers fast and predictable performance with seamless scalability; and using
    one of our many relational database AMIs on Amazon EC2 and Amazon EBS allows
    you to operate your own relational database in the cloud. There are important
    differences between these alternatives that may make one more appropriate for
    your use case. See Running Databases on AWS for guidance on which solution is
    best for you.

Amazon DynamoDB
============================================

General Info
--------------------------------------------

[Get Started with Amazon DynamoDB](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStartedDynamoDB.html):

    Amazon DynamoDB automatically spreads the data and traffic for the table over a
    sufficient number of servers to handle the request capacity specified by the
    customer and the amount of data stored, while maintaining consistent and fast
    performance. All data items are stored on Solid State Disks (SSDs) and are
    automatically replicated across multiple Availability Zones in a Region to
    provide built-in high availability and data durability.

Read / write throughput limits:

    Provisioned Throughput - When you create or update a table, you specify how
    much provisioned throughput capacity you want to reserve for reads and writes.
    Amazon DynamoDB will reserve the necessary machine resources to meet your
    throughput needs while ensuring consistent, low-latency performance.  If your
    application requirements change, simply update your table throughput capacity
    using the AWS Management Console or the Amazon DynamoDB APIs. You are still
    able to achieve your prior throughput levels while scaling is underway.

    When you create a table, you must provide a table name, its primary key and
    your required read and write throughput values. Except for the required primary
    key, an Amazon DynamoDB table is schema-less. Individual items in an Amazon
    DynamoDB table can have any number of attributes, although there is a limit of
    64 KB on the item size.  A unit of read capacity represents one strongly
    consistent read per second (or two eventually consistent reads per second) for
    items as large as 4 KB. A unit of write capacity represents one write per
    second for items as large as 1 KB.  Reads = Number of item reads per second × 4
    KB item size (If you use eventually consistent reads, you'll get twice as many
    reads per second.) Writes = Number of item writes per second × 1 KB item size.

Amazon DynamoDB supports the following two types of primary keys:

* Hash Primary Key – The primary key is made of one attribute, a hash
  attribute. For example, a ProductCatalog table can have ProductID as its
  primary key. Amazon DynamoDB builds an unordered hash index on this primary
  key attribute.
* Hash and Range Primary Key – The primary key is made of two attributes. The
  first attribute is the hash attribute and the second attribute is the range
  attribute. For example, the forum Thread table can have ForumName and Subject
  as its primary key, where ForumName is the hash attribute and Subject is the
  range attribute. Amazon DynamoDB builds an unordered hash index on the hash
  attribute and a sorted range index on the range attribute.

Local Secondary Indexes:

    When you create a table with a hash-and-range key, you
    can optionally define one or more local secondary indexes on that table. A
    local secondary index lets you query the data in the table using an alternate
    range key, in addition to queries against the primary key.

Amazon DynamoDB Data Types Amazon DynamoDB supports the following data types:

* Scalar data types - Number, String, and Binary.
* Multi-valued types - String Set, Number Set, and Binary Set.

API / SDK:

    Amazon DynamoDB is a web service that uses HTTP and HTTPS as a transport and
    JavaScript Object Notation (JSON) as a message serialization format. Your
    application code can make requests directly to the Amazon DynamoDB web service
  API Reference
  (http://docs.aws.amazon.com/amazondynamodb/latest/APIReference/Welcome.html).
  There is also PHP library as a part of Amazon PHP SDK
  (https://github.com/aws/aws-sdk-php).

DynamoDB as events storage and limitations
--------------------------------------------

For example, we have a table to store events data with following fields:

  - id, primary key, integer
  - user_id, integer
  - event_type_id, integer
  - data, text
  - create_time, timestamp

And we have large amount of events for which we need to perform following operations:

  - filter by date
  - filter by user
  - filter by event type
  - filter by data
  - sorting
  - pagination
  - export

DynamoDB limitations:

* For 'data' filed we can store up to 64KB (DynamoDB limitation). If we need
  to store more data then we need to use compression.
* Filters require additional indexes (not recommended to add many indexes, also
  indexes consume write limits and add a data size limitation).
* Filter by 'data' filed - can be not possible if we will compress data (or we may need to maintain own index)
* Sorting - no, query results are always sorted by the range key
* Pagination - yes (using Limit, LastEvaluatedKey, and ExclusiveStartKey), but
  single request can not return more then 1MB, so if page contains more data
  then less items will be returned.
* Export - no (we can not get more then 1MB via single request)

Additional indexes to perform data filtering (see
http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GuidelinesForLSI.html):
having many indexes is not recommended because they consume storage and
provisioned throughput and make table operations slower.

For tables with local secondary indexes there is size limit of 10GB for data
with the same hash key
(http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html).

    The maximum size of any item collection is 10 GB. This limit does not apply to
    tables without local secondary indexes; only tables that have one or more local
    secondary indexes are affected.
    We recommend as a best practice that you instrument your application to monitor
    the sizes of your item collections. One way to do so is to set the
    ReturnItemCollectionMetrics parameter to SIZE whenever you use BatchWriteItem,
    DeleteItem, PutItem or UpdateItem. Your application should examine the
    ReturnItemCollectionMetrics object in the output and log an error message
    whenever an item collection exceeds a user-defined limit (8 GB, for example).
    Setting a limit that is less than 10 GB would provide an early warning system
    so you know that an item collection is approaching the limit in time to do
    something about it.

    Item collection - is any group of items that have the same hash key, across a
    table and all of its local secondary indexes. For instance, consider an
    e-commerce application that stores customer order data in a DynamoDB table with
    hash-range schema of customer id-order timestamp. Without LSI, to find an
    answer to the question “Display all orders made by Customer X with shipping
    date in the past 30 days, sorted by shipping date”, you had to use the Query
    API to retrieve all the objects under the hash key “X”, sort the results by
    shipment date and then filter out older records.

It is not possible to add secondary indexes into existing table. Existing
indexes also can not be changed or deleted. This complicates future changes.

Query results sorting:

    Query results are always sorted by the range key. If the data type of the range
    key is Number, the results are returned in numeric order; otherwise, the
    results are returned in order of ASCII character code values. By default, the
    sort order is ascending. To reverse the order use the ScanIndexForward
    parameter set to false.

Get records count:

    in a request, set the Count parameter to true if you want
    Amazon DynamoDB to provide the total number of items that match the scan filter
    or query condition, instead of a list of the matching items. In a response,
    Amazon DynamoDB returns a Count value for the number of matching items in a
    request. If the matching items for a scan filter or query condition is over 1
    MB, Count contains a partial count of the total number of items that match the
    request. To get the full count of items that match a request, use the
    LastEvaluatedKey in a subsequent request. Repeat the request until Amazon
    DynamoDB no longer returns a LastEvaluatedKey.

Single operation size limit:

    A single operation can retrieve up to 1 MB of data, which can comprise
    as many as 100 items. BatchGetItem will return a partial result if the response
    size limit is exceeded, the table's provisioned throughput is exceeded, or an
    internal processing failure occurs. If a partial result is returned, the
    operation returns a value for UnprocessedKeys. You can use this value to retry
    the operation starting with the next item to get.  For example, if you ask to
    retrieve 100 items, but each individual item is 50 KB in size, the system
    returns 20 items (1 MB) and an appropriate UnprocessedKeys value so you can get
    the next page of results. If desired, your application can include its own
    logic to assemble the pages of results into one dataset.

Export - data export can be implemented as off-line operation using Elastic Map
Reduce
(http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EMRforDynamoDB.html):

* Exporting data stored in Amazon DynamoDB to Amazon S3.
* Importing data in Amazon S3 to Amazon DynamoDB.
* Querying live Amazon DynamoDB data using SQL-like statements (HiveQL).
* Joining data stored in Amazon DynamoDB and exporting it or querying against the joined data.
* Loading Amazon DynamoDB data into the Hadoop Distributed File System (HDFS) and using it as input into an Amazon EMR job flow.

To perform advanced queries on data it is also possible to copy data to Amazon
Redshift
(http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/RedshiftforDynamoDB.html):

    Amazon Redshift complements Amazon DynamoDB with advanced business intelligence
    capabilities and a powerful SQL-based interface. When you copy data from an
    Amazon DynamoDB table into Amazon Redshift, you can perform complex data
    analysis queries on that data, including joins with other tables in your Amazon
    Redshift cluster.
    In terms of provisioned throughput, a copy operation from an Amazon DynamoDB
    table counts against that table's read capacity. After the data is copied, your
    SQL queries in Amazon Redshift do not affect Amazon DynamoDB in any way. This
    is because your queries act upon a copy of the data from DynamoDB, rather than
    upon DynamoDB itself.

Cost
--------------------------------------------
Dynamo db pricing page (http://aws.amazon.com/dynamodb/pricing/) and price calculator (http://calculator.s3.amazonaws.com/calc5.html#s=DYNAMODB).

Assumptions (having events table described above):

- for each user (user_id in events table) we add 1 million records per day (24 hours)
- each logged event data size is about 256 Bytes
- DynamoDB table does not have local secondary indexes

Parameters for cost calculation:

* Number of events logged per day (24 hours): 1 million records
* Number of write requests (events logged per second): 12427 (for simplicity assume app launches are uniformly distributed across 24 hours)
* Number or read requests: for simplicity assume we have no read requests (read requests are much cheaper then write requests)
* Data size added per month: 30 million record * 0.25KB ~= 7GB

Cost (according to the price calculator) is about $6500 per month (for one user data).

Notes:

* reads are much cheaper then writes, so if we add the same number or reads into calculation it will not grow much.
* we pay not for actual load, but for reserved load. So if we set writes per
  second to 12000 then we will pay for this even if actual load will be low.
  Limits can be changed to higher value at any time, but can be lowered only 4
  times per day.

Other DynamoDB resources
--------------------------------------------
[Amazon Dynamo: The Next Generation Of Virtual Distributed Storage](http://readwrite.com/2007/10/30/amazon_dynamo)

[Amazon DynamoDB – a Fast and Scalable NoSQL Database Service Designed for Internet Scale Applications](http://www.allthingsdistributed.com/2012/01/amazon-dynamodb.html)

[Why My Team Went with DynamoDB Over MongoDB] (http://slashdot.org/topic/bi/why-my-team-went-with-dynamodb-over-mongodb/)
- limitations and possible solutions
- custom index (maybe at that time secondary indexes where not present).
- data as json + compression (other way is to store data as file to S3)

[DynamoDB shortcomings (and our work arounds)](https://www.dailycred.com/blog/10/dynamodb-shortcomings-and-our-work-arounds)
- they decided not to use DynamoDB for events (SQL was a better tool in this
  case, so we decided not to use DynamoDB at all for storing events) and build
  cache between DynamoDB and their app.

[DYNAMODB IS AWESOME, BUT…](http://simondlr.com/post/26360955465/dynamodb-is-awesome-but) - about limitations

[Amazon DynamoDB](http://blog.coredumped.org/2012/01/amazon-dynamodb.html) - about provisioned throughput, Query, Scan and indexing

[Expanding the Cloud: Faster, More Flexible Queries with DynamoDB](http://www.allthingsdistributed.com/2013/04/dynamdb-local-secondary-indices.html) - about secondary indexes

[My Disappointments with Amazon DynamoDB] (http://whynosql.com/my-disappointments-with-amazon-dynamodb/)

[Amazon DynamoDB Part III: MapReducin’ Logs] (http://www.newvem.com/amazon-dynamodb-part-iii-mapreducin-logs/)

Amazon forum threads:

* https://forums.aws.amazon.com/message.jspa?messageID=333470
* https://forums.aws.amazon.com/thread.jspa?messageID=454521
* https://forums.aws.amazon.com/thread.jspa?messageID=449080
* https://forums.aws.amazon.com/thread.jspa?messageID=449927
* https://forums.aws.amazon.com/message.jspa?messageID=361520
* https://forums.aws.amazon.com/thread.jspa?messageID=312101

DynamoDB mocks:

* https://forums.aws.amazon.com/message.jspa?messageID=329773
* https://github.com/perrystreetsoftware/clientside_aws
* https://github.com/ananthakumaran/fake_dynamo
* https://github.com/mboudreau/Alternator

Amazon SimpleDB
============================================

General Info
--------------------------------------------
[Simple DB description page](http://aws.amazon.com/simpledb/).

API/SDK: There is PHP SDK (https://github.com/aws/aws-sdk-php) and HTTP API
(http://docs.aws.amazon.com/AmazonSimpleDB/2009-04-15/DeveloperGuide/SDB_API.html).

[Developer Guide](http://docs.aws.amazon.com/AmazonSimpleDB/2009-04-15/DeveloperGuide/Welcome.html).

**[Complex Queries](http://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/UsingSelect.html)**

    One of the main uses for Amazon SimpleDB involves making complex queries
    against your data set, so you can get exactly the data you need. For more
    information, refer to the Select section of the Amazon SimpleDB Developer
    Guide.

Select operator supports:

- where (comparison operators, including like), sort and limit.
- count() (but query should be no longer then 5 seconds otherwise you will get partial result and need to repeat count() operation)
- max / min values can be selected using ordering by value and limit 1.

**Data Storage and Performance**

    For information on how quickly stored data is recorded to Amazon SimpleDB,
    refer to the Consistency section of the Amazon SimpleDB Developer Guide.

**[Limits and Restrictions](http://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/SDBLimits.html)**

    During development, it is important to understand Amazon SimpleDB's limits
    when storing data, the amount of data Amazon SimpleDB can return from a
    query, and what to do if the limits are exceeded. For more information, refer
    to the Limits section of the Amazon SimpleDB Developer Guide.

Limits:

  * Currently, you can store up to 10 GB per domain and you can create up to 250 domains.
  * Maximum 1 billion attributes per domain (probably this is summary number of attributes for all items).
  * Attribute value length - 1024 bytes.
  * Maximum items in select response - 2500.
  * Maximum query execution time - 5 seconds.
  * Maximum response size for Select - 1MB.
  * Requests per second (did not found in SimpleDB docs, but it is in the DynamoDB FAQ) - 25 writes per second (per domain?)

    Designed for use with other Amazon Web Services—Amazon SimpleDB is designed to integrate easily with other web-scale services such as Amazon EC2 and Amazon S3.
    For example, developers can run their applications in Amazon EC2 and store their data objects in Amazon S3. Amazon SimpleDB can then be used to query the object metadata from within the application in Amazon EC2 and return pointers to the objects stored in Amazon S3.

[Partition data to domains](http://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/DataSetPartitioningConcepts.html).

SimpleDB as events storage and limitations
--------------------------------------------

A table to store events data has following fields:

  - id, primary key, integer
  - user_id, integer
  - event_type_id, integer
  - data, text
  - create_time, timestamp

There is large amount of data and we want to perform following operations:

  - filter by date
  - filter by user
  - filter by event type
  - filter by data
  - sorting
  - pagination
  - export

Limitations:

* Now data size is not limited (text field) and it will be limited (total item size has a limit of 1KB).
* Filters - select operation allows us to implement all filters
* Sorting - select operation allows us to sort data
* Pagination - yes, but single request can not return more then 1MB, so if page contains more data
  then less items will be returned.
* Export - no (we can not get more then 1MB via single request)

Cost
--------------------------------------------

Cost calculator for SimpleDB (http://calculator.s3.amazonaws.com/calc5.html#s=SIMPLEDB).

If we have the same assumtions as for DynamoDB (see above) then we have:

  * Number of Items - 30000000 (per month)
  * Average Number of Attributes Per Item - 5
  * Total Size of Attribute Values - 7GB
  * Number of BatchPuts - 30000000
  * Number of Gets - 30000000
  * Number of Simple Selects - 30000000
  * Data Transfer Out: - 7GB
  * Data Transfer In: - 7GB

Calculated cost: $173.

Amazon SimpleDB vs DynamoDB
============================================

Q: How does Amazon DynamoDB differ from Amazon SimpleDB? Which should I use?
(http://aws.amazon.com/dynamodb/faqs/#How_does_Amazon_DynamoDB_differ_from_Amazon_SimpleDB_Which_should_I_use)

    Both services are non-relational databases that remove the work of database
    administration. Amazon DynamoDB focuses on providing seamless scalability and
    fast, predictable performance. It runs on solid state disks (SSDs) for
    low-latency response times, and there are no limits on the request capacity or
    storage size for a given table. This is because Amazon DynamoDB automatically
    partitions your data and workload over a sufficient number of servers to meet
    the scale requirements you provide.
    In contrast, a table in Amazon SimpleDB has a strict storage limitation of 10
    GB and is limited in the request capacity it can achieve (typically under 25
    writes/second); it is up to you to manage the partitioning and re-partitioning
    of your data over additional SimpleDB tables if you need additional scale.
    While SimpleDB has scaling limitations, it may be a good fit for smaller
    workloads that require query flexibility. Amazon SimpleDB automatically indexes
    all item attributes and thus supports query flexibility at the cost of
    performance and scale.

Note: in the DynamoDB there is also 10GB limitation for item collections (items
with the same hash key) if we use secondary indexes.

For events storage we have:

* query /filter data with different criteria: can be done with SimpleDB, much more complex with DynamoDB
* sort data: yes for SimpleDB, no for DynamoDB (fixed sorting)
* export data: complex for both, need to introduce offline operation (this is good to do even with MySQL)
* response time / requests performane: much better with DynamoDB
* table size limit and scaling: better with DynamoDB
* cost: much higher with DynamoDB

Additional resources
--------------------------------------------

[Quora: What is the difference between SimpleDB and DynamoDB?](http://www.quora.com/Amazon-DynamoDB/What-is-the-difference-between-SimpleDB-and-DynamoDB)

[Stackoverflow: Amazon SimpleDB vs Amazon DynamoDB](http://stackoverflow.com/questions/8961333/amazon-simpledb-vs-amazon-dynamodb)

[Stackoverflow: Amazon SimpleDB or DynamoDB](http://stackoverflow.com/questions/9746702/amazon-simpledb-or-dynamodb)


General Resourses
============================================
[Overview of Big Data and NoSQL Technologies as of January 2013](http://www.syoncloud.com/big_data_technology_overview)

[What The Heck Are You Actually Using NoSQL For?](http://highscalability.com/blog/2010/12/6/what-the-heck-are-you-actually-using-nosql-for.html)

[Scaling Twitter: Making Twitter 10000 Percent Faster](http://highscalability.com/scaling-twitter-making-twitter-10000-percent-faster)

[Stackoverflow: How to store 7.3 billion rows of market data (optimized to be read)?](http://stackoverflow.com/questions/9815234/how-to-store-7-3-billion-rows-of-market-data-optimized-to-be-read)

[Running Databases on AWS](http://aws.amazon.com/running_databases/)

[Running NoSQL Databases on AWS](http://aws.amazon.com/nosql/)

[Anti-RDBMS: A list of distributed key-value stores](http://www.metabrew.com/article/anti-rdbms-a-list-of-distributed-key-value-stores)

[CouchDB: Why NoSQL?](http://www.couchbase.com/why-nosql/nosql-database)

[Thoughts on SimpleDB, DynamoDB and Cassandra](http://perfcap.blogspot.com/2012/01/thoughts-on-simpledb-dynamodb-and.html)

Some Other NoSQL solutions
--------------------------------------------
[MongoDB](http://www.mongodb.org/)

  [MongoDB use cases: Storing log data](http://docs.mongodb.org/manual/use-cases/storing-log-data/)
  [MongoDB NoSQL Database on AWS](http://d36cz9buwru1tt.cloudfront.net/AWS_NoSQL_MongoDB.pdf)

[Cassandra](http://cassandra.apache.org/)

  [Cassandra use cases](http://wiki.apache.org/cassandra/UseCases)

[Hypertable](http://hypertable.com/why_hypertable/)

[HBase](http://hbase.apache.org/)

[CouchDB](http://couchdb.apache.org/)

  Syncing online and offline data. This is a niche CouchDB has targeted.

[VoltDB](http://voltdb.com/)

  [Is VoltDB really as scalable as they claim?](http://www.mysqlperformanceblog.com/2011/02/28/is-voltdb-really-as-scalable-as-they-claim/)

  [VoltDB Decapitates Six SQL Urban Myths And Delivers Internet Scale OLTP In The Process](http://highscalability.com/blog/2010/6/28/voltdb-decapitates-six-sql-urban-myths-and-delivers-internet.html)
