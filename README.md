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
# Our First ksql streams
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
-----

# Create Stream with JSON

```
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic USERPROFILE
Created topic USERPROFILE.
```

```
ksql> CREATE STREAM userprofile (userid INT, firstname VARCHAR, lastname VARCHAR, countrycode VARCHAR, rating DOUBLE) \
>  WITH (VALUE_FORMAT = 'JSON', KAFKA_TOPIC = 'USERPROFILE');

 Message        
----------------
 Stream created 
----------------
ksql> list streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed 
------------------------------------------------------------------------------------------
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false    
 USERPROFILE         | USERPROFILE                 | KAFKA      | JSON         | false    
------------------------------------------------------------------------------------------
ksql> describe USERPROFILE;

Name                 : USERPROFILE
 Field       | Type            
-------------------------------
 USERID      | INTEGER         
 FIRSTNAME   | VARCHAR(STRING) 
 LASTNAME    | VARCHAR(STRING) 
 COUNTRYCODE | VARCHAR(STRING) 
 RATING      | DOUBLE          
-------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
ksql> 
```

```
kafka-console-producer --bootstrap-server localhost:9092 --topic USERPROFILE 
>{"userid": 1000, "firstname":"Alison", "lastname":"Smith", "countrycode":"GB", "rating":4.7}    
>{"userid": 1001, "firstname":"Bob", "lastname":"Smith", "countrycode":"US", "rating":4.2}
```

```
ksql> select firstname, lastname, countrycode, rating from USERPROFILE emit changes;
+----------------------------+----------------------------+----------------------------+----------------------------+
|FIRSTNAME                   |LASTNAME                    |COUNTRYCODE                 |RATING                      |
+----------------------------+----------------------------+----------------------------+----------------------------+
|Alison                      |Smith                       |GB                          |4.7                         |
|Bob                         |Smith                       |US                          |4.2                         |


```
-------

# KSQL Datagen Generating Stream

At UNIX prompt

```
ksql-datagen schema=./datagen/userprofile.avro format=json topic=USERPROFILE key=userid msgRate=1 iterations=1000

(io.confluent.ksql.logging.processing.ProcessingLogConfig:376)
log4j:WARN No appenders could be found for logger (org.apache.kafka.connect.json.JsonConverterConfig).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
['1000'] --> ([ '1000' | 'Grace' | 'Fawcett' | 'GB' | '3.4' ]) ts:1663149692187
['1001'] --> ([ '1001' | 'Ivan' | 'Jones' | 'IN' | '3.4' ]) ts:1663149692206
['1002'] --> ([ '1002' | 'Bob' | 'Edison' | 'GB' | '3.4' ]) ts:1663149693197
['1003'] --> ([ '1003' | 'Ivan' | 'Fawcett' | 'IN' | '4.4' ]) ts:1663149694198
['1004'] --> ([ '1004' | 'Eve' | 'Edison' | 'GB' | '2.2' ]) ts:1663149695189
['1005'] --> ([ '1005' | 'Grace' | 'Jones' | 'AU' | '3.7' ]) ts:1663149696212
```

At KSQL prompt

-- Review a stream - every 5th row
```
print 'USERPROFILE' interval 5;
```

-------

# Manipulate a Stream

```
ksql> describe userprofile;

Name                 : USERPROFILE
 Field       | Type
-----------------------------------------
 USERID      | INTEGER
 FIRSTNAME   | VARCHAR(STRING)
 LASTNAME    | VARCHAR(STRING)
 COUNTRYCODE | VARCHAR(STRING)
 RATING      | DOUBLE


select rowtime, firstname from userprofile emit changes;

ksql> select rowtime, firstname from userprofile emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|ROWTIME                                                    |FIRSTNAME                                                  |
+-----------------------------------------------------------+-----------------------------------------------------------+
|1663137765144                                              |Alison                                                     |
|1663137782250                                              |Bob                                                        |
|1663149692187                                              |Grace                                                      |
|1663149692206                                              |Ivan                                                       |
|1663149693197                                              |Bob                                                        |
|1663149694198                                              |Ivan                                                       |
|1663149695189                                              |Eve                                                        |
|1663149696212                                              |Grace                                                      |
|1663149697197                                              |Eve                                                        |
|1663149698196                                              |Heidi                                                      |

```
Review Scalar functions at https://docs.confluent.io/current/ksql/docs/developer-guide/syntax-reference.html#scalar-functions

```
ksql> select  TIMESTAMPTOSTRING(rowtime, 'dd/MMM HH:mm') as createtime, firstname + ' ' + ucase(lastname)  as full_name
>from userprofile emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|CREATETIME                                                 |FULL_NAME                                                  |
+-----------------------------------------------------------+-----------------------------------------------------------+
|14/Sep 12:12                                               |Alison SMITH                                               |
|14/Sep 12:13                                               |Bob SMITH                                                  |
|14/Sep 15:31                                               |Grace FAWCETT                                              |
|14/Sep 15:31                                               |Ivan JONES                                                 |
|14/Sep 15:31                                               |Bob EDISON                                                 |
|14/Sep 15:31                                               |Ivan FAWCETT                                               |
|14/Sep 15:31                                               |Eve EDISON                                                 |
|14/Sep 15:31                                               |Grace JONES                                                |
|14/Sep 15:31                                               |Eve JONES                                                  |
|14/Sep 15:31                                               |Heidi DOTTY                                                |
|14/Sep 15:31                                               |Dan JONES                                                  |
|14/Sep 15:31                                               |Dan JONES                                                  |
|14/Sep 15:31                                               |Bob COEN                                                   |
|14/Sep 15:31                                               |Grace DOTTY                                                |
|14/Sep 15:31                                               |Ivan JONES                                                 |
```

----------

# Streams from streams and functions

```
select firstname + ' ' 
+ ucase( lastname) 
+ ' from ' + countrycode 
+ ' has a rating of ' + cast(rating as varchar) + ' stars. ' 
+ case when rating < 2.5 then 'Poor'
       when rating between 2.5 and 4.2 then 'Good'
       else 'Excellent' 
   end as description
from userprofile emit changes;


+------------------------------------------------------------------------------------------------------------------------+
|DESCRIPTION                                                                                                             |
+------------------------------------------------------------------------------------------------------------------------+
|Alison SMITH from GB has a rating of 4.7 stars. Excellent                                                               |
|Bob SMITH from US has a rating of 4.2 stars. Good                                                                       |
|Grace FAWCETT from GB has a rating of 3.4 stars. Good                                                                   |
|Ivan JONES from IN has a rating of 3.4 stars. Good                                                                      |
|Bob EDISON from GB has a rating of 3.4 stars. Good                                                                      |
|Ivan FAWCETT from IN has a rating of 4.4 stars. Excellent                                                               |
|Eve EDISON from GB has a rating of 2.2 stars. Poor                                                                      |
|Grace JONES from AU has a rating of 3.7 stars. Good                                                                     |
|Eve JONES from IN has a rating of 2.2 stars. Poor                                                                       |
|Heidi DOTTY from US has a rating of 3.9 stars. Good                                                                     |
|Dan JONES from GB has a rating of 3.4 stars. Good                                                                       |
|Dan JONES from US has a rating of 3.7 stars. Good                                                                       |
|Bob COEN from AU has a rating of 4.9 stars. Excellent                                                                   |
|Grace DOTTY from IN has a rating of 4.4 stars. Excellent                                                                |
|Ivan JONES from IN has a rating of 2.2 stars. Poor                                                                      |
|Eve EDISON from GB has a rating of 3.7 stars. Good                                                                      |
|Heidi JONES from US has a rating of 2.2 stars. Poor                                                                     |
|Alice FAWCETT from IN has a rating of 3.7 stars. Good                                                                   |
|Ivan EDISON from AU has a rating of 3.7 stars. Good                                                                     |
|Grace COEN from IN has a rating of 3.7 stars. Good                                                                      |

```

```
ksql> run script '/Users/prats/Downloads/ksql-course-master/user_profile_pretty.ksql'

 Message                                          
--------------------------------------------------
 Created query with ID CSAS_USER_PROFILE_PRETTY_7 
--------------------------------------------------
ksql> 

ksql> select description from user_profile_pretty emit changes;
+------------------------------------------------------------------------------------------------------------------------+
|DESCRIPTION                                                                                                             |
+------------------------------------------------------------------------------------------------------------------------+
|Alison SMITH from GB has a rating of 4.7 stars. Excellent                                                               |
|Bob SMITH from US has a rating of 4.2 stars. Good                                                                       |
|Grace FAWCETT from GB has a rating of 3.4 stars. Good                                                                   |
|Ivan JONES from IN has a rating of 3.4 stars. Good                                                                      |
|Bob EDISON from GB has a rating of 3.4 stars. Good                                                                      |
|Ivan FAWCETT from IN has a rating of 4.4 stars. Excellent                                                               |
|Eve EDISON from GB has a rating of 2.2 stars. Poor                                                                      |
|Grace JONES from AU has a rating of 3.7 stars. Good                                                                     |
|Eve JONES from IN has a rating of 2.2 stars. Poor                                                                       |


ksql> drop stream user_profile_pretty;

 Message                                                                
------------------------------------------------------------------------
 Source `USER_PROFILE_PRETTY` (topic: USER_PROFILE_PRETTY) was dropped. 
------------------------------------------------------------------------
```
-------

# ksqlDB Tables

```
kafka-topics --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic COUNTRY-CSV
Created topic COUNTRY-CSV.
```

```
ksql-course-master % kafka-console-producer --bootstrap-server localhost:9092 --topic COUNTRY-CSV --property "parse.key=true"  --property "key.separator=:"
>AU:Australia
>IN:India
>GB:UK
>US:United States
>
```

```
ksql> CREATE TABLE COUNTRYTABLE  (countrycode VARCHAR PRIMARY KEY, countryname VARCHAR) WITH (KAFKA_TOPIC='COUNTRY-CSV', VALUE_FORMAT='DELIMITED');

 Message       
---------------
 Table created 
---------------
ksql> show tables;

 Table Name   | Kafka Topic | Key Format | Value Format | Windowed 
-------------------------------------------------------------------
 COUNTRYTABLE | COUNTRY-CSV | KAFKA      | DELIMITED    | false    
-------------------------------------------------------------------
ksql> describe COUNTRYTABLE;

Name                 : COUNTRYTABLE
 Field       | Type                           
----------------------------------------------
 COUNTRYCODE | VARCHAR(STRING)  (primary key) 
 COUNTRYNAME | VARCHAR(STRING)                
----------------------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;


ksql> select countrycode, countryname from countrytable emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|AU                                                         |Australia                                                  |
|IN                                                         |India                                                      |
|GB                                                         |UK                                                         |
|US                                                         |United States                                              |
^CQuery terminated


ksql> select countrycode, countryname from countrytable where countrycode='GB' emit changes limit 1;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|GB                                                         |UK                                                         |
Limit Reached
Query terminated


ksql> select countrycode, countryname from countrytable where countrycode='FR' emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+


```

# Update a table
One record updated (UK->United Kingdom), one record added (FR)

At UNIX prompt

```
kafka-console-producer --broker-list localhost:9092 --topic COUNTRY-CSV --property "parse.key=true"  --property "key.separator=:"
GB:United Kingdom
FR:France
```
At KSQL prompt

```

select countrycode, countryname from countrytable emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|AU                                                         |Australia                                                  |
|IN                                                         |India                                                      |
|US                                                         |United States                                              |
|GB                                                         |United Kingdom                                             |
|FR                                                         |France                                                     |
^CQuery terminated
ksql> select countrycode, countryname from countrytable where countrycode='GB' emit changes limit 1;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|GB                                                         |United Kingdom                                             |
Limit Reached
Query terminated
ksql> select countrycode, countryname from countrytable where countrycode='FR' emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|FR                                                         |France                                                     |
^CQuery terminated
ksql> 
```

------

# ksqlDB and KSQL Intermediate

# KSQL Joins

```sh
ksql-datagen schema=./datagen/userprofile.avro format=json topic=USERPROFILE key=userid msgRate=1 iterations=1000
```

```
ksql> select countrycode, countryname from countrytable emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |COUNTRYNAME                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|AU                                                         |Australia                                                  |
|IN                                                         |India                                                      |
|US                                                         |United States                                              |
|GB                                                         |United Kingdom                                             |
|FR                                                         |France                                                     |
^CQuery terminated


ksql> select firstname,lastname,countrycode,rating from userprofile emit changes;
+----------------------------+----------------------------+----------------------------+----------------------------+
|FIRSTNAME                   |LASTNAME                    |COUNTRYCODE                 |RATING                      |
+----------------------------+----------------------------+----------------------------+----------------------------+
|Bob                         |Jones                       |US                          |3.4                         |
|Carol                       |Jones                       |GB                          |4.4                         |
|Eve                         |Fawcett                     |US                          |3.7                         |
|Carol                       |Smith                       |US                          |4.4                         |
|Carol                       |Fawcett                     |GB                          |4.9                         |
^CQuery terminated

ksql> select up.firstname, up.lastname, up.countrycode, ct.countryname 
>from USERPROFILE up 
>left join COUNTRYTABLE ct on ct.countrycode=up.countrycode emit changes;
+----------------------------+----------------------------+----------------------------+----------------------------+
|FIRSTNAME                   |LASTNAME                    |UP_COUNTRYCODE              |COUNTRYNAME                 |
+----------------------------+----------------------------+----------------------------+----------------------------+
|Alice                       |Edison                      |US                          |United States               |
|Dan                         |Edison                      |AU                          |Australia                   |
|Ivan                        |Jones                       |AU                          |Australia                   |
|Heidi                       |Fawcett                     |US                          |United States               |
|Frank                       |Dotty                       |US                          |United States               |
|Alice                       |Fawcett                     |AU                          |Australia                   |
|Eve                         |Jones                       |US                          |United States               |
|Alice                       |Smith                       |AU                          |Australia                   |
^CQuery terminated

ksql> create stream up_joined as 
>select up.firstname 
>+ ' ' + ucase(up.lastname) 
>+ ' from ' + ct.countryname
>+ ' has a rating of ' + cast(rating as varchar) + ' stars.' as description 
>, up.countrycode
>from USERPROFILE up 
>left join COUNTRYTABLE ct on ct.countrycode=up.countrycode;

 Message                                 
-----------------------------------------
 Created query with ID CSAS_UP_JOINED_13 
-----------------------------------------
ksql> 

ksql> describe up_joined;

Name                 : UP_JOINED
 Field          | Type                   
-----------------------------------------
 UP_COUNTRYCODE | VARCHAR(STRING)  (key) 
 DESCRIPTION    | VARCHAR(STRING)        
-----------------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
ksql> 
^C

ksql> DESCRIBE up_joined EXTENDED;

Name                 : UP_JOINED
Type                 : STREAM
Timestamp field      : Not set - using <ROWTIME>
Key format           : KAFKA
Value format         : JSON
Kafka topic          : UP_JOINED (partitions: 1, replication: 1)
Statement            : CREATE STREAM UP_JOINED WITH (KAFKA_TOPIC='UP_JOINED', PARTITIONS=1, REPLICAS=1) AS SELECT
  (((((((UP.FIRSTNAME + ' ') + UCASE(UP.LASTNAME)) + ' from ') + CT.COUNTRYNAME) + ' has a rating of ') + CAST(UP.RATING AS STRING)) + ' stars.') DESCRIPTION,
  UP.COUNTRYCODE UP_COUNTRYCODE
FROM USERPROFILE UP
LEFT OUTER JOIN COUNTRYTABLE CT ON ((CT.COUNTRYCODE = UP.COUNTRYCODE))
EMIT CHANGES;

 Field          | Type                   
-----------------------------------------
 UP_COUNTRYCODE | VARCHAR(STRING)  (key) 
 DESCRIPTION    | VARCHAR(STRING)        
-----------------------------------------

Queries that write from this STREAM
-----------------------------------
CSAS_UP_JOINED_13 (RUNNING) : CREATE STREAM UP_JOINED WITH (KAFKA_TOPIC='UP_JOINED', PARTITIONS=1, REPLICAS=1) AS SELECT   (((((((UP.FIRSTNAME + ' ') + UCASE(UP.LASTNAME)) + ' from ') + CT.COUNTRYNAME) + ' has a rating of ') + CAST(UP.RATING AS STRING)) + ' stars.') DESCRIPTION,   UP.COUNTRYCODE UP_COUNTRYCODE FROM USERPROFILE UP LEFT OUTER JOIN COUNTRYTABLE CT ON ((CT.COUNTRYCODE = UP.COUNTRYCODE)) EMIT CHANGES;

For query topology and execution plan please run: EXPLAIN <QueryId>

Local runtime statistics
------------------------


(Statistics of the local KSQL server interaction with the Kafka topic UP_JOINED)

Consumer Groups summary:

Consumer Group       : _confluent-ksql-default_query_CSAS_UP_JOINED_13

Kafka topic          : COUNTRY-CSV
Max lag              : 0

 Partition | Start Offset | End Offset | Offset | Lag 
------------------------------------------------------
 0         | 0            | 6          | 6      | 0   
------------------------------------------------------
ksql> 


ksql> select * from up_joined emit changes;
+-----------------------------------------------------------+-----------------------------------------------------------+
|UP_COUNTRYCODE                                             |DESCRIPTION                                                |
+-----------------------------------------------------------+-----------------------------------------------------------+
|US                                                         |Carol JONES from United States has a rating of 2.2 stars.  |
|AU                                                         |Alice JONES from Australia has a rating of 3.4 stars.      |
|AU                                                         |Frank COEN from Australia has a rating of 2.2 stars.       |
|US                                                         |Dan FAWCETT from United States has a rating of 3.9 stars.  |
|US                                                         |Frank SMITH from United States has a rating of 4.4 stars.  |
|US                                                         |Eve FAWCETT from United States has a rating of 4.9 stars.  |
|GB                                                         |Heidi JONES from United Kingdom has a rating of 4.9 stars. |
|AU                                                         |Dan DOTTY from Australia has a rating of 2.2 stars.        |
|GB                                                         |Grace JONES from United Kingdom has a rating of 3.4 stars. |
^CQuery terminated
ksql> 
```

--------



