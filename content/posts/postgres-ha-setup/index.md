---
title: "PostgreSQL HA Setup"
date: 2024-08-11T17:34:52+09:00
draft: false

description: "Setting up postgresql for HA and failover handling using streaming replication and pgpool. Step by step commands to set up your postgresql clusters for high availability."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["underthehood", "setup"]
categories: ["Pgpool"]
theme: "full"
images: ["https://theundersurfers.com/postgres-ha-setup/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/postgres-ha-setup/featured-image.webp"]
---

<!--more-->

Simple step by step command to setup up postgres server for HA with pgpool

### Streaming Replication

Create primary db:
```bash
 bin/initdb primary
```
Change in `postgresql.conf`. For now I am just setting the listen_address  to `*` but in real scenarios we only allow specific server address to listen at. Also change the port no

```conf
listen_address = '*'  # not safe
port=5433             #  default:5432
```

Create user `rep_user` with replication flag set
```sql
CREATE USER repuser REPLICATION;
```
Add repuser in pg_hba.conf as ipv connections
```conf
host    all             repuser             127.0.0.1/32            trust
```

Now restart server. We have to make a replica db now. For than we can use `pg_basebackup`, use the following command to make the secondary server 

```bash
bin/pg_basebackup -h localhost -U  repuser --checkpoint=fast -D ha/replica -R --slot=some_name -C --port=5433
```
New data cluster will be cerated,  open it and configure the port no to be different from the primary server. Set it to `5434` for now.

Now our primary server would be running by now, start the recently created replica sever. It will be automatically be connected to the primary server and will start the streaming replication. You can notice that the streaming replication is been started successfully in the sever logs.
```bash
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

### Setting up Pgpool

Now we can add pgpool over our setup for the connection pooling. Install the pgpool maybe from the rpms availables at there site or may be from the source. Open up the `pgpool.sample` if not available copy it from the sample conf file.
```bash
`etc % cp pgpool.conf.sample pgpool.conf`
```

Now change some configrations for the pgpool.
```bash
listens_address = '*'

# primary node (localhost for now)
backend_hostname1 = 'localhost'
backend_port1 = 5433
backend_weight1 = 0
backend_data_directory1 = '/Users/imran/Desktop/work/db/server/postgres/pg-master/builds/pg-15-bins/ha/primary'

# set replica node confs
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

### Settings for HA

Now we should add some handling for the failover scenarios and improve our HA. First open the `postgersql.conf` for the primary server and change following confs.
```bash
synchronous_commit = 'remote_apply'
synchronous_standby_names = '*' # standby servers that provide sync rep
```

Now we need a simple script that will be executed in case of the fail over. Make a script fail_over.sh and place it somewhere.
```bash
#! /bin/sh
failed_node=$1
trigger_file=$2

if [ $failed_node = 1 ];
then exit 0;
fi

touch $trigger_file
exit 0;
```

Now open the `pgpool.conf` and set the following confs.

```conf
failover_command = 'failover.sh %d replica_db/trigger_file.trg'
```


Open configrations for the replica_db and set this.
```conf
promote_trigger_file = '/home/ubuntu/Desktop/db/pg15/rep/data2/down.trg' 
```

Now the setup is done for handling the failover scenario. Now when ever the primary db fials, the pgpool creates the trigger file. The replica db will keep on looking the for the trigger file, when ever the pgpool creates the trigger file the failover script will be executed.

```bash
postgres=# show pool_nodes;
 node_id | hostname  | port | status | pg_status | lb_weight |  role   | pg_role | select_cnt | load_balance_node | replication_delay | replication_state | replication_sync_state | last_status_change  
---------+-----------+------+--------+-----------+-----------+---------+---------+------------+-------------------+-------------------+-------------------+------------------------+---------------------
 0       | localhost | 5432 | down   | up        | 0.000000  | standby | primary | 0          | false             | 0                 |                   |                        | 2024-04-15 02:52:52
 1       | localhost | 5433 | up     | up        | 1.000000  | standby | standby | 0          | true              | 0                 |                   |                        | 2024-04-15 02:52:52
(2 rows)

```
