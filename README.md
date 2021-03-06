# A MongoDB-based store for Quartz.

This is a Cassandra and MongoDB-backed job store for the [Quartz scheduler](http://quartz-scheduler.org/).

## Maven Artifacts

This is a fork from: https://github.com/michaelklishin/quartz-mongodb.git
Inovaworks have not published this artifact to any repository. Please build it first:


## Usage

Like most things in Quartz, this job store is configured
via a property file, `quartz.properties`:

	# Using MongoDB

    # Use the MongoDB store
    org.quartz.jobStore.class=com.novemberain.quartz.mongodb.MongoDBJobStore
    # MongoDB URI (optional if 'org.quartz.jobStore.addresses' is set)
    org.quartz.jobStore.mongoUri=mongodb://localhost:27020
    # comma separated list of mongodb hosts/replica set seeds (optional if 'org.quartz.jobStore.mongoUri' is set)
    org.quartz.jobStore.addresses=host1,host2
    # database name
    org.quartz.jobStore.dbName=quartz
    # Will be used to create collections like mycol_jobs, mycol_triggers, mycol_calendars, mycol_locks
    org.quartz.jobStore.collectionPrefix=mycol
    # thread count setting is ignored by the MongoDB store but Quartz requries it
    org.quartz.threadPool.threadCount=1
    

    # Using Cassandra
    
    # Use the Cassandra store
    org.quartz.jobStore.class=com.inovaworkscc.quartz.cassandra.CassandraJobStore
    # Cassandra Contact Point
    org.quartz.jobStore.contactPoint=localhost
    # Cassandra Port
    org.quartz.jobStore.port=9042
    # Keyspace name
    org.quartz.jobStore.dbName=quartz_nosql
    # thread count setting is ignored by the Cassandra store but Quartz requries it
    org.quartz.threadPool.threadCount=1
    
The DDL is available at
    src/main/resources/cassandra_ddl 



### Error Handling in Clustered Mode

When running in clustered mode, the store will periodically check in
with the cluster. Should that operation fail, the store needs to
decide what to do:

 * Shut down
 * Do nothing and optimistically proceed

Different strategies make sense in different scenarios. Pausing Quartz would
be optimal but this job store currently doesn't have that option.

The `org.quartz.jobStore.checkInErrorHandler.class` property controls the error handler
implementation.

To shut down the JVM (which is the default), add the following key to `quartz.properties`

    org.quartz.jobStore.checkInErrorHandler.class=com.novemberain.quartz.mongodb.cluster.KamikazeErrorHandler
    org.quartz.jobStore.checkInErrorHandler.class=com.inovaworkscc.quartz.cassandra.cluster.KamikazeErrorHandler

to ignore the failure:

    org.quartz.jobStore.checkInErrorHandler.class=com.novemberain.quartz.mongodb.cluster.NoOpErrorHandler
    org.quartz.jobStore.checkInErrorHandler.class=com.inovaworkscc.quartz.cassandra.cluster.NoOpErrorHandler




### Clojure and Quartzite

If you use [Quartzite](http://clojurequartz.info) or want your job classes to be available
to Clojure code, use:

    org.quartz.jobStore.class=com.novemberain.quartz.mongodb.DynamicMongoDBJobStore
    org.quartz.jobStore.class=com.inovaworkscc.quartz.cassandra.DynamicCAssandraJobStore

(this assumes Clojure jar is on classpath).

### Job Data storage
By default you are allowed to pass any `java.io.Serializable` objects inside `JobDataMap`.
It will be serialized and stored as a `base64` string.

If your `JobDataMap` only contains simple types, it may be stored directly inside MongoDB or Cassandra to save some performance.

    org.quartz.jobStore.jobDataAsBase64=false

## Clustering

To enable clustering set the following property:

    # turn clustering on:
    org.quartz.jobStore.isClustered=true

    # Must be unique for each node or AUTO to use autogenerated:
    org.quartz.scheduler.instanceId=AUTO
    # org.quartz.scheduler.instanceId=node1

    # The same cluster name on each node:
    org.quartz.scheduler.instanceName=clusterName

Each node in a cluster must have the same properties, except *instanceId*.
To setup other clusters use different collection prefix:

    org.quartz.scheduler.collectionPrefix=yourCluster

Different time settings for cluster operations:

    # Frequency (in milliseconds) at which this instance checks-in to cluster.
    # Affects the rate of detecting failed instances.
    # Defaults to 7500 ms.
    org.quartz.jobStore.clusterCheckinInterval=10000

    # Time in millis after which a trigger can be considered as expired.
    # Defaults to 10 minutes:
    org.quartz.jobStore.triggerTimeoutMillis=1200000

    # Time in millis after which a job can be considered as expired.
    # Defaults to 10 minutes:
    org.quartz.jobStore.jobTimeoutMillis=1200000

    # Time limit in millis after which a trigger should be treated as misfired.
    # Defaults to 5000 ms.
    org.quartz.jobStore.misfireThreshold=10000

    # WriteConcern timeout in millis when writing in Replica Set.
    # Defaults to 5000 ms.
    org.quartz.jobStore.mongoOptionWriteConcernTimeoutMillis=10000

## Continuous Integration

[![Build Status](https://secure.travis-ci.org/michaelklishin/quartz-mongodb.png?branch=master)](http://travis-ci.org/michaelklishin/quartz-mongodb)

CI is hosted by [Travis CI](http://travis-ci.org/)


## Copyright & License

(c) Michael S. Klishin, Alex Petrov, 2011-2018.

[Apache Public License 2.0](http://www.apache.org/licenses/LICENSE-2.0.html)


## FAQ

### Project Origins

The project was originally started by MuleSoft. It supports all Quartz trigger types and
tries to be as feature complete as possible.

### Why the Fork?

MuleSoft developers did not respond to attempts to submit pull
requests for several months. As more and more functionality was added
and implementation code refactored, I decided to completely separate
this fork form GitHub forks network because the project is now too
different from the original one. All changes were made with respect to
the Apache Public License 2.0.
