---
title: "Database High Availability using PGPOOL-II"
date: 2023-01-03T01:35:57+05:00
draft: false

description: "Increase the throughput of your database system. Add pgpool to database cluster to add connection pooling, Automatic Failover, Load balancing and to make your system highly available"

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["pgpool", "database", "load balancing", "database cluster"]
categories: ["Pgpool", "Database"]
theme: "full"
images:
  ["https://theundersurfers.netlify.app/pgpool-features/featured-image.webp"]
seo:
  images:
    ["https://theundersurfers.netlify.app/pgpool-features/featured-image.webp"]
---

<!--more-->

# Database High Availability using PGPOOL-II

Pgpool-II is an amazing solution to increase your database availability. Pgpool-II is a proxy software that sits between PostgreSQL servers and a PostgreSQL database client.

It reduces the connection overhead and improves the system's overall throughput.

With Pgpool we can create highly available systems that can continue to operate even if a system failure occurs. It also allows scalability.

# Features

Following are some pgpool-II cool features that can improve your DB throughput and make it highly available.

## Load Balancing

### Postgres Streaming Replication

As we scale out to increase the database processing capacity. In Postgress we scale out using **streaming replication or logical replication** but still, postgres cannot distribute the queries by its replication

So in this case pg-pool can be used to distribute the queries between the master and standby servers

### Pgpool for load balancing

Pgpool handles the load balancing like a charm.

- We can assign weights to all servers to decide how much load to send on each server
- We can also ban some queries to be executed by defining the `black_functions_list`

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Database%20High%20Availibilty%20using%20PGPOOL-II%202a8daa71d2b6442ab3c658227ab74c09_Screenshot_from_2023-01-03_00-24-49_1672691150637..png?alt=media&token=1bdffd7d-ca8d-424c-af99-44a208e8c096" caption="Standby node used for Read operations" >}}

## Connection Pooling

- We know making a new connection on every single operation has an overhead of resources.
- So we make a connection pool with pgpool.
- Connection with the same properties is taken from the pool of connections and then returned.
- You can set parameters for the number of pools and connections in pgpool.

Also, **pgbouncer** is a lightweight tool for connection pooling. If you only want connection pooling in your database use pgbouncer instead.

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Database%20High%20Availibilty%20using%20PGPOOL-II%202a8daa71d2b6442ab3c658227ab74c09_with_connection_pool_1672691178219..png?alt=media&token=9c3bad61-8b87-4988-a093-34e0f897cc97" caption="Only 3 connections are opened for the database instead of 5">}}

## Automatic Fail-over

- In case of a server failover, pgpool can help you handle it
- You can write a script of how to handle the fail-over and pgpool will automatically execute the script when a fail-over is detected
- It can handle fail-over by promoting the standby server as a primary server

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Database%20High%20Availibilty%20using%20PGPOOL-II%202a8daa71d2b6442ab3c658227ab74c09_Screenshot_from_2023-01-03_00-34-55_1672691199446..png?alt=media&token=ffdfe7f5-c255-4e8a-aa87-2c801cf2d1ae" caption="Primary crashed, Standby promoted" >}}

## High Availability

So what is high availability? It helps us to make the database always available and work without interruptions, it adds great redundancy to handle errors.

Pgpool is highly available itself. This is done so that pgpool should not become a single point of failure.

- Pgpool makes itself highly available
- It makes itself redundant using the feature called **watchdog**

`watchdog` is a sub-process of pgpool-II aiming for adding high availability feature to it.

# Obstacles

There are still some obstacles in making our cluster highly available.

## Split brains

- It occurs when there is more than one master node.
- It creates data inconsistency

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Database%20High%20Availibilty%20using%20PGPOOL-II%202a8daa71d2b6442ab3c658227ab74c09_1_PWa8dDFLP_mquyIUTLQWBw_1672691212686..png?alt=media&token=2dcc5cd3-66e9-4e81-ba0c-5bd8f280591c" caption=" Two or more nodes become primary nodes, so cannot take a decision" >}}

## Consistency and availability

We cannot ensure both consistency and availability at the same time. We have to prioritize one in our system. CAP theory explains this problem as _“In the event of a network failure on a distributed database, it is possible to provide either consistency or availability—but not both.”_

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Database%20High%20Availibilty%20using%20PGPOOL-II%202a8daa71d2b6442ab3c658227ab74c09_cap-theorem-diagram-800x753-1_1672691218438..webp?alt=media&token=93ed48a4-0542-44d3-bc2f-47ddc15fada5" caption="CAP theory shows we cannot achieve consistency and availability at same time">}}

# Recommended Setup

Recommended Setup for High Availability

- So for replication, we can use Postgres streaming replication. Pgpool also provides replication but It is recommended to use Postgres built-in streaming replication.
- Use Pgpool for: Handling fail-overs, Load balancing, High Availability, and Connection pooling

# References

- Watch the Pgpool tutorial [https://www.youtube.com/watch?v=qpxKlH7DBjU](https://www.youtube.com/watch?v=qpxKlH7DBjU)
- [https://www.ashnik.com/scaling-up-and-load-balancing-your-postgresql-cluster-using-pgpool-ii/](https://www.ashnik.com/scaling-up-and-load-balancing-your-postgresql-cluster-using-pgpool-ii/)
