---
title: "Install Apache AGE on your Machine"
date: 2022-12-05T18:45:25+05:00
draft: false

description: "Guide to install Apache AGE on you machine from source."

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["apache", "graphs"]
categories: ["Database", "Documentation"]
theme: "full"
images:
  ["https://theundersurfers.netlify.app/age-installation/featured-image.webp"]
seo:
  images:
    ["https://theundersurfers.netlify.app/age-installation/featured-image.webp"]
---

<!--more-->

## Install AGE & PSQL from Source

You can convert your existing relational database to a graph database by simply adding an extension.
Apache AGE is a PostgreSQL extension that provides graph database functionality. It enables users to leverage a graph database on top of the existing relational databases.

We will take a look at

- _Installing Postgres from source._
- _Installing & configuring Apache AGE with Postgres._
- _Installing AGE-Viewer for Graphs analysis._

This article will explain to you how to install age on your Linux machine from the source. For macOS users, there is a detailed video for the overall installation process. Linux users can also watch this video as most of the steps are identical.

{{< youtube 0-qMwpDh0CA >}}

### Getting Started

We gonna install the age and PostgreSQL from the source and configure it.

```bash
mkdir age_installation
cd age_installation
mkdir pg
cd pg
```

#### Dependencies

Install the following essential libraries before the installation. Dependencies may differ based on each OS You can refer to this for further details [https://age.apache.org/age-manual/master/intro/setup.html#pre-installation](https://age.apache.org/age-manual/master/intro/setup.html#pre-installation).

So in case, you have ubuntu installed

```bash
sudo apt-get install build-essential libreadline-dev zlib1g-dev flex bison
```

It is also recommended to install `postgresql-server-dev-xx` package. These are development files for PostgreSQL server-side programming. For example if you are using ubuntu.

```
sudo apt install postgresql-server-dev-11
```

Further you can refer to [https://age.apache.org/age-manual/master/intro/setup.html#postgres-11](https://age.apache.org/age-manual/master/intro/setup.html#postgres-11)

## Installing From source

### PostgreSQL

#### Downloading

You will need to install an AGE-compatible version of Postgres, for now AGE only supports Postgres 11 and 12. Download the source package for postgresql-11.8 [https://www.postgresql.org/ftp/source/v11.18/](https://www.postgresql.org/ftp/source/v11.18/)

Downloading it and extracting it to the `/age_installation/pg`

```bash
wget https://ftp.postgresql.org/pub/source/v11.18/postgresql-11.18.tar.gz && tar -xvf postgresql-11.18.tar.gz && rm -f postgresql-11.18.tar.gz
```

This will download the tar file for the Linux users and also extract it in the working directory.

#### Installing

Now we will install the pg. The `—prefix` have the path where we need to install the psql. In this case, we are installing it in the current directory `pwd` .

```bash

cd postgresql-11.18

# configure by setting flags
./configure --enable-debug --enable-cassert --prefix=$(pwd) CFLAGS="-ggdb -Og -fno-omit-frame-pointer"

# now install
make install

# go back
cd ../../

```

We should configure it by setting the debugging flags. In case of the macOS users

```bash
./configure --enable-debug --enable-cassert --prefix=$(pwd) CFLAGS="-glldb -ggdb -Og -g3 -fno-omit-frame-pointer"
```

### AGE

#### Downloading

Now download the AGE from the GitHub repo [https://github.com/apache/age](https://github.com/apache/age). Clone it under `/age_installation` directory.

```bash
git clone https://github.com/apache/age.git
```

#### Installing

Now we will install the extension AGE on PostgreSQL. First checkout to the PG11 branch.

```bash

cd age/
git checkout PG11

# install
sudo make PG_CONFIG=/home/imran/age_installation/pg/postgresql-11.18/bin/pg_config install

# install check
make PG_CONFIG=/home/imran/age_installation/pg/postgresql-11.18/bin/pg_config installcheck

```

`PG_CONFIG` require the path to the `pg_config` file. Give the path from the recently installed PostgreSQL.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Install%20AGE%20on%20you%20Machine%20d9843f0f39df418e80ca089691cec847_Untitled_1670247501531..png?alt=media&token=9f32b65b-6133-40aa-9784-364ddf4a52c1)

## DB Initialization

Now we will initialize the database cluster using the `initdb`

```bash
cd postgresql-11.18/

# intitialization
bin/initdb demo
```

So we named our database cluster `demo`.

### Start Server

Now start the server. We will make a simple database named `demodb`, however a database named `postgres` is also created by default.

```bash
bin/pg_ctl -D demo -l logfile start
bin/createdb demodb
```

If you wanna change the port number use `bin/createdb --port=5430 demodb`

### Start Querying

AGE added to pg successfully. Now we can enter in to `pg_sql` console to start testing.

```bash
bin/psql demodb
```

If you have server running on some other port like in my case it is 5430 use `bin/createdb --port=5430 demodb`

Now when you have initialize a new DB we have to load the AGE extension in order to start using AGE. Also, we need to set `search_path` and other variables if didn’t set them earlier in the `/postgresql.conf` file.

> Note that you can also set these parameter in the `postgresql.conf` file. [See more](#config-file)

```bash
CREATE EXTENSION age;
LOAD 'age';
SET search_path = ag_catalog, "$user", public;
```

Now try some queries having cypher commands.

```bash
SELECT create_graph('demo_graph');
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "james", bornIn : "US"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ MATCH (v) RETURN v $$) as (v agtype);
```

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Install%20AGE%20on%20you%20Machine%20d9843f0f39df418e80ca089691cec847_Untitled%201_1670247503554..png?alt=media&token=f484b8e0-5f3d-44d6-a21a-b064d951b681)

## Installing AGE-Viewer

So age-viewer is a web app that helps us visualize our data.

### Dependencies

Install `nodejs` and `npm` to run that app. You are recommended to install node `version 14.16.0`. You can install the specific version using the `nvm install 14.16.0`. You can install nvm from [https://www.freecodecamp.org/news/node-version-manager-nvm-install-guide/](https://www.freecodecamp.org/news/node-version-manager-nvm-install-guide/).

```bash
sudo apt install nodejs npm

or

# recommended
nvm install 14.16.0
```

### Downloading

Go back to directory `/age_installation`

```bash
git clone https://github.com/apache/age-viewer.git
```

### Starting

Now set up the environment.

```bash

cd age-viewer

npm run setup
npm run start
```

Viewer will now be running on the localhost. Now connect the viewer with the database server.Enter the details required to login

```bash
# address (default : localhost)
url: server_url;

# port_num (default 5432)
port: port_num;

# username for the database
username: username;

# password for the user
pass: password;

# database name you wanna connect to
dbname: demodb;
```

**Note** that If you are following the above steps and installed postgres form source, then postgres a user would be created automatically during the installation. The `username` will be same as the name of your linux user. In my case a user with name `imran` was created as my linux user name was `imran`. Also there would be no password on this default user, you can set the password later.

In my case I used the following settings.

```bash
url: localhost;

port: 5432;

username: imran;

pass: 1234;

dbname: demodb;
```

Here I am just giving a radom pass for now, as I haven't set any password yet for the user `imran` yet, also auth mode is set to trust so no error would be thrown on a invalid pass.


![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Install%20AGE%20on%20you%20Machine%20d9843f0f39df418e80ca089691cec847_Untitled%202_1670247505361..png?alt=media&token=8704df66-450b-418d-8a42-369b90784a24)

### Visualizing Graphs

Now try creating some nodes and then you can visualize their relationship on the age-viewer.

First we will create `PERSON` nodes

```bash
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "imran", bornIn : "Pakistan"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "ali", bornIn : "Pakistan"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "usama", bornIn : "Pakistan"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "akabr", bornIn : "Pakistan"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "james", bornIn : "US"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Person {name : "david", bornIn : "US"}) $$) AS (a agtype);
```

Now creates some nodes of `COUNTRY`.

```bash
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Country{name : "Pakistan"}) $$) AS (a agtype);
SELECT * FROM cypher('demo_graph', $$ CREATE (n:Country{name : "US"}) $$) AS (a agtype);
```

Now create a relationship between them.

```bash
SELECT * FROM cypher('demo_graph', $$ MATCH (a:Person), (b:Country) WHERE a.bornIn = b.name CREATE (a)-[r:BORNIN]->(b) RETURN r $$) as (r agtype);
```

Now we can visualize our whole graph created.

```bash
SELECT * from cypher('demo_graph', $$ MATCH (a:Person)-[r]-(b:Country) WHERE a.bornIn = b.name RETURN a, r, b $$) as (a agtype, r agtype, b agtype);
```

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Install%20AGE%20on%20you%20Machine%20d9843f0f39df418e80ca089691cec847_Untitled%203_1670247506758..png?alt=media&token=8eb9a33c-a6de-4312-9f60-462d94f1303c)

## Help

### Config File

In case you want to change the port number (default5432) and other variables you can take a look at the`demo/postgresql.conf` file. You have to restart the server after that. Setting the parameters here will save you from the initialization commands like `LOAD 'age';` as used earlier.

```python
nano demo/postgresql.conf

# In postgresql.conf file update and uncomment

port = 5432  #new port if wanted
shared_preload_libraris = 'age'
search_path = 'ag_catalog, "$user", public'
```

You can also set these variables using the commands shown earlier.

### Error with WSL

When installing age-viewer. If you are using WSL restart it from PowerShell after installing `node, npm` using `wsl --shutdown`. WSL path variable can intermingle with node path variable installed on windows. [https://stackoverflow.com/questions/67938486/after-installing-npm-on-wsl-ubuntu-20-04-i-get-the-message-usr-bin-env-bash](https://stackoverflow.com/questions/67938486/after-installing-npm-on-wsl-ubuntu-20-04-i-get-the-message-usr-bin-env-bash)

Now start WSL again. Don’t forget to start the pg server again after the WSL restart. Run server again using `bin/pg_ctl -D demo -l logfile start` .

## Acknowledgment

These resources helped me a lot during the installation. You can also refer to these if help is needed.

- [https://age.apache.org/age-manual/master/intro/setup.html](https://age.apache.org/age-manual/master/intro/setup.html)
- [https://www.postgresql.org/docs/current/install-procedure.html](https://www.postgresql.org/docs/current/install-procedure.html)
- Join the discord for further help [https://discord.com/invite/NMsBs9X8Ss](https://discord.com/invite/NMsBs9X8Ss)
