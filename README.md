# Docker Hive Task

This guide provides step-by-step instructions to set up Docker Hive and perform various operations using Hive queries.

## Prerequisites
Before proceeding with the setup, make sure you have the following prerequisites:

Docker installed on your system.

## Setup Instructions

### Step1: Clone the Docker Hive repository:

```
$ git clone https://github.com/big-data-europe/docker-hive.git
```

### Step2: Run Docker Compose to start the Docker Hive containers:

```
$ sudo docker compose up
```

### Step3: Move the data files to the Hive namenode container:


```
$ sudo docker cp part-m-00000 docker-hive-namenode-1:part-m-00000
$ sudo docker cp part-m-00001 docker-hive-namenode-1:part-m-00001
$ sudo docker cp part-m-00002 docker-hive-namenode-1:part-m-00002
$ sudo docker cp part-m-00003 docker-hive-namenode-1:part-m-00003
$ sudo docker cp part-m-00004 docker-hive-namenode-1:part-m-00004
$ sudo docker cp part-m-00005 docker-hive-namenode-1:part-m-00006
$ sudo docker cp part-m-00006 docker-hive-namenode-1:part-m-00006
```

### Step4: Access the Hive namenode container:

```
$ sudo docker exec -it docker-hive-namenode-1 bash
```

### Step5: Create a directory in HDFS to store the data:

```
$ hdfs dfs -mkdir -p /data
```

### Step6: Upload the data files to the HDFS directory:


```
$ hdfs dfs -put part-m-00000 /data/part-m-00000
$ hdfs dfs -put part-m-00001 /data/part-m-00001
$ hdfs dfs -put part-m-00002 /data/part-m-00002
$ hdfs dfs -put part-m-00003 /data/part-m-00003
$ hdfs dfs -put part-m-00004 /data/part-m-00004
$ hdfs dfs -put part-m-00005 /data/part-m-00005
$ hdfs dfs -put part-m-00006 /data/part-m-00006
```


## Query Operations

### Query1: Create an External Table

To create an external table named customer_table:

```
CREATE EXTERNAL TABLE customer_table (
  cust_id INT,
  creationdate STRING,
  expirationDate STRING,
  fname STRING,
  lname STRING,
  address STRING,
  city STRING,
  state STRING,
  zipcode STRING
)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```
Time taken: 2.212 seconds


### Query2: Load Data into the Table
To load data into the customer_table:

```
LOAD DATA INPATH '/data/part-m-00000' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00001' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00002' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00003' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00004' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00005' INTO TABLE customer_table;
LOAD DATA INPATH '/data/part-m-00006' INTO TABLE customer_table;
```
Time taken: 0.253 seconds


### Query3: Create a Partitioned Table
To create a new partitioned table named customer_table_state_partitioned:

```
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.exec.max.dynamic.partitions.pernode = 400;
```
```
CREATE TABLE customer_table_state_partitioned (
  cust_id INT,
  creationdate STRING,
  expirationDate STRING,
  fname STRING,
  lname STRING,
  address STRING,
  city STRING,
  zipcode STRING
)
PARTITIONED BY (state STRING);

INSERT OVERWRITE TABLE customer_table_state_partitioned PARTITION (state)
SELECT cust_id, creationdate, expirationDate, fname, lname, address, city, zipcode, state
FROM customer_table;
```

Time taken: 2.462 seconds

###Query4: Perform Queries on Partitioned Tables
To perform queries on the partitioned tables:

Get the first name and last name of all customers from the "NV" state:

```
SELECT fname, lname FROM customer_table_state_partitioned WHERE state = 'NV';
```
Time taken: 0.509 seconds, Fetched: 11017 row(s)


### Query5: Create a new table partitioned by state and city:

```
CREATE TABLE customer_table_state_city_partitioned (
  cust_id INT,
  creationdate STRING,
  expirationDate STRING,
  fname STRING,
  lname STRING,
  address STRING,
  zipcode STRING
)
PARTITIONED BY (state STRING, city STRING);

INSERT OVERWRITE TABLE customer_table_state_city_partitioned PARTITION (state, city)
SELECT cust_id, creationdate, expirationDate, fname, lname, address, zipcode, state, city
FROM customer_table;
```
Time taken: 20.373 seconds


### Query6: Retrieve the names of the customers in CA, Oakland from the regular table, the partitioned table (state), and the partitioned table (state, city):

```
SELECT fname, lname FROM customer_table WHERE state = 'CA' AND city = 'Oakland';
```
Time taken: 0.108 seconds, Fetched: 3491 row(s)

```
SELECT fname, lname FROM customer_table_state_partitioned WHERE state = 'CA' AND city = 'Oakland';
```
Time taken: 0.091 seconds, Fetched: 3491 row(s)

```
SELECT fname, lname FROM customer_table_state_city_partitioned WHERE state = 'CA' AND city = 'Oakland';
```
Time taken: 0.098 seconds, Fetched: 3491 row(s)
