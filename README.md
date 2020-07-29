# Query Diagnostics
This is a simple example of reading and writing to an [Apache Cassandra](https://cassandra.apache.org)™ database
cluster while giving a window into the internal routing and execution tracing within
the cluster. It utilizes basic logging, Cassandra query tracing, and events from
the driver's connection to the cluster. The driver events can include
notifications that members of the clusters go down or come back up along with connections
to local and remote data centers.

There are often questions about why the server throws certain exceptions to the
client application. For example, why do I get [`NoNodeAvailableException`](https://docs.datastax.com/en/drivers/java/4.7/com/datastax/oss/driver/api/core/NoNodeAvailableException.html)
errors when I know nodes in my cluster are available? Diagnosis of a fault in a 
distributed system is tricky. The application and driver have limited visibility 
of the status of the network and members of the database cluster. Each exception 
emanates from a limited knowledge of the state of all components in the system. See the
[Byzantine Generals problem](https://en.wikipedia.org/wiki/Byzantine_fault) 
for a general explanation.  However with more information about what is seen at each stage
through tracing and logging, you will hopefully arrive at a diagnosis more efficiently.

Not considered directly for this example are multi-data center connections and data center failover.  However,
the logging in this example shows what connections are made from the driver to nodes in the cluster.  Best practices
for failover, especially in a multi-data center environment, are discussed at length in this
[white paper](https://www.datastax.com/resources/whitepaper/designing-fault-tolerant-applications-datastax-and-apache-cassandratm)
and in this [webinar](https://www.datastax.com/resources/webinar/designing-fault-tolerant-applications-datastax-enterprise-and-apache-cassandra)
about designing fault tolerant applications. There is also an accompanying [demo](https://github.com/datastax/dc-failover-demo)
to show best practices for data center failover with the latest 4.x Java driver.

Contributors: [Jeremy Hanna](http://github.com/jeromatron)

## Objectives

- Understand how queries interact with and within a Cassandra cluster
- Learn about tools to help diagnose problems such as logging, tracing, and those in [Additional Resources](#additional-resources)

## Project Layout

The project has a standard [Apache Maven](https://maven.apache.org) project layout with a single Java class: [QueryDiagnostics](/src/main/java/com/datastax/examples/QueryDiagnostics.java). 

## Setup and Running

### Prerequisites

- [Apache Maven 3](https://maven.apache.org) should be installed and in the path
- JDK 14
- An Apache Cassandra™ cluster is running and accessible through the contacts points and data center identified in [application.conf](/src/main/resources/application.conf).
The program will create a keyspace with `NetworkToplogyStrategy` with replication in `dc1`, the Apache Cassandra default when using the `GossipingPropertyFileSnitch`.

### Running
#### Building

At the project root level, execute

`mvn clean package`

#### Configuration changes
The main configuration file is [`src/main/resources/application.conf`](/src/main/resources/application.conf).
Consider the following changes from defaults for your cluster and environment: 

- Change `basic.contact-points = ["127.0.0.1:9042"]` to the address and cql port of your Cassandra cluster.
- Change `basic.local-datacenter = dc1` to connect to a data center in your Cassandra cluster.
- Change `basic.request.consistency = LOCAL_ONE` to your preferred consistency level. 
- Change `replication = {'class': 'NetworkTopologyStrategy', 'dc1' : 1}` in [`QueryDiagnostics.java`](/src/main/java/com/datastax/examples/QueryDiagnostics.java#L31)
to your preferred replication settings. 

#### Running the program
To execute the program, run the following:

`mvn exec:java -D"exec.mainClass"="com.datastax.examples.QueryDiagnostics"`

## Additional Resources
- Note in this repo that we've enabled debug logging of `com.datastax.oss.driver.internal.core.channel` in the [logback configuration](src/main/resources/logback.xml#L26) to log connection updates with different nodes in the cluster
- [Java driver documentation on query tracing](https://docs.datastax.com/en/developer/java-driver/4.7/manual/core/tracing/)
- [`cqlsh` documentation on query tracing](https://docs.datastax.com/en/cql-oss/3.3/cql/cql_reference/cqlshTracing.html)
- DataStax Studio developer notebooks have [execution configurations that can perform query traces](https://docs.datastax.com/en/studio/6.8/studio/gs/manageRunConfigurations.html)
- [Interactive request tracing](https://www.datastax.com/blog/2012/11/request-tracing-cassandra-12)
- [Probabilistic tracing](https://www.datastax.com/blog/2012/11/advanced-request-tracing-cassandra-12)
- [Packet capture for dynamic tracing](https://cassandra.apache.org/doc/latest/troubleshooting/use_tools.html#packet-capture)
- [Using Wireshark for dynamic cql tracing](http://www.redshots.com/finding-rogue-cassandra-queries/)
- [Replacing Cassandra tracing with Zipkin](https://thelastpickle.com/blog/2015/12/07/using-zipkin-for-full-stack-tracing-including-cassandra.html) (this method is not yet possible with DataStax Enterprise)
