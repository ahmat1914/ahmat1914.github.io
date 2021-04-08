---
title: Debezium Tutorial
category:
  - Tech
tags:
  - 大数据
mathjax: false
date: 2021-04-08 17:49:36
img:
---

### What is Debezium?

Debezium is a set of distributed services to capture changes in your databases so that your applications can see those changes and respond to them. 

Debezium’s goal is to build up a library of connectors that capture changes from a variety of database management systems and produce events with very similar structures.

Currently have the following connectors:

* MongoDB
* MySQL
* PostgreSQL
* SQL Server
* Oracle (Incubating)
* Db2
* Cassandra (Incubating)
* Vitess (Incubating)

Debezium是一个开源项目，为捕获数据更改(change data capture,CDC)提供了一个低延迟的流式处理平台。
* Debezium为所有的数据库更改事件提供了一个统一的模型，所以你的应用不用担心每一种数据库管理系统的错综复杂性。
* Debezium用持久化的、有副本备份的日志来记录数据库数据变化的历史，因此，你的应用可以随时停止再重启，而不会错过它停止运行时发生的事件，保证了所有的事件都能被正确地、完全地处理掉。

Debezium是一个捕获数据更改(CDC)平台，并且利用Kafka和Kafka Connect实现了自己的持久性、可靠性和容错性。每一个部署在Kafka Connect分布式的、可扩展的、容错性的服务中的connector监控一个上游数据库服务器，捕获所有的数据库更改，然后记录到一个或者多个Kafka topic(通常一个数据库表对应一个kafka topic)。

使用场景：
* 缓存失效(Cache invalidation)
* 简化单体应用(Simplifying monolithic applications)
* 共享数据库(Sharing databases)
* 数据集成(Data integration)


是通过抽取数据库日志来获取变更的。

Debezium is a distributed platform that turns your existing databases into event streams, so applications can see and respond immediately to each row-level change in the databases.

Debezium is a distributed platform that turns your existing databases into event streams, so applications can see and respond immediately to each row-level change in the databases.
* Debezium is built on top of Apache Kafka and provides Kafka Connect compatible connectors that monitor specific database management systems. 
* Debezium records the history of data changes in Kafka logs, from where your application consumes them. 
    * This makes it possible for your application to easily consume all of the events correctly and completely.
    * Even if your application stops unexpectedly, it will not miss anything: when the application restarts, it will resume consuming the events where it left off.



### Run a demo
This tutorial demonstrates how to use Debezium to monitor a MySQL database. 

Will start the Debezium services, run a MySQL database server with a simple example database, and use Debezium to monitor the database for changes.

#### Starting the services

```bash
# zookeeper
docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.4
# Kafka
docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.4
# mysql
docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:1.4
# mysql client
docker run -it --rm --name mysqlterm --link mysql --rm mysql:5.7 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD”'
# Kafka connect
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:1.4

```

After starting the Debezium and MySQL services, you are ready to deploy the Debezium MySQL connector so that it can start monitoring the sample MySQL database (inventory).

#### Registering a connector to monitor the inventory database

By registering the Debezium MySQL connector, the connector will start monitoring the MySQL database server’s binlog. The binlog records all of the database’s transactions (such as changes to individual rows and changes to the schemas). When a row in the database changes, Debezium generates a change event.

1. the configuration of the Debezium MySQL connector that you will register.
(For more information, see [MySQL connector configuration properties](https://debezium.io/documentation/reference/1.5/connectors/mysql.html#mysql-connector-properties) .)

2. Use the curl command to register the Debezium MySQL connector.

```bash
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }’
```

3. Verify that inventory-connector is included in the list of connectors:

```bash
curl -H "Accept:application/json" localhost:8083/connectors/
# ["inventory-connector"]
```

4. Review the connector’s tasks:

```bash
curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector

# HTTP/1.1 200 OK Date: Thu, 06 Feb 2020 22:12:03 GMT Content-Type: application/json Content-Length: 531Server: Jetty(9.4.20.v20190813) { "name": "inventory-connector", ... "tasks": [ { "connector": "inventory-connector", "task": 0 } ] }
```

When you register a connector, it generates a large amount of log output in the Kafka Connect container. By reviewing this output, you can better understand the process that the connector goes through from the time it is created until it begins reading the MySQL server’s binlog.

#### Viewing change events
After deploying the Debezium MySQL connector, it starts monitoring the inventory database for data change events.

Start the `watch-topic` utility to watch the dbserver1.inventory.customers topic from the beginning of the topic.

```bash
docker run -it --rm --name watcher --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:1.4 watch-topic -a -k dbserver1.inventory.customers
```

1. Create event

Here are the details of the key of the last event (formatted for readability):

```bash

```
- [ ] json here 

The event has two parts: a `schema` and a `payload`. 

* The schema contains a Kafka Connect schema describing what is in the payload. 
* The payload has a single id field, with a value of `1004`.

2. Updating the database and viewing the update event

In the terminal that is running the MySQL command line client, run the following statement:

```mysql
mysql> UPDATE customers SET first_name='Anne Marie' WHERE id=1004;

/* Query OK, 1 row affected (0.05 sec) Rows matched: 1 Changed: 1 Warnings: 0

mssql> select * from customers;
```

Switch to the terminal running watch-topic to see a new fifth event.

By changing a record in the customers table, the Debezium MySQL connector generated a new event. 
You should see two new JSON documents: 
* one for the event’s key, 
* and one for the new event’s value.

Here are the details of the key for the update event (formatted for readability):

- [ ] json here 

Here is that new event’s value. There are no changes in the schema section, so only the payload section is shown (formatted for readability):

- [ ] json here 


By viewing the payload section, you can learn several important things about the update event:
* By comparing the before and after structures, you can determine what actually changed in the affected row because of the commit.
* By reviewing the source structure, you can find information about MySQL’s record of the change (providing traceability).
* By comparing the payload section of an event to other events in the same topic (or a different topic), you can determine whether the event occurred before, after, or as part of the same MySQL commit as another event.

3. Deleting a record in the database and viewing the delete event

In the terminal that is running the MySQL command line client, run the following statement:

```mysql
mysql> DELETE FROM customers WHERE id=1004;
```

- [ ] sql results here

By deleting a row in the customers table, the Debezium MySQL connector generated two new events.

Review the key and value for the first new event.
Here are the details of the key for the first new event (formatted for readability):
- [ ] json here

Here is the value of the first new event (formatted for readability):

- [ ] json here

Review the key and value for the second new event.
Here is the key for the second new event (formatted for readability):

- [ ] json here

Here is the value of that same event (formatted for readability):

- [ ] json here

If Kafka is set up to be log compacted, it will remove older messages from the topic if there is at least one message later in the topic with same key. This last event is called a tombstone event, because it has a key and an empty value. This means that Kafka will remove all prior messages with the same key. Even though the prior messages will be removed, the tombstone event means that consumers can still read the topic from the beginning and not miss any events.

4. cleaning up

```bash
docker stop mysqlterm watcher connect mysql kafka zookeeper
```
  
### References
* [Tutorial](https://debezium.io/documentation/reference/1.5/tutorial.html#updating-database-viewing-update-event)
