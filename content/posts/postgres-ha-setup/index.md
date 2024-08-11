---
title: "Postgres Ha Setup"
date: 2024-08-11T17:34:52+09:00
draft: true

description: "Add description for the post here."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["tags"]
categories: ["Documentation"]
theme: "full"
images: ["https://theundersurfers.com/postgres-ha-setup/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/postgres-ha-setup/featured-image.webp"]
---


Simple step by step command to setup up postgres server for HA with pgpool

## Setup
### Streaming Replication

Create primary db:
```
 bin/initdb primary
```
Change in `postgresql.conf`. For now I am jsut settting the listen_address  to `*` but in real scenarios we only allow specific server address to listen at. Also change the port no

```jsx

listen_address = '*' // not safe
port=5433 //  default:5432
```

Create user rep_user with replication flag set
`CREATE USER repuser REPLICATION;`

* add repuser in pg_hba.conf as ipv connections
`host    all             repuser             127.0.0.1/32            trust`

* restart server

* use bg_basebackup comamnds to make replica
`bin/pg_basebackup -h localhost -U  repuser --checkpoint=fast -D ha/replica -R --slot=some_name -C --port=5433`

* new datadir will be crerated ,  open it and configure the port no

* now start the new cluster dir, straming would be runing now.

imran@Imrans-MacBook-Air pg-15-bins % bin/pg_ctl -D ha/replica start  
waiting for server to start....2024-07-18 14:54:09.625 KST [15363] LOG:  starting PostgreSQL 15.3 on aarch64-apple-darwin23.5.0, compiled by Apple clang version 15.0.0 (clang-1500.3.9.4), 64-bit
2024-07-18 14:54:09.626 KST [15363] LOG:  listening on IPv6 address "::", port 5434
2024-07-18 14:54:09.626 KST [15363] LOG:  listening on IPv4 address "0.0.0.0", port 5434
2024-07-18 14:54:09.627 KST [15363] LOG:  listening on Unix socket "/tmp/.s.PGSQL.5434"
2024-07-18 14:54:09.630 KST [15366] LOG:  database system was shut down in recovery at 2024-07-18 14:53:56 KST
2024-07-18 14:54:09.630 KST [15366] LOG:  entering standby mode
2024-07-18 14:54:09.632 KST [15366] LOG:  redo starts at 0/3000060
2024-07-18 14:54:09.632 KST [15366] LOG:  consistent recovery state reached at 0/3000110
2024-07-18 14:54:09.632 KST [15366] LOG:  invalid record length at 0/3000148: wanted 24, got 0
2024-07-18 14:54:09.632 KST [15363] LOG:  database system is ready to accept read-only connections
2024-07-18 14:54:09.639 KST [15367] LOG:  started streaming WAL from primary at 0/3000000 on timeline 1
 done
server started
imran@Imrans-MacBook-Air pg-15-bins % 
```

## Setting up Pgpool

```jsx
* copy the sample file in etc 'pgpool.conf.sample' as 'pgpool.conf'

`etc % cp pgpool.conf.sample pgpool.conf`

listens_address = '*'

backend_hostname1 = 'localhost'
backend_port1 = 5434
backend_weight1 = 1
backend_data_directory1 = '/Users/imran/Desktop/work/db/server/postgres/pg-master/builds/pg-15-bins/ha/replica'

# change backend connection settings
#pid_file_name = 'pgpool.pid'

# change sr_check user setting
sr_check_user = 'repuser'

# change health check and user settting
health_check_user = 'repuser'
health_check_period = 10

# change pid file location
pid_file_name = 'pgpool.pid'

# change log statements setting

log_statement = on 
                                   # Log all statements
log_per_node_statement = on 

```

## Settings for HA

```jsx
* go to primary db conf
synchronous_commit = 'remote_apply'

synchronous_standby_names = '*' # standby servers that provide sync rep

* we need to setup script for fail_over. jsut make a fail_over.sh script and place it some where

```
#! /bin/sh
failed_node=$1
trigger_file=$2

if [ $failed_node = 1 ];
then exit 0;
fi

touch $trigger_file
exit 0;
```

* now go to the pgpool conf change

failover_command = 'failover.sh %d replica_db/trigger_file.trg'

* go to the replica_db

promote_trigger_file = '/home/ubuntu/Desktop/db/pg15/rep/data2/down.trg' 

# when every primary db fials,  pgpool created the trigger file, replica db keep on
# looking for this file and when get it whrn failover happers

```

## Imp commands

```jsx

postgres=# show pool_nodes;
 node_id | hostname  | port | status | pg_status | lb_weight |  role   | pg_role | select_cnt | load_balance_node | replication_delay | replication_state | replication_sync_state | last_status_change  
---------+-----------+------+--------+-----------+-----------+---------+---------+------------+-------------------+-------------------+-------------------+------------------------+---------------------
 0       | localhost | 5432 | down   | up        | 0.000000  | standby | primary | 0          | false             | 0                 |                   |                        | 2024-04-15 02:52:52
 1       | localhost | 5433 | up     | up        | 1.000000  | standby | standby | 0          | true              | 0                 |                   |                        | 2024-04-15 02:52:52
(2 rows)

```
