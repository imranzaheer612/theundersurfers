---
title: "Postgres Internals Working"
date: 2023-12-09T23:38:04+05:00
draft: true

description: "Summary and notes for the first 3 chapters of the book pg-internals. You will learn about postgres internal structus, designs, memory, process managment and more"

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["postgres", "internals", "database", "database design", "scans", "cost-estimation", "comopiler"]
categories: ["Database", "Postgres"]
theme: "full"
images: ["https://theundersurfers.com/pg-insternals/featured-image.webp"]
seo:
  images: ["https://theundersurfers.com/pg-insternals/featured-image.webp"]
---

<!--more-->

# The Internals of PostgreSQL

In this blog I will try to summarize the first three chapters of the book  **[The Internals of PostgreSQL](https://www.interdb.jp/pg/).**

I have just created to the point notes for each chapter and will try to explain the concepts related the postures architectures and working in just simple words

## Chapter 1
### Structures & Designs 
This chapters mostly covers the structures and designs in PostgreSQL Database Clusters, Databases,  and Tables

1. **Logical structure of db cluster**
    1. *Database cluster —> Databases —> table, indexes, views*

    Database clusters includes databases which further includes the table, indexes and views. You can see the similar structure with the directories too.

2. **Physical structure of db cluster**
    1. Database cluster : There is a `PGDATA`  directory created.
    2. Databases : Under Data directory there are sub dirs for databases.
    3. Tables and indexes: Under Each database dirs there are files for table, indexes, etc.
    4. tablespaces

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_Screenshot_from_2022-12-02_16-04-15_1702145876594..png?alt=media&token=3069f7bf-9615-460a-8780-7aaecba00393" >}}

Directories under `base/`  are the databases dirs

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_Screenshot_from_2022-12-02_16-05-30_1702145878025..png?alt=media&token=e7c11f3c-57f1-441f-aa06-87c9c1f6e423" >}}

They are structured as
```
`base/db_dirs/tables_files` these are table, index, views files

`base/` —>  Is a `PGDATA or Database Cluster`, contain databases

`base/123` —> single database under `PGDATA`

`bsae/123/123` —> table, index files

tables , indexes layout `base/16384/18740`
```

1. **Internal layout of Heap Table File**
    1. It consist of pages or blocks, slotted page
2. **Writing and reading heap tuples**
    1. Writing tuples `pd_lower, pd_upper, line_pointer` , all these pointer changes as new tuple added
    2. Reading Heap Tuples
        1. Sequential Scan : reading all line pointer sequentially
        2. B-tree index Scan :  
            
            Index file containing `index tuples (index_key, TID→target_heaptuple)`
            
            Searching in index file as using B-tree for the key. This will point to the target heap tuple
            
            {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-1-06_1702145879431..png?alt=media&token=4965e3aa-2128-45c0-a648-c1de638ae92e" >}}
            

## Chapter 2 / Process and Memory Architecture

### 1. Processes

1. Postgres Server Process (Postmaster)
    1. parent of all the processs in the postgress server.
    2. Listen to the new connection, make background processs and assign to the clients
    3. Starts by using `pg_ctl` 
    4. This process listen on port 5432 (default)
2. Backend Processes
    1. start by postgress server process
    2. handles all queries by one clients, one backend process for each client
    3. `max_connection` parameter to set max clients for pgserver (default 100)
3. Background Process
    1. `background writer, checkpointer, autovacuum launcher, WAL writer, statistics collector, logging collector (logger), archiver`
    
    ### 2. Memory Architecture
    
    Classified to 2 categories
    
    1. Local Memory Area: 
        1. Only used by a  backend process for its local use.
        2. Each backend process have one of it.
    2. Shared Memory Area:
        1. Used by pgServer on startup.
        2. Used by all process
        3. Sub Areas: `WAL buffer, commit log, shared buffer pool`

## Chapter 3 / Query Processing

### Overview

#### Parser

- Checks the Syntax & return syntax error.
- Generates the Parse Tree from a given SQL statement.
- Root node is `SelectStmt` Defined in `parsenodes.h`
- Doesn’t check the semantics.
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-3-02_1702145880830..png?alt=media&token=62d841c3-8f65-419f-b59e-e517a8f5501c" >}}
    

#### Analyzer

- Runs Semantic analysis on parse Tree
- Generate a Query Tree.
- Root of the tree is Query Stmt. Defined in `parsenode.h` [postgres/src/include/nodes/parsenodes.h](https://github.com/postgres/postgres/tree/master/src/include/nodes)
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-3-03_1702145882480..png?alt=media&token=767eee24-a6d1-45f7-b8a3-f4167bb82a96" >}}
    
    Main lists in the Query are: 
    
    - `targetlist` : List of columns that are the result. i.e. *id'*
     and *‘data*
    - `*rtable` : List of tables that will be used in the query
    - `*jointree` : stores FROM & WHERE clause.
    - `*sortClause` :  List of SortGroupClause.

#### Re writer

- Go through the rules systems and transform the query tree according to the rules if necessary.
- Rules stored in `pg_rules`
- `Views` is psql is an example of rewriter. When ever Views are created the corresponding commands are saved on the catalog
- Following figures shows hhow the rewritter updates the query tree when it containsa a reference to a `View`.
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-3-04_1702145883943..png?alt=media&token=0fa9e120-fcfe-4136-8b59-cfa7d1e63c13" >}}
    

#### Planner

- Gets Query Tree from rewriter.
- Generates query plan tree that will be processed by the executor.
- `EXPLAIN` command displays the `plan tree`.

```sql
testdb=# EXPLAIN SELECT * FROM tbl_a WHERE id < 300 ORDER BY data;
                              QUERY PLAN                           
    ---------------------------------------------------------------
     Sort  (cost=182.34..183.09 rows=300 width=8)
       Sort Key: data
       ->  Seq Scan on tbl_a  (cost=0.00..170.00 rows=300 width=8)
             Filter: (id < 300)
    (4 rows)
```

- Root of tree is `PlannedStmt`. And tree is composed of `plan nodes` defined `plannodes.h`.

#### Executor

- `Plannodes` contains info about for the executor.
- Uses `buffer manager` for reading & writing tables, indexes, etc. from database cluster.
- Executor uses mem areas like `temp_buffers and work_mem,` which are the memory areas in the backend process.
- While accessing tuples(rows) is uses concurrency controls mechanisms.
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-3-06_1702145885610..png?alt=media&token=047e76b7-0e46-4498-a51d-57b0747dc9d8" >}}
    

## Cost Estimation in Single-Table Query

Query Optimization is based on cost. There are three kinds of cost:

- `start-up` cost for first tuples.
- `run` cost to fetch all tuples
- `total` sum of both costs.

As we know the `EXPLAIN` command gives us info about the `plan tree`.  It gives us info about the sequential scan, index scan and sort operation and there cost. 

```sql
testdb=# EXPLAIN SELECT * FROM tbl;
                           QUERY PLAN                        
    ---------------------------------------------------------
     Seq Scan on tbl  (cost=0.00..145.00 rows=10000 width=8)
    (1 row)
```

Here it is showing that the start-up and total costs are 0.00 and 145.00, respectively. Now we will see how we can estimate the cost for these scans i.e sequential or index scan, sort

## Scans

How a sequential scan on a table happens??

How sequentail scan on a tables heaps,  blocks, pages?

How a index scan happens?

B-Tree scan?

Index only scan don’t need to scan from the heap after its the `ct_id` is conformed by the index.

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_Screenshot_from_2023-01-06_01-04-38_1702145886794..png?alt=media&token=7b11c952-0dce-4c54-9047-d7dcec031740" >}}

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_Screenshot_from_2023-01-06_01-04-00_1702145888684..png?alt=media&token=5bc7b9fb-a97b-4a9c-9d68-72b87cadc75c" >}}

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_Screenshot_from_2023-01-06_01-08-17_1702145890580..png?alt=media&token=51e1dc7f-e2ea-4267-9ddd-cbb522e4767f" >}}

### Sequential Scan:

The Seq Scan operation scans the entire relation (table) as stored on disk.

- Estimated by `cost_seqscan()`
- **In seq scan `start-up` cost is 0.**
- **`run cost`** can be calculated bu following function
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_equation_1702145892591..png?alt=media&token=a78aadc6-f982-42ea-b110-a057e3bd1c8f" >}}
    
- Default values are
    - `seq_page_cost` = 1.0
    - `cpu_tuple_cost` = 0.01
    - `cpu_operator_cost` = 0.0025
    - `N_tuples` = num of tuples
    - `N_pages` = all pages of the table
- `total_cost  = start-up + run cost`,  start-up will be zero in this case.

More details can be found [here](https://www.interdb.jp/pg/pgsql03.html#_3.2.1.).

### Index Scan

he Index Scan performs a B-tree traversal, walks through the leaf nodes to find all matching entries, and fetches the corresponding table data.

- Estimated by `cost_index()`
- **[Start-Up Cost](https://www.interdb.jp/pg/pgsql03.html#_3.2.2.1.):**
    - Cost to access the first tupple in target table.
        
        {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_startup-cost_1702145893663..png?alt=media&token=070966c9-3ed2-40ae-a4e2-eb9cdc15f6f4" >}}
        
- **[Run Cost](https://www.interdb.jp/pg/pgsql03.html#_3.2.2.2.):**
    - The cost of fetching all the tuples.
        
        {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_run-cost_1702145894622..png?alt=media&token=ece73877-a5d5-43c3-a463-cbe438505907" >}}
        
- **Total Cost**
    - `total_cost  = start-up + run cost`

For further explanation read the detailed article [here](https://www.interdb.jp/pg/pgsql03.html#_3.2.2.).

## Sort

Sort is used when using ORDER BY, the p reprocessing of merge join operations and other functions. 

- **Startup Cost :** is the cost of sorting the target tuples, therefore, the cost is O(Nsort×log2(Nsort)), where Nsort is the number of the tuples to be sorted.
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_sort-startup-cost_1702145895595..png?alt=media&token=fd329e5c-2b20-4dca-865a-66c6923982de" >}}
    
- **Run Cost:** The run cost of the sort path is the cost of reading the sorted tuples, therefore the cost is O(Nsort).
    
    {{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_sort-runcost_1702145896560..png?alt=media&token=06d5bae1-a43c-43f8-8377-fdc8f52b49c8" >}}
    
- **Total Cost:** Its the sum of both

More details can be found [here](https://www.interdb.jp/pg/pgsql03.html#_3.2.3.).

## Creating Plan Tree of Single Table Query

For understaing the plan tree we will start by discussing the single table query. The planner in PostgreSQL performs three steps, as shown below:

1. Carry out preprocessing.
2. Get the cheapest access path by estimating the costs of all possible access paths.
3. Create the plan tree from the cheapest path.

### Per-processing

Before creating a plan tree, the planner carries out some preprocessing of the query tree.

- Simplification target lists, limit clauses, and so on. i.e. ‘2 + 2’ is rewritten to ‘4’
- Normalizing Boolean expressions. i.e. ‘NOT (NOT a)’ is rewritten to ‘a’.
- Flattening AND/OR expressions. Nested AND and OR operation are flattened.

{{< image src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/pginternals-img%2Fpginternals-img_fig-3-09_1702145897498..png?alt=media&token=d585566e-97e3-419b-bb4c-38ae07c606a1" >}}

### Getting the Cheapest Access Path

Planner estimates the costs of all possible access paths and choices the cheapest one.

- `RelOptInfo` structure to store the access paths and the corresponding costs.
- Estimate the **costs of all possible access paths**, and add the access paths to the `RelOptInfo` structure. Try estimating the cost if the sequential scan,  index scan and bitmap scan etc and add these acess path info to the `RelOptInfo` structure.
- **Get the cheapest access path** in the pathlist of the `RelOptInfo` structure.
- Estimating LIMIT, ORDER BY and ARREGISFDD costs if necessary.


## How the Executor Performs

In single-table queries, the executor takes the plan nodes in an order from the end of the plan tree to the root and then invokes the functions that perform the processing of the corresponding nodes. The best way to understand how the executor performs is to read the output of the EXPLAIN command because PostgreSQL's EXPLAIN shows the plan tree almost as it is.

```bash
testdb=# EXPLAIN SELECT * FROM tbl_1 WHERE id < 300 ORDER BY data;
                              QUERY PLAN                           
    ---------------------------------------------------------------
     Sort  (cost=182.34..183.09 rows=300 width=8)
       Sort Key: data
       ->  Seq Scan on tbl_1  (cost=0.00..170.00 rows=300 width=8)
             Filter: (id < 300)
    (4 rows)
```

The executor uses `work_men` and `temp_buffers`, for query processing, it uses temporary files if processing cannot be performed within only the memory.