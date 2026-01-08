---
title: "Setting Up a PostgreSQL BDR Cluster for Multi-Master Replication: A Step-by-Step Guide"
date: 2025-01-13T18:25:47+05:00
draft: false

description: "Learn how to set up an EDB BDR (Bi-Directional Replication) cluster on a single host machine using multiple instances with different socket directories. This guide focuses on EPAS (EnterpriseDB Postgres Advanced Server) and provides step-by-step instructions for installing required packages and configuring the cluster."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["setup", "compilation"]
categories: ["Documentation"]
theme: "full"
images: ["https://theundersurfers.com/pg-bdr/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/pg-bdr/featured-image.webp"]
---

<!--more-->

Master the setup of a PostgreSQL BDR (Bi-Directional Replication) cluster for Multi-Master Replication. This guide walks you through installing EPAS (EnterpriseDB Postgres Advanced Server), configuring multiple instances, and achieving a robust distributed database system with high availability.

## Install EPAS

Make sure to install the edb postgres (epas) server before installing the bdr. You can also install bdr with postgres but for this setup I am setting it up for the epas. Use this link to install epas on ubuntu. https://www.enterprisedb.com/docs/epas/latest/installing/

Note that in order to access the edb bdr and epass you should have the access to the edb repository and only then you would be able to install the packages. Take a look  https://www.enterprisedb.com/docs/repos/getting_started/with_cli/adding_edb_repositories/ and add the edb repo.

## Setting up PGD

Also take a look at the edb distributed postgres setup documentation for a detailed look . https://www.enterprisedb.com/docs/pgd/latest/deploy-config/deploy-manual/deploying/04-installing-software/

## Install Packages

Install the bdr packages. You may install the proxies but we will not setting up the proxies in this setup.

```bash
sudo dnf -y install edb-bdr5-epas17 edb-pgd5-proxy edb-pgd5-cli
sudo systemctl status edb-as-17
```

## Setting up clusters

Lets create some bash var that will be helpful in setting up the clusters.

```bash

PGBIN="/usr/lib/edb-as/17/bin"
PGCLUSTER="$/home/ubuntu/Desktop/work/edb-clusters"
PGDATA1_PORT=5445
PGDATA2_PORT=5446
PGDATA3_PORT=5447
PGPARENT_SOCKETDIR=$/var/run

```

Bellow is a script for setting up three data clusters and then setting up the bdr extension.

### Node 1

```bash

-- node 1 ----------------------------------------------------------

$PGBIN/initdb -D $PGCLUSTER/data1 -E UTF-8

echo -e "port = '$PGDATA1_PORT'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
echo -e "unix_socket_directories = '$PGPARENT_SOCKETDIR/edb-pgd1'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
sudo mkdir $PGPARENT_SOCKETDIR/edb-pgd1/
sudo chmod 777 $PGPARENT_SOCKETDIR/edb-pgd1/

$PGBIN/pg_ctl -D $PGCLUSTER/data1 start -l log1

echo -e "shared_preload_libraries = '\$libdir/bdr'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
echo -e "wal_level = 'logical'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
echo -e "track_commit_timestamp = 'on'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
echo -e "max_worker_processes = '16'" | tee -a $PGCLUSTER/data1/postgresql.conf >/dev/null
	$PGBIN/psql edb -c "ALTER USER ubuntu WITH PASSWORD 'edb'" -p $PGDATA1_PORT -h $PGPARENT_SOCKETDIR/edb-pgd1
	echo -e "host all all all md5\nhost replication all all md5" | sudo tee -a $PGCLUSTER/data1/pg_hba.conf >/dev/null
	echo -e "*:*:*:ubuntu:edb" | sudo tee /var/lib/pgsql/.pgpass; sudo chmod 0600 /var/lib/pgsql/.pgpass

$PGBIN/pg_ctl -D $PGCLUSTER/data1 restart -l log1

$PGBIN/psql postgres -c "CREATE DATABASE bdrdb" -p $PGDATA1_PORT -h $PGPARENT_SOCKETDIR/edb-pgd1
$PGBIN/psql bdrdb -c "CREATE EXTENSION bdr CASCADE" -p $PGDATA1_PORT -h $PGPARENT_SOCKETDIR/edb-pgd1
$PGBIN/psql bdrdb -p $PGDATA1_PORT -h $PGPARENT_SOCKETDIR/edb-pgd1

```

### Node 2

```bash

-- node 2 ----------------------------------------------------------

$PGBIN/initdb -D $PGCLUSTER/data2 -E UTF-8

echo -e "port = '$PGDATA2_PORT'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
echo -e "unix_socket_directories = '$PGPARENT_SOCKETDIR/edb-pgd2'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
sudo mkdir $PGPARENT_SOCKETDIR/edb-pgd2/
sudo chmod 777 $PGPARENT_SOCKETDIR/edb-pgd2/

$PGBIN/pg_ctl -D $PGCLUSTER/data2 start -l log2

echo -e "shared_preload_libraries = '\$libdir/bdr'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
echo -e "wal_level = 'logical'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
echo -e "track_commit_timestamp = 'on'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
echo -e "max_worker_processes = '16'" | tee -a $PGCLUSTER/data2/postgresql.conf >/dev/null
$PGBIN/psql edb -c "ALTER USER ubuntu WITH PASSWORD 'edb'" -p $PGDATA2_PORT -h $PGPARENT_SOCKETDIR/edb-pgd2
echo -e "host all all all md5\nhost replication all all md5" | sudo tee -a $PGCLUSTER/data2/pg_hba.conf >/dev/null
echo -e "*:*:*:ubuntu:edb" | sudo tee /var/lib/pgsql/.pgpass; sudo chmod 0600 /var/lib/pgsql/.pgpass

$PGBIN/pg_ctl -D $PGCLUSTER/data2 restart -l log2

$PGBIN/psql postgres -c "CREATE DATABASE bdrdb" -p $PGDATA2_PORT -h $PGPARENT_SOCKETDIR/edb-pgd2
$PGBIN/psql bdrdb -c "CREATE EXTENSION bdr CASCADE" -p $PGDATA2_PORT -h $PGPARENT_SOCKETDIR/edb-pgd2
$PGBIN/psql bdrdb -p $PGDATA2_PORT -h $PGPARENT_SOCKETDIR/edb-pgd2
```

### Node 3

```bash
-- node 3 ----------------------------------------------------------

$PGBIN/initdb -D $PGCLUSTER/data3 -E UTF-8

echo -e "port = '$PGDATA3_PORT'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null
echo -e "unix_socket_directories = '$PGPARENT_SOCKETDIR/edb-pgd3'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null
sudo mkdir $PGPARENT_SOCKETDIR/edb-pgd3/
sudo chmod 777 $PGPARENT_SOCKETDIR/edb-pgd3/

$PGBIN/pg_ctl -D $PGCLUSTER/data3 start -l log3

echo -e "shared_preload_libraries = '\$libdir/bdr'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null
echo -e "wal_level = 'logical'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null
echo -e "track_commit_timestamp = 'on'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null
echo -e "max_worker_processes = '16'" | tee -a $PGCLUSTER/data3/postgresql.conf >/dev/null

$PGBIN/psql edb -c "ALTER USER ubuntu WITH PASSWORD 'edb'" -p $PGDATA3_PORT -h $PGPARENT_SOCKETDIR/edb-pgd3

echo -e "host all all all md5\nhost replication all all md5" | sudo tee -a $PGCLUSTER/data3/pg_hba.conf >/dev/null
echo -e "*:*:*:ubuntu:edb" | sudo tee /var/lib/pgsql/.pgpass; sudo chmod 0600 /var/lib/pgsql/.pgpass

$PGBIN/pg_ctl -D $PGCLUSTER/data3 restart -l log3

$PGBIN/psql postgres -c "CREATE DATABASE bdrdb" -p $PGDATA3_PORT -h $PGPARENT_SOCKETDIR/edb-pgd3
$PGBIN/psql bdrdb -c "CREATE EXTENSION bdr CASCADE" -p $PGDATA3_PORT -h $PGPARENT_SOCKETDIR/edb-pgd3
$PGBIN/psql bdrdb -p $PGDATA3_PORT -h $PGPARENT_SOCKETDIR/edb-pgd3
```

## Make a Distributed Cluster

Now we need to run some commands on seperate nodes. First, lets run the command on the first node.

```sql

-- 1st node
select bdr.create_node('node-one','host=localhost dbname=bdrdb port=5445');
select bdr.create_node_group('pgd');
select bdr.create_node_group('dc1','pgd');

```

Run this on the second data node

```sql

-- 2nd node
select bdr.create_node('node-two','host=localhost dbname=bdrdb port=5446');
select bdr.join_node_group('host=localhost dbname=bdrdb port=5445','dc1');

```

Run this on third data node

```sql

-- 3rd node
select bdr.create_node('node-three','host=localhost dbname=bdrdb port=5447');
select bdr.join_node_group('host=localhost dbname=bdrdb port=5445','dc1');

```

Now let add some data on any node and check if the data is bieng replicated

```sql

-- testing the group
CREATE TABLE quicktest ( id SERIAL PRIMARY KEY, value INT ); 
INSERT INTO quicktest (value) SELECT random()*10000 FROM generate_series(1,10000);

```

You may check the status by using these utilit;

```sql
select * from bdr.node_replication_rates;
select COUNT(*),SUM(value) from quicktest;

bdrdb=# SELECT *  FROM bdr.node_group_summary WHERE parent_group_name IS NOT NULL;
-[ RECORD 1 ]-----------+-----
node_group_name         | dc1
default_repset          | dc1
parent_group_name       | pgd
node_group_type         | data
apply_delay             | 
check_constraints       | 
num_writers             | 
enable_wal_decoder      | 
streaming_mode          | 
default_commit_scope    | 
location                | 
enable_proxy_routing    | t
enable_raft             | t
route_writer_max_lag    | -1
route_reader_max_lag    | -1
route_writer_wait_flush | f
analytics_storage       | 

```

## References

- https://www.enterprisedb.com/docs/epas/latest/installing/
- https://www.enterprisedb.com/docs/repos/getting_started/with_cli/adding_edb_repositories/
- https://www.enterprisedb.com/docs/pgd/latest/deploy-config/deploy-manual/deploying/04-installing-software/