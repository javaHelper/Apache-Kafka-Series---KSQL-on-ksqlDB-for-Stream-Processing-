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
