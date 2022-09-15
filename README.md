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
ksql> print 'USERS';

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
ksql> CREATE STREAM userprofile (userid INT, firstname VARCHAR, lastname VARCHAR, countrycode VARCHAR, rating DOUBLE) WITH (VALUE_FORMAT = 'JSON', KAFKA_TOPIC = 'USERPROFILE');

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
ksql> describe USERPROFILE extended;

Name                 : USERPROFILE
Type                 : STREAM
Timestamp field      : Not set - using <ROWTIME>
Key format           : KAFKA
Value format         : JSON
Kafka topic          : USERPROFILE (partitions: 1, replication: 1)
Statement            : CREATE STREAM USERPROFILE (USERID INTEGER, FIRSTNAME STRING, LASTNAME STRING, COUNTRYCODE STRING, RATING DOUBLE) WITH (KAFKA_TOPIC='USERPROFILE', KEY_FORMAT='KAFKA', VALUE_FORMAT='JSON');

 Field       | Type            
-------------------------------
 USERID      | INTEGER         
 FIRSTNAME   | VARCHAR(STRING) 
 LASTNAME    | VARCHAR(STRING) 
 COUNTRYCODE | VARCHAR(STRING) 
 RATING      | DOUBLE          
-------------------------------

Local runtime statistics
------------------------


(Statistics of the local KSQL server interaction with the Kafka topic USERPROFILE)
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
ksql> describe USER_PROFILE_PRETTY extended;

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


ksql> terminate CSAS_USER_PROFILE_PRETTY_7;

 Message           
-------------------
 Query terminated. 
-------------------

ksql> drop stream user_profile_pretty;

 Message                                                                
------------------------------------------------------------------------
 Source `USER_PROFILE_PRETTY` (topic: USER_PROFILE_PRETTY) was dropped. 
------------------------------------------------------------------------
ksql>

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
from USERPROFILE up 
left join COUNTRYTABLE ct on ct.countrycode=up.countrycode emit changes;
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
select up.firstname 
+ ' ' + ucase(up.lastname) 
+ ' from ' + ct.countryname
+ ' has a rating of ' + cast(rating as varchar) + ' stars.' as description 
, up.countrycode
from USERPROFILE up 
left join COUNTRYTABLE ct on ct.countrycode=up.countrycode;

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

# Pull Queries

```
ksql> CREATE STREAM driverLocations (driverId VARCHAR KEY, countrycode VARCHAR, city VARCHAR, driverName VARCHAR)
WITH (kafka_topic='driverlocations', value_format='json', partitions=1);

 Message        
----------------
 Stream created 
----------------

ksql> INSERT INTO driverLocations (driverId, countrycode, city, driverName) VALUES ('1', 'AU', 'Sydney', 'Alice');
ksql> INSERT INTO driverLocations (driverId, countrycode, city, driverName) VALUES ('2', 'AU', 'Melbourne', 'Bob');
ksql> INSERT INTO driverLocations (driverId, countrycode, city, driverName) VALUES ('3', 'GB', 'London', 'Carole');
ksql> INSERT INTO driverLocations (driverId, countrycode, city, driverName) VALUES ('4', 'US', 'New York', 'Derek');
```

```
ksql> create table countryDrivers as select countrycode, count(*) as numDrivers from driverLocations group by countrycode;

 Message                                      
----------------------------------------------
 Created query with ID CTAS_COUNTRYDRIVERS_17 
----------------------------------------------
```

```
ksql> select countrycode, numdrivers from countryDrivers where countrycode='AU';
+-----------------------------------------------------------+-----------------------------------------------------------+
|COUNTRYCODE                                                |NUMDRIVERS                                                 |
+-----------------------------------------------------------+-----------------------------------------------------------+
|AU                                                         |2                                                          |
Query terminated
ksql> 
```

```
ksql> select * from countryDrivers emit changes;
+--------------------------------------------------------------+--------------------------------------------------------------+
|COUNTRYCODE                                                   |NUMDRIVERS                                                    |
+--------------------------------------------------------------+--------------------------------------------------------------+
|AU                                                            |2                                                             |
|GB                                                            |1                                                             |
|US                                                            |1                                                             |

```

```
INSERT INTO driverLocations (driverId, countrycode, city, driverName) VALUES ('5', 'AU', 'Sydney', 'Emma');

select countrycode, numdrivers from countryDrivers where countrycode='AU';
```
------------

# Kafka Connect with ksqlDB

- Execute below on PostgresDB

```
CREATE TABLE carusers (
    username VARCHAR
  , ref SERIAL PRIMARY KEY
  );

INSERT INTO carusers (username) VALUES ('Alice');
INSERT INTO carusers (username) VALUES ('Bob');
INSERT INTO carusers (username) VALUES ('Charlie');
```

# steps to install kafka connectors


```
confluent-hub install debezium/debezium-connector-postgresql:1.9.3

The component can be installed in any of the following Confluent Platform installations: 
  1. /Users/prats/confluent-7.0.1 (based on $CONFLUENT_HOME) 
  2. /Users/prats/confluent-7.0.1 (where this tool is installed) 
Choose one of these to continue the installation (1-2): 1
Do you want to install this into /Users/prats/confluent-7.0.1/share/confluent-hub-components? (yN) y

 
Component's license: 
Apache 2.0 
https://github.com/debezium/debezium/blob/master/LICENSE.txt 
I agree to the software license agreement (yN) y

You are about to install 'debezium-connector-postgresql' from Debezium Community, as published on Confluent Hub. 
Do you want to continue? (yN) y

Downloading component Debezium PostgreSQL CDC Connector 1.9.3, provided by Debezium Community from Confluent Hub and installing into /Users/prats/confluent-7.0.1/share/confluent-hub-components 
Detected Worker's configs: 
  1. Standard: /Users/prats/confluent-7.0.1/etc/kafka/connect-distributed.properties 
  2. Standard: /Users/prats/confluent-7.0.1/etc/kafka/connect-standalone.properties 
  3. Standard: /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-distributed.properties 
  4. Standard: /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-standalone.properties 
  5. Based on CONFLUENT_CURRENT: /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
  6. Used by Connect process with PID 41219: /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
Do you want to update all detected configs? (yN) y

Adding installation directory to plugin path in the following files: 
  /Users/prats/confluent-7.0.1/etc/kafka/connect-distributed.properties 
  /Users/prats/confluent-7.0.1/etc/kafka/connect-standalone.properties 
  /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-distributed.properties 
  /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-standalone.properties 
  /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
  /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
 
Completed 


confluent-hub install confluentinc/kafka-connect-jdbc:10.5.2
The component can be installed in any of the following Confluent Platform installations: 
  1. /Users/prats/confluent-7.0.1 (based on $CONFLUENT_HOME) 
  2. /Users/prats/confluent-7.0.1 (where this tool is installed) 
Choose one of these to continue the installation (1-2): 1
Do you want to install this into /Users/prats/confluent-7.0.1/share/confluent-hub-components? (yN) y

 
Component's license: 
Confluent Community License 
https://www.confluent.io/confluent-community-license 
I agree to the software license agreement (yN) y

Downloading component Kafka Connect JDBC 10.5.2, provided by Confluent, Inc. from Confluent Hub and installing into /Users/prats/confluent-7.0.1/share/confluent-hub-components 
Detected Worker's configs: 
  1. Standard: /Users/prats/confluent-7.0.1/etc/kafka/connect-distributed.properties 
  2. Standard: /Users/prats/confluent-7.0.1/etc/kafka/connect-standalone.properties 
  3. Standard: /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-distributed.properties 
  4. Standard: /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-standalone.properties 
  5. Based on CONFLUENT_CURRENT: /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
Do you want to update all detected configs? (yN) y

Adding installation directory to plugin path in the following files: 
  /Users/prats/confluent-7.0.1/etc/kafka/connect-distributed.properties 
  /Users/prats/confluent-7.0.1/etc/kafka/connect-standalone.properties 
  /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-distributed.properties 
  /Users/prats/confluent-7.0.1/etc/schema-registry/connect-avro-standalone.properties 
  /var/folders/kn/4wr9__651l37hckxvnnwt4hh0000gn/T/confluent.218210/connect/connect.properties 
 
Completed 
@Prateeks-MacBook-Pro ksql-course-master % 
```

```
ksql> CREATE SOURCE CONNECTOR `postgres-jdbc-source` WITH (
>   "connector.class"='io.confluent.connect.jdbc.JdbcSourceConnector',
>   "connection.url"='jdbc:postgresql://localhost:5432/postgres',
>   "mode"='incrementing',
>   "incrementing.column.name"='ref',
>   "table.whitelist"='carusers',
>   "connection.password"='postgres',
>   "connection.user"='postgres',
>   "topic.prefix"='db-',
>   "key"='username');

 Message                                
----------------------------------------
 Created connector postgres-jdbc-source 
----------------------------------------
ksql> 
```

# Connectors - https://www.confluent.io/hub/

<img width="1424" alt="Screenshot 2022-09-14 at 10 38 19 PM" src="https://user-images.githubusercontent.com/54174687/190218601-dba67153-0950-4a73-805a-939bf85c6e26.png">

```
ksql> print 'db-carusers' from beginning;
Key format: KAFKA_STRING
Value format: AVRO or KAFKA_STRING
rowtime: 2022/09/14 17:02:35.159 Z, key: Alice, value: {"username": "Alice", "ref": 1}, partition: 0
rowtime: 2022/09/14 17:02:35.164 Z, key: Bob, value: {"username": "Bob", "ref": 2}, partition: 0
rowtime: 2022/09/14 17:02:35.1

```

Now try to insert record in PostgresDB named as Derek and getting below 

```
ksql> print 'db-carusers' from beginning;
Key format: KAFKA_STRING
Value format: AVRO or KAFKA_STRING
rowtime: 2022/09/14 17:02:35.159 Z, key: Alice, value: {"username": "Alice", "ref": 1}, partition: 0
rowtime: 2022/09/14 17:02:35.164 Z, key: Bob, value: {"username": "Bob", "ref": 2}, partition: 0
rowtime: 2022/09/14 17:02:35.164 Z, key: Charlie, value: {"username": "Charlie", "ref": 3}, partition: 0
rowtime: 2022/09/14 17:05:45.179 Z, key: Derek, value: {"username": "Derek", "ref": 4}, partition: 0
```
----------

# CSV Delimitted Data

```sh
kafka-topics --bootstrap-server localhost:9092 --create --partitions 1 --replication-factor 1 --topic COMPLAINTS_CSV
```

```
ksql> CREATE STREAM complaints_csv (customer_name VARCHAR, complaint_type VARCHAR, trip_cost DOUBLE, new_customer BOOLEAN) \
>  WITH (VALUE_FORMAT = 'DELIMITED', KAFKA_TOPIC = 'COMPLAINTS_CSV');

 Message        
----------------
 Stream created 
----------------

ksql> select * from complaints_csv emit changes;
+------------------------------+------------------------------+------------------------------+------------------------------+
|CUSTOMER_NAME                 |COMPLAINT_TYPE                |TRIP_COST                     |NEW_CUSTOMER                  |
+------------------------------+------------------------------+------------------------------+------------------------------+
|Alice                         | Late arrival                 |43.1                          |false                         |
```

```sh
kafka-console-producer --bootstrap-server localhost:9092 --topic COMPLAINTS_CSV
>Alice, Late arrival, 43.10, true       
```

# CSV - experience with bad data

when u sent bad data, nothing will in thw stream, instead you should be able to see logs of ksql-server for error.

```
kafka-console-producer --bootstrap-server localhost:9092 --topic COMPLAINTS_CSV
>Alice, Late arrival, 43.10, true      
>Alice, Bob and Carole, Bad driver, 43.10, true
```

-----

# JSON Data

```sh
kafka-topics --bootstrap-server localhost:9092 --create --partitions 1 --replication-factor 1 --topic COMPLAINTS_JSON
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic COMPLAINTS_JSON.
```

```
ksql> CREATE STREAM complaints_json (customer_name VARCHAR, complaint_type VARCHAR, trip_cost DOUBLE, new_customer BOOLEAN) \
>  WITH (VALUE_FORMAT = 'JSON', KAFKA_TOPIC = 'COMPLAINTS_JSON');

 Message        
----------------
 Stream created 
----------------
ksql> 

ksql> select * from complaints_json emit changes;
+------------------------------+------------------------------+------------------------------+------------------------------+
|CUSTOMER_NAME                 |COMPLAINT_TYPE                |TRIP_COST                     |NEW_CUSTOMER                  |
+------------------------------+------------------------------+------------------------------+------------------------------+
|Alice, Bob and Carole         |Bad driver                    |22.4                          |true                          |
```

```
kafka-console-producer --bootstrap-server localhost:9092 --topic COMPLAINTS_JSON
>{"customer_name":"Alice, Bob and Carole", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": true}    
>
```

# experience with bad data

```
kafka-console-producer --bootstrap-server localhost:9092 --topic COMPLAINTS_JSON
>{"customer_name":"Alice, Bob and Carole", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": true}    
>{"customer_name":"Bad Data", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": ShouldBeABoolean}
```

Review the KSQL Server logs `confluent local services ksql-server log`

Now look at the KSQL Server log. We can see bad data is noticed; but hidden in a conversion error message

```
  at [Source: (byte[])"{"customer_name":"Bad Data", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": ShouldBeABoolean}"; line: 1, column: 105]
 Caused by: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'ShouldBeABoolean': was expecting ('true', 'false' or 'null')
 ```
 --------------
 
 # Avro Data
 
 ```sh
 ksql-course-master % kafka-topics --bootstrap-server localhost:9092 --create --partitions 1 --replication-factor 1 --topic COMPLAINTS_AVRO
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic COMPLAINTS_AVRO.
 ```
 
 ```
 ksql-course-master % kafka-avro-console-producer  --bootstrap-server localhost:9092 --topic COMPLAINTS_AVRO \
--property value.schema='
{
  "type": "record",
  "name": "myrecord",
  "fields": [
      {"name": "customer_name",  "type": "string" }
    , {"name": "complaint_type", "type": "string" }
    , {"name": "trip_cost", "type": "float" }
    , {"name": "new_customer", "type": "boolean"}
  ]
}'
{"customer_name":"Carol", "complaint_type":"Late arrival", "trip_cost": 19.60, "new_customer": false}

```
 
```
ksql> print 'COMPLAINTS_AVRO' from beginning;
Key format: ¯\_(ツ)_/¯ - no data processed
Value format: AVRO
rowtime: 2022/09/15 05:15:53.450 Z, key: <null>, value: {"customer_name": "Carol", "complaint_type": "Late arrival", "trip_cost": 19.6, "new_customer": false}, partition: 0


ksql> create stream complaints_avro with (kafka_topic='COMPLAINTS_AVRO', value_format='AVRO');

 Message        
----------------
 Stream created 
----------------
ksql> 

ksql> select * from complaints_avro emit changes;
+------------------------------+------------------------------+------------------------------+------------------------------+
|CUSTOMER_NAME                 |COMPLAINT_TYPE                |TRIP_COST                     |NEW_CUSTOMER                  |
+------------------------------+------------------------------+------------------------------+------------------------------+
|Carol                         |Late arrival                  |19.600000381469727            |false                         |

```

# experience with bad data

```
ksql-course-master % kafka-avro-console-producer  --bootstrap-server localhost:9092 --topic COMPLAINTS_AVRO \
--property value.schema='
{
  "type": "record",
  "name": "myrecord",
  "fields": [
      {"name": "customer_name",  "type": "string" }
    , {"name": "complaint_type", "type": "string" }
    , {"name": "trip_cost", "type": "float" }
    , {"name": "new_customer", "type": "boolean"}
  ]
}'
{"customer_name":"Carol", "complaint_type":"Late arrival", "trip_cost": 19.60, "new_customer": false}
{"customer_name":"Bad Data", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": ShouldBeABoolean}
org.apache.kafka.common.errors.SerializationException: Error deserializing json {"customer_name":"Bad Data", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": ShouldBeABoolean} to Avro of schema {"type":"record","name":"myrecord","fields":[{"name":"customer_name","type":"string"},{"name":"complaint_type","type":"string"},{"name":"trip_cost","type":"float"},{"name":"new_customer","type":"boolean"}]}
	at io.confluent.kafka.formatter.AvroMessageReader.readFrom(AvroMessageReader.java:131)
	at io.confluent.kafka.formatter.SchemaMessageReader.readMessage(SchemaMessageReader.java:325)
	at kafka.tools.ConsoleProducer$.main(ConsoleProducer.scala:51)
	at kafka.tools.ConsoleProducer.main(ConsoleProducer.scala)
Caused by: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'ShouldBeABoolean': was expecting (JSON String, Number, Array, Object or token 'null', 'true' or 'false')
 at [Source: (String)"{"customer_name":"Bad Data", "complaint_type":"Bad driver", "trip_cost": 22.40, "new_customer": ShouldBeABoolean}"; line: 1, column: 113]
	at com.fasterxml.jackson.core.JsonParser._constructError(JsonParser.java:2337)
	at com.fasterxml.jackson.core.base.ParserMinimalBase._reportError(ParserMinimalBase.java:720)
	at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._reportInvalidToken(ReaderBasedJsonParser.java:2903)
	at com.fasterxml.jackson.core.json.ReaderBasedJsonParser._handleOddValue(ReaderBasedJsonParser.java:1949)
	at com.fasterxml.jackson.core.json.ReaderBasedJsonParser.nextToken(ReaderBasedJsonParser.java:781)
	at org.apache.avro.io.JsonDecoder.readFloat(JsonDecoder.java:186)
	at org.apache.avro.io.ResolvingDecoder.readFloat(ResolvingDecoder.java:182)
	at org.apache.avro.generic.GenericDatumReader.readWithoutConversion(GenericDatumReader.java:199)
	at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:160)
	at org.apache.avro.generic.GenericDatumReader.readField(GenericDatumReader.java:259)
	at org.apache.avro.generic.GenericDatumReader.readRecord(GenericDatumReader.java:247)
	at org.apache.avro.generic.GenericDatumReader.readWithoutConversion(GenericDatumReader.java:179)
	at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:160)
	at org.apache.avro.generic.GenericDatumReader.read(GenericDatumReader.java:153)
	at io.confluent.kafka.schemaregistry.avro.AvroSchemaUtils.toObject(AvroSchemaUtils.java:213)
	at io.confluent.kafka.formatter.AvroMessageReader.readFrom(AvroMessageReader.java:124)
	... 3 more
prateekashtikar@Prateeks-MacBook-Pro ksql-course-master % 
```
----

# Avro Schema Evolution

```sh
kafka-avro-console-producer  --broker-list localhost:9092 --topic COMPLAINTS_AVRO \
--property value.schema='
{
  "type": "record",
  "name": "myrecord",
  "fields": [
      {"name": "customer_name",  "type": "string" }
    , {"name": "complaint_type", "type": "string" }
    , {"name": "trip_cost", "type": "float" }
    , {"name": "new_customer", "type": "boolean"}
    , {"name": "number_of_rides", "type": "int", "default" : 1}
  ]
}'
{"customer_name":"Ed", "complaint_type":"Dirty car", "trip_cost": 29.10, "new_customer": false, "number_of_rides": 22}
```

```
ksql> select * from complaints_avro emit changes;
+------------------------------+------------------------------+------------------------------+------------------------------+
|CUSTOMER_NAME                 |COMPLAINT_TYPE                |TRIP_COST                     |NEW_CUSTOMER                  |
+------------------------------+------------------------------+------------------------------+------------------------------+
|Carol                         |Late arrival                  |19.600000381469727            |false                         |
|Ed                            |Dirty car                     |29.100000381469727            |false                         |
^CQuery terminated
ksql> 


ksql> describe complaints_avro;

Name                 : COMPLAINTS_AVRO
 Field          | Type            
----------------------------------
 CUSTOMER_NAME  | VARCHAR(STRING) 
 COMPLAINT_TYPE | VARCHAR(STRING) 
 TRIP_COST      | DOUBLE          
 NEW_CUSTOMER   | BOOLEAN         
----------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
ksql>
```

<img width="1129" alt="Screenshot 2022-09-15 at 11 02 11 AM" src="https://user-images.githubusercontent.com/54174687/190322279-327cec37-5775-4f7a-89e3-c455d271549f.png">

```
ksql> create stream complaints_avro_v2 with (kafka_topic='COMPLAINTS_AVRO', value_format='AVRO');

 Message        
----------------
 Stream created 
----------------
ksql> describe complaints_avro_v2;

Name                 : COMPLAINTS_AVRO_V2
 Field           | Type            
-----------------------------------
 CUSTOMER_NAME   | VARCHAR(STRING) 
 COMPLAINT_TYPE  | VARCHAR(STRING) 
 TRIP_COST       | DOUBLE          
 NEW_CUSTOMER    | BOOLEAN         
 NUMBER_OF_RIDES | INTEGER         
-----------------------------------
For runtime statistics and query details run: DESCRIBE <Stream,Table> EXTENDED;
ksql> 

ksql> list streams;

 Stream Name         | Kafka Topic                 | Key Format | Value Format | Windowed 
------------------------------------------------------------------------------------------
 COMPLAINTS_AVRO     | COMPLAINTS_AVRO             | KAFKA      | AVRO         | false    
 COMPLAINTS_AVRO_V2  | COMPLAINTS_AVRO             | KAFKA      | AVRO         | false    
 COMPLAINTS_CSV      | COMPLAINTS_CSV              | KAFKA      | DELIMITED    | false    
 COMPLAINTS_JSON     | COMPLAINTS_JSON             | KAFKA      | JSON         | false    
 KSQL_PROCESSING_LOG | default_ksql_processing_log | KAFKA      | JSON         | false    
------------------------------------------------------------------------------------------
ksql> 

ksql> select * from complaints_avro_v2 emit changes;
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
|CUSTOMER_NAME          |COMPLAINT_TYPE         |TRIP_COST              |NEW_CUSTOMER           |NUMBER_OF_RIDES        |
+-----------------------+-----------------------+-----------------------+-----------------------+-----------------------+
^CQuery terminated
ksql> 
```

-------

# Nested JSON

```
kafka-topics --bootstrap-server localhost:9092 --create --partitions 1 --replication-factor 1 --topic WEATHERNESTED
Created topic WEATHERNESTED.

cat demo-weather.json | kafka-console-producer --bootstrap-server localhost:9092 --topic WEATHERNESTED

```

```
ksql> CREATE STREAM weather 
      (city STRUCT <name VARCHAR, country VARCHAR, latitude DOUBLE, longitude DOUBLE>, 
       description VARCHAR, 
       clouds BIGINT, 
       deg BIGINT, 
       humidity BIGINT, 
       pressure DOUBLE, 
       rain DOUBLE) 
>WITH (KAFKA_TOPIC='WEATHERNESTED', VALUE_FORMAT='JSON');

 Message        
----------------
 Stream created 
----------------


ksql> select * from weather emit changes;
+---------------+---------------+---------------+---------------+---------------+---------------+---------------+
|CITY           |DESCRIPTION    |CLOUDS         |DEG            |HUMIDITY       |PRESSURE       |RAIN           |
+---------------+---------------+---------------+---------------+---------------+---------------+---------------+
|{NAME=Sydney, C|light rain     |92             |26             |94             |1025.12        |1.25           |
|OUNTRY=AU, LATI|               |               |               |               |               |               |
|TUDE=-33.8688, |               |               |               |               |               |               |
|LONGITUDE=151.2|               |               |               |               |               |               |
|093}           |               |               |               |               |               |               |
|{NAME=Seattle, |heavy rain     |92             |19             |94             |1025.12        |7.0            |
|COUNTRY=US, LAT|               |               |               |               |               |               |
|ITUDE=47.6062, |               |               |               |               |               |               |
|LONGITUDE=-122.|               |               |               |               |               |               |
|3321}          |               |               |               |               |               |               |
|{NAME=San Franc|fog            |92             |19             |94             |1025.12        |10.0           |
|isco, COUNTRY=U|               |               |               |               |               |               |
|S, LATITUDE=37.|               |               |               |               |               |               |
|7749, LONGITUDE|               |               |               |               |               |               |
|=-122.4194}    |               |               |               |               |               |               |
|{NAME=San Jose,|light rain     |92             |23             |94             |1025.12        |3.0            |
| COUNTRY=US, LA|               |               |               |               |               |               |
|TITUDE=37.3382,|               |               |               |               |               |               |
| LONGITUDE=-121|               |               |               |               |               |               |
|.8863}         |               |               |               |               |               |               |
|{NAME=Fresno, C|heavy rain     |92             |22             |94             |1025.12        |6.0            |
|OUNTRY=US, LATI|               |               |               |               |               |               |
|TUDE=36.7378, L|               |               |               |               |               |               |
|ONGITUDE=-119.7|               |               |               |               |               |               |
|871}           |               |               |               |               |               |               |
|{NAME=Los Angel|haze           |92             |19             |94             |1025.12        |2.0            |
|es, COUNTRY=US,|               |               |               |               |               |               |
| LATITUDE=34.05|               |               |               |               |               |               |
|22, LONGITUDE=-|               |               |               |               |               |               |
|118.2437}      |               |               |               |               |               |               |
|{NAME=San Diego|fog            |92             |19             |94             |1025.12        |2.0            |
|, COUNTRY=US, L|               |               |               |               |               |               |
|ATITUDE=32.7157|               |               |               |               |               |               |
|, LONGITUDE=-11|               |               |               |               |               |               |
|7.1611}        |               |               |               |               |               |               |
|{NAME=Birmingha|light rain     |92             |26             |94             |1025.12        |4.0            |
|m, COUNTRY=UK, |               |               |               |               |               |               |
|LATITUDE=52.486|               |               |               |               |               |               |
|2, LONGITUDE=-1|               |               |               |               |               |               |
|.8904}         |               |               |               |               |               |               |
|{NAME=London, C|heavy rain     |92             |19             |94             |1025.12        |8.0            |
|OUNTRY=GB, LATI|               |               |               |               |               |               |
|TUDE=51.5074, L|               |               |               |               |               |               |
|ONGITUDE=-0.127|               |               |               |               |               |               |
|8}             |               |               |               |               |               |               |
|{NAME=Mancheste|fog            |92             |26             |94             |1025.12        |3.0            |
|r, COUNTRY=GB, |               |               |               |               |               |               |
|LATITUDE=53.480|               |               |               |               |               |               |
|8, LONGITUDE=-2|               |               |               |               |               |               |
|.2426}         |               |               |               |               |               |               |
|{NAME=Bristol, |light rain     |92             |19             |94             |1025.12        |3.0            |
|COUNTRY=GB, LAT|               |               |               |               |               |               |
|ITUDE=51.4545, |               |               |               |               |               |               |
|LONGITUDE=-2.58|               |               |               |               |               |               |
|79}            |               |               |               |               |               |               |
|{NAME=Newcastle|heavy rain     |92             |19             |94             |1025.12        |12.0           |
|, COUNTRY=GB, L|               |               |               |               |               |               |
|ATITUDE=54.9783|               |               |               |               |               |               |
|, LONGITUDE=-1.|               |               |               |               |               |               |
|6178}          |               |               |               |               |               |               |
|{NAME=Liverpool|haze           |92             |23             |94             |1025.12        |3.0            |
|, COUNTRY=GB, L|               |               |               |               |               |               |
|ATITUDE=53.4084|               |               |               |               |               |               |
|, LONGITUDE=-2.|               |               |               |               |               |               |
|9916}          |               |               |               |               |               |               |


ksql> SELECT city->name AS city_name, city->country AS city_country, city->latitude as latitude, city->longitude as longitude, description, rain from weather emit changes;
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|CITY_NAME                |CITY_COUNTRY             |LATITUDE                 |LONGITUDE                |DESCRIPTION              |RAIN                     |
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|Sydney                   |AU                       |-33.8688                 |151.2093                 |light rain               |1.25                     |
|Seattle                  |US                       |47.6062                  |-122.3321                |heavy rain               |7.0                      |
|San Francisco            |US                       |37.7749                  |-122.4194                |fog                      |10.0                     |
|San Jose                 |US                       |37.3382                  |-121.8863                |light rain               |3.0                      |
|Fresno                   |US                       |36.7378                  |-119.7871                |heavy rain               |6.0                      |
|Los Angeles              |US                       |34.0522                  |-118.2437                |haze                     |2.0                      |
|San Diego                |US                       |32.7157                  |-117.1611                |fog                      |2.0                      |
|Birmingham               |UK                       |52.4862                  |-1.8904                  |light rain               |4.0                      |
|London                   |GB                       |51.5074                  |-0.1278                  |heavy rain               |8.0                      |
|Manchester               |GB                       |53.4808                  |-2.2426                  |fog                      |3.0                      |
|Bristol                  |GB                       |51.4545                  |-2.5879                  |light rain               |3.0                      |
|Newcastle                |GB                       |54.9783                  |-1.6178                  |heavy rain               |12.0                     |
|Liverpool                |GB                       |53.4084                  |-2.9916                  |haze                     |3.0                      |

Press CTRL-C to interrupt
```
--------

# Build a rekeyed table

```
ksql> create stream weatherraw with (value_format='AVRO') as SELECT city->name AS city_name, city->country AS city_country, city->latitude as latitude, city->longitude as longitude, description, rain from weather ;  

 Message                                 
-----------------------------------------
 Created query with ID CSAS_WEATHERRAW_3 
-----------------------------------------
ksql>

ksql> create stream weatherrekeyed as select * from weatherraw partition by city_name;

 Message                                     
---------------------------------------------
 Created query with ID CSAS_WEATHERREKEYED_5 
---------------------------------------------
ksql> 

ksql> select * from weatherrekeyed emit changes;
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|CITY_NAME                |CITY_COUNTRY             |LATITUDE                 |LONGITUDE                |DESCRIPTION              |RAIN                     |
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|Sydney                   |AU                       |-33.8688                 |151.2093                 |light rain               |1.25                     |
|Seattle                  |US                       |47.6062                  |-122.3321                |heavy rain               |7.0                      |
|San Francisco            |US                       |37.7749                  |-122.4194                |fog                      |10.0                     |
|San Jose                 |US                       |37.3382                  |-121.8863                |light rain               |3.0                      |
|Fresno                   |US                       |36.7378                  |-119.7871                |heavy rain               |6.0                      |
|Los Angeles              |US                       |34.0522                  |-118.2437                |haze                     |2.0                      |
|San Diego                |US                       |32.7157                  |-117.1611                |fog                      |2.0                      |
|Birmingham               |UK                       |52.4862                  |-1.8904                  |light rain               |4.0                      |
|London                   |GB                       |51.5074                  |-0.1278                  |heavy rain               |8.0                      |
|Manchester               |GB                       |53.4808                  |-2.2426                  |fog                      |3.0                      |
|Bristol                  |GB                       |51.4545                  |-2.5879                  |light rain               |3.0                      |
|Newcastle                |GB                       |54.9783                  |-1.6178                  |heavy rain               |12.0                     |
|Liverpool                |GB                       |53.4084                  |-2.9916                  |haze                     |3.0                      |
^CQuery terminated


ksql> create table weathernow (city_name varchar primary key, city_country varchar, latitude double, longitude double, description varchar, rain double) with (kafka_topic='WEATHERREKEYED', value_format='AVRO');

 Message       
---------------
 Table created 
---------------

ksql> select * from weathernow emit changes;
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|CITY_NAME                |CITY_COUNTRY             |LATITUDE                 |LONGITUDE                |DESCRIPTION              |RAIN                     |
+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+-------------------------+
|Sydney                   |AU                       |-33.8688                 |151.2093                 |light rain               |1.25                     |
|Seattle                  |US                       |47.6062                  |-122.3321                |heavy rain               |7.0                      |
|San Francisco            |US                       |37.7749                  |-122.4194                |fog                      |10.0                     |
|San Jose                 |US                       |37.3382                  |-121.8863                |light rain               |3.0                      |
|Fresno                   |US                       |36.7378                  |-119.7871                |heavy rain               |6.0                      |
|Los Angeles              |US                       |34.0522                  |-118.2437                |haze                     |2.0                      |
|San Diego                |US                       |32.7157                  |-117.1611                |fog                      |2.0                      |
|Birmingham               |UK                       |52.4862                  |-1.8904                  |light rain               |4.0                      |
|London                   |GB                       |51.5074                  |-0.1278                  |heavy rain               |8.0                      |
|Manchester               |GB                       |53.4808                  |-2.2426                  |fog                      |3.0                      |
|Bristol                  |GB                       |51.4545                  |-2.5879                  |light rain               |3.0                      |
|Newcastle                |GB                       |54.9783                  |-1.6178                  |heavy rain               |12.0                     |
|Liverpool                |GB                       |53.4084                  |-2.9916                  |haze                     |3.0                      |


ksql> select * from weathernow where city_name = 'San Diego' emit changes;
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|CITY_NAME                    |CITY_COUNTRY                 |LATITUDE                     |LONGITUDE                    |DESCRIPTION                  |RAIN                         |
+-----------------------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+-----------------------------+
|San Diego                    |US                           |32.7157                      |-117.1611                    |fog                          |2.0                          |
^CQuery terminated
ksql> 

```

```
cat demo-weather-changes.json | kafka-console-producer --broker-list localhost:9092 --topic WEATHERNESTED
```

```
ksql> select * from weathernow where city_name = 'San Diego' emit changes;
+----------------------------+----------------------------+----------------------------+----------------------------+----------------------------+----------------------------+
|CITY_NAME                   |CITY_COUNTRY                |LATITUDE                    |LONGITUDE                   |DESCRIPTION                 |RAIN                        |
+----------------------------+----------------------------+----------------------------+----------------------------+----------------------------+----------------------------+
|San Diego                   |US                          |32.7157                     |-117.1611                   |SUNNY                       |2.0                         |
```
------

# Repartition a Stream

```
kafka-topics --bootstrap-server localhost:9092 --create --partitions 2 --replication-factor 1 --topic DRIVER_PROFILE
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic DRIVER_PROFILE.

kafka-console-producer --bootstrap-server localhost:9092 --topic DRIVER_PROFILE
>{"driver_name":"Mr. Speedy", "countrycode":"AU", "rating":2.4}
```

```
ksql> CREATE STREAM DRIVER_PROFILE (driver_name VARCHAR, countrycode VARCHAR, rating DOUBLE) 
>  WITH (VALUE_FORMAT = 'JSON', KAFKA_TOPIC = 'DRIVER_PROFILE');

 Message        
----------------
 Stream created 
----------------


select dp.driver_name, ct.countryname, dp.rating 
from DRIVER_PROFILE dp 
left join COUNTRYTABLE ct on ct.countrycode=dp.countrycode emit changes;    

Can't join DRIVER_PROFILE with COUNTRYTABLE since the number of partitions don't match. DRIVER_PROFILE partitions = 2; COUNTRYTABLE partitions = 1. Please repartition either one so that the number of partitions match.
```
















