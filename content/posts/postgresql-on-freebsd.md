---
title: "PostgreSQL on FreeBSD"
date: 2020-09-19T22:00:00Z
weight: 1
aliases: ["/postgresql-on-freebsd"]
tags: ["PostgreSQL", "FreeBSD"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1601815246/jjba-site/blog/postgresql-install-linux/postgresql2_k06kid.png"
    alt: "PostgreSQL"
    caption: ""
    relative: false
    hidden: false
comments: true
description: "How to set up a proper database environment running on our beloved FreeBSD."
disableHLJS: false
---

PostgreSQL or Postgres is a powerful object-relational high-performance database management system (ORDBMS) published under a flexible BSD-style license. PostgreSQL is well suited for large databases and has many advanced features.

In this tutorial, I will show you how to install and configure a PostgreSQL database server on FreeBSD. We will install the latest version of PostgreSQL 11 on the FreeBSD 12.0 system.

## Prerequisites

For this guide, we will use FreeBSD 12 with 2 GB of RAM memory and 2 CPUs. If you have a large deployment, you will need more than that. You will also need the root privileges for package installation.

## Update and Upgrade Packages

Firstly, we will update the packages repositories and upgrade all packages to the latest version using the `pkg` package management tool for FreeBSD.

```sh
pkg update
pkg upgrade
```

## Install PostgreSQL 11

In this step, we're going to install the latest stable version PostgreSQL 11. By default, the FreeBSD repository provides multiple versions of PostgreSQL package. You can use the following command to check all available version of PostgreSQL packages.

```sh
pkg search postgresql
```

You will get multiple versions of PostgreSQL database server. 

Now install the PostgreSQL 11 package using the command below.

```sh
pkg install postgresql11-server postgresql11-client
```

Next, we need to add the PostgreSQL service to the system boot and initialize the database before starting the service. Add the PostgreSQL to the system boot using the command below.

```sh
sysrc postgresql_enable=yes
```

Now initialize the PostgreSQL database, and then start the service, using the following commands

```sh
/usr/local/etc/rc.d/postgresql initdb
service postgresql start
service postgresql status
```

Verify the output of the last command gives a similar output:

```sh
root@bsdstation:/home/joe # service postgresql status
pg_ctl: server is running (PID: 1122)
/usr/local/bin/postgres "-D" "/var/db/postgres/data11"
```

The PostgreSQL service is up and running on FreeBSD 12.0. Additionally you can check the system port used by the PostgreSQL service using the sockstat command below. You should get the port `5432` is used by the PostgreSQL service.

```sh
sockstat -l4 -P tcp
```

## Configure PostgreSQL Authentication

In this step, we're going to set up the authentication method for PostgreSQL. PostgreSQL supports different authentication methods such as trust authentication (default), password-based authentication, Kerberos, GSSAPI, LDAP, RADIUS, and PAM. For this guide, we're going to set up the password-based authentication using MD5. Go to the `/var/db/postgresql/data11` directory, edit the `pg_hba.conf` file using a text editor.

```sh
cd /var/db/postgres/data11
vi pg_hba.conf
```

Now change the authentication method for all local connection to `md5` as below, and add a line to allow external access to the database if needed.

```sh
# TYPE  DATABASE        USER            ADDRESS                 METHOD
    
# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# External connections
host    all             all             192.168.66.1/32         md5
```

Save and close the file, and then restart the PostgreSQL service, enabling the password-based authentication using `md5` for the PostgreSQL server.

```sh
service postgresql restart
```

## Setup New User and Database

In this step, we're going to set up a new user and database on PostgreSQL. We're going to create a new password for default user `postgres`, and create a new user and database. Log in to the 'postgres' user using the command below.

```sh
su - postgres
```

Now login to the interactive PostgreSQL shell `psql`.

```sh
psql
```

Then create a new password for the `postgres` user.

```sh
\password postgres
TYPE THE PASSWORD
```

Next, we will create a new user called `joe` with the database `joe_db`. And the give privileges for the user to the database by running the following queries.

```sql
CREATE DATABASE joe_db;
CREATE USER joe WITH ENCRYPTED PASSWORD 'password123#';
GRANT ALL PRIVILEGES ON DATABASE joe_db TO joe;
```

You can now exit from the PostgreSQL interactive shell with `\q`. As a result, the password for the default `postgres` user has been created. And the new user and database have been set up.

## Testing connection

Log in to the `postgres` user and then run the `psql` command to get into the PostgreSQL interactive shell.

```sh
su - postgres
psql
```

Show list users and database on the PostgreSQL server using the following queries. You will get the new user `joe` and the database `joe_db` on the result.

```sh
\du
\l 
```

Type `\q` to exit from the psql shell. Next, we will log in using the created user `joe` to the database `joe_db` using the command below.

```sh
psql -U joe -d joe_db -W  
```

Type the `joe` password to continue. Now we will proceed to create a new table `user_table` and insert some data into it.

```sql
CREATE TABLE user_table (id INT, name TEXT, site TEXT);
INSERT INTO user_table (id, name, site) VALUES ( 1 , 'Joe the Man', 'jjbigorra.netlify.app');
```

Show content of tables using the following query.

```sql
SELECT * FROM user_table; 
```

Finally, the installation and configuration of PostgreSQL 11 on the FreeBSD 12.0 system has been completed successfully.