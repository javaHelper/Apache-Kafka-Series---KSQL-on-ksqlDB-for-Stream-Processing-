# Apache-Kafka-Series---KSQL-on-ksqlDB-for-Stream-Processing-

- References:

- https://simon-aubury.medium.com/using-ksql-apache-kafka-a-raspberry-pi-and-a-software-defined-radio-to-find-the-plane-that-wakes-14f6f9e74584
- https://www.rittmanmead.com/blog/2017/11/taking-ksql-for-a-spin-using-real-time-device-data/
- https://simon-aubury.medium.com/machine-learning-kafka-ksql-stream-processing-bug-me-when-ive-left-the-heater-on-bd47540cd1e8
- https://courses.datacumulus.com/downloads/kafka-ksql-na2/


```sh
user@Prateeks-MacBook-Pro ~ % ksql
Java HotSpot(TM) 64-Bit Server VM warning: Option UseConcMarkSweepGC was deprecated in version 9.0 and will likely be removed in a future release.
                  
                  ===========================================
                  =       _              _ ____  ____       =
                  =      | | _____  __ _| |  _ \| __ )      =
                  =      | |/ / __|/ _` | | | | |  _ \      =
                  =      |   <\__ \ (_| | | |_| | |_) |     =
                  =      |_|\_\___/\__, |_|____/|____/      =
                  =                   |_|                   =
                  =        The Database purpose-built       =
                  =        for stream processing apps       =
                  ===========================================

Copyright 2017-2021 Confluent Inc.

CLI v7.0.1, Server v7.0.1 located at http://localhost:8088
Server Status: RUNNING

Having trouble? Type 'help' (case-insensitive) for a rundown of how things work!

ksql> list topics;

 Kafka Topic                 | Partitions | Partition Replicas 
---------------------------------------------------------------
 default_ksql_processing_log | 1          | 1                  
---------------------------------------------------------------
ksql> 
```
- Then create topic

```sh
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic USERS    
Created topic USERS.
```

```sh
ksql> list topics;

 Kafka Topic                 | Partitions | Partition Replicas 
---------------------------------------------------------------
 USERS                       | 1          | 1                  
 default_ksql_processing_log | 1          | 1                  
---------------------------------------------------------------
```

- Send some data through producer 

```sh
kafka-console-producer --bootstrap-server localhost:9092 --topic USERS
>Alice,US
>
```

Then on ksql screen. Note - By default ksql only shows newly arriving data.

```
ksql> print 'USERS'
>
```

Yoi keep sending data and data should arrive here.

```sh
kafka-console-producer --bootstrap-server localhost:9092 --topic USERS
>Alice,US
>Bob,GB
>Carole,AU
>Dan,PO
>

```

```sh
ksql> print 'USERS';
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2022/09/14 05:29:00.340 Z, key: <null>, value: Bob,GB, partition: 0
rowtime: 2022/09/14 05:29:45.897 Z, key: <null>, value: Carole,AU, partition: 0
rowtime: 2022/09/14 05:29:53.947 Z, key: <null>, value: Dan,PO, partition: 0
```

- Show all data already there in that topic

```sh
ksql> print 'USERS' from beginning;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2022/09/14 05:25:23.746 Z, key: <null>, value: Alice,US, partition: 0
rowtime: 2022/09/14 05:29:00.340 Z, key: <null>, value: Bob,GB, partition: 0
rowtime: 2022/09/14 05:29:45.897 Z, key: <null>, value: Carole,AU, partition: 0
rowtime: 2022/09/14 05:29:53.947 Z, key: <null>, value: Dan,PO, partition: 0
```

- What if I want to see only two data from beginning;

```sh
ksql> print 'USERS' from beginning limit 2;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2022/09/14 05:25:23.746 Z, key: <null>, value: Alice,US, partition: 0
rowtime: 2022/09/14 05:27:27.582 Z, key: <null>, value: Bob,GB, partition: 0
Topic printing ceased
```

```
ksql> print 'USERS' from beginning interval 2 limit 2;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: KAFKA_STRING
rowtime: 2022/09/14 05:25:23.746 Z, key: <null>, value: Alice,US, partition: 0
rowtime: 2022/09/14 05:28:20.351 Z, key: <null>, value: Carole,AU, partition: 0
Topic printing ceased
```
---------

# Steps to create Stream

```
ksql> create stream users_stream (name VARCHAR, countrycode VARCHAR) WITH (KAFKA_TOPIC='USERS', VALUE_FORMAT='DELIMITED');
 Message        
----------------
 Stream created 
----------------
ksql>
```

- List all streams available 

```
ksql> list streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed 
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false    
 USERS_STREAM        | USERS                       | KAFKA      | DELIMITED    | false    
------------------------------------------------------------------------------------------
ksql> 
```

- Now, post some data and you should be able to see data, make sure execute select statement first.

```
kafka-console-producer --bootstrap-server localhost:9092 --topic USERS
>Deepa,AA
>John,SL
>

```

```
ksql> select name, countrycode from USERS_STREAM emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|NAME                                                       |COUNTRYCODE                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|Deepa                                                      |AA                                                         |
|John                                                       |SL                                                         |


```

```
ksql> SET 'auto.offset.reset'='earliest';
Successfully changed local property 'auto.offset.reset' to 'earliest'. Use the UNSET command to revert your change.
ksql>
```

```
ksql> select name, countrycode  from users_stream emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|NAME                                                       |COUNTRYCODE                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|Alice                                                      |US                                                         |
|Bob                                                        |GB                                                         |
|Prateek                                                    |IND                                                        |
|Bob                                                        |GB                                                         |
|Carole                                                     |AU                                                         |
|Dan                                                        |PO                                                         |
|Deepa                                                      |AA                                                         |
|John                                                       |SL                                                         |


```

- Get only first 4 records

```
ksql> select name, countrycode  from users_stream emit changes limit 4;
+-----------------------------------------------------------+-----------------------------------------------------------+
|NAME                                                       |COUNTRYCODE                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|Alice                                                      |US                                                         |
|Bob                                                        |GB                                                         |
|Prateek                                                    |IND                                                        |
|Bob                                                        |GB                                                         |
Limit Reached
Query terminated
```

- Basic Aggregate

```
ksql> select countrycode, count(*) from users_stream group by countrycode emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |KSQL_COL_0                                                 |
+-----------------------------------------------------------+-----------------------------------------------------------+
|US                                                         |1                                                          |
|IND                                                        |1                                                          |
|GB                                                         |2                                                          |
|AU                                                         |1                                                          |
|PO                                                         |1                                                          |
|AA                                                         |1                                                          |
|SL                                                         |1                                                          |

```

- How to delete stream

```
ksql> drop stream if exists users_stream delete topic;

 Message                                           
---------------------------------------------------
 Source `USERS_STREAM` (topic: USERS) was dropped. 
---------------------------------------------------
ksql> list topics;

 Kafka Topic                 | Partitions | Partition Replicas 
---------------------------------------------------------------
 default_ksql_processing_log | 1          | 1                  
---------------------------------------------------------------
ksql>
ksql> show topics;

 Kafka Topic                 | Partitions | Partition Replicas 
---------------------------------------------------------------
 default_ksql_processing_log | 1          | 1                  
---------------------------------------------------------------
ksql>
```
