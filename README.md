

# Hive Programming with Hadoop - Docker

This guide provides step-by-step instructions for setting up Hive tables and running queries in a Hadoop environment using Docker. It includes commands for creating external and staging tables, loading data, and querying with Hive.

## Prerequisites

Ensure Docker is installed on your system. The data file `data.csv` should be available in the directory `/home/hadoop/data` inside the container.

## Docker Setup and Initial Commands

1. **Run the Docker Container**  
   Start the Hadoop container with the specified ports and volume mapping:
   ```bash
   docker run -p 9870:9870 -p 8088:8088 -v D:/hedoop:/home/hadoop/data -it --name=hadoop macio232/hadoop-pseudo-distributed-mode
   ```

2. **Start Hadoop Services**  
   ```bash
   docker start hadoop
   docker exec -it hadoop /bin/bash
   start-all.sh
   ```

## Setting Up HDFS Directories and Files

1. **Create Directories and Upload CSV File to HDFS**  
   ```bash
   hdfs dfs -mkdir -p /user/hadoop/folder119
   hdfs dfs -put /home/hadoop/data/data.csv /user/hadoop/folder119/
   ```

2. **Open Hive CLI**  
   ```bash
   hive
   ```

3. **Check Databases**  
   List available databases and switch to the student database (create it if necessary):
   ```sql
   SHOW DATABASES;
   CREATE  DATABASE student119;
   USE student119;
   ```

## Creating Tables in Hive

### 1. External Table - `Info119`

Create an external partitioned table named `Info119`:
```sql
CREATE EXTERNAL TABLE Info119 (
    Id INT,
    Name STRING,
    Age INT
)
PARTITIONED BY (Gender STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/hadoop/folder119/info/';
```

### 2. Staging Table - `Info119_staging`

Create a staging table to load data initially:
```sql
CREATE TABLE Info119_staging (
    Id INT,
    Name STRING,
    Age INT,
    Gender STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

## Loading Data into Hive Tables

1. **Load Data into Staging Table**  
   Load the data from the HDFS path into the staging table:
   ```sql
   LOAD DATA INPATH '/user/hadoop/folder119/data.csv' OVERWRITE INTO TABLE Info119_staging;
   ```

2. **Insert Data with Partitioning into `Info119`**  
   Enable dynamic partitioning and insert data from the staging table:
   ```sql
   SET hive.exec.dynamic.partition.mode=nonstrict;

   INSERT OVERWRITE TABLE Info119
   PARTITION (Gender)
   SELECT Id, Name, Age, Gender
   FROM Info119_staging;
   ```

## Querying and Viewing Results

1. **Retrieve Data from `Info119`**  
   ```sql
   SELECT * FROM Info119;
   ```

2. **Check Partitions in `Info119`**  
   ```sql
   SHOW PARTITIONS Info119;
   ```

3. **View Data Files in HDFS**  
   Access the partitioned data files:
   ```bash
   hdfs dfs -ls /user/hadoop/folder119/info
   hdfs dfs -cat /user/hadoop/folder119/info/gender=F/000000_0
   ```

## Working with Address Data

### 1. External Table - `Address119`

Create an external partitioned table for address data:
```sql
DROP TABLE IF EXISTS Address119;

CREATE EXTERNAL TABLE Address119 (
    Id INT
)
PARTITIONED BY (City STRING)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
LOCATION '/user/hadoop/folder119/address/';
```

### 2. Staging Table - `Address119_staging`

Create a staging table to load data initially:
```sql
CREATE TABLE Address119_staging (
    Id INT,
    Name STRING,
    Age INT,
    Gender STRING,
    City STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ',';
```

### 3. Load Data and Insert into Partitioned Table

1. **Load Data into `Address119_staging`**  
   ```sql
   LOAD DATA INPATH '/user/hadoop/folder119/data.csv' OVERWRITE INTO TABLE Address119_staging;
   ```

2. **Insert Data with Partitioning into `Address119`**  
   ```sql
   SET hive.exec.dynamic.partition.mode=nonstrict;

   INSERT OVERWRITE TABLE Address119
   PARTITION (City)
   SELECT Id, City
   FROM Address119_staging;
   ```

3. **Retrieve Data from `Address119`**  
   ```sql
   SELECT * FROM Address119;
   ```

4. **Check Partitions in `Address119`**  
   ```sql
   SHOW PARTITIONS Address119;
   ```

## Advanced Queries

1. **Filter by Gender in `Info119`**  
   Retrieve records for a specific gender:
   ```sql
   SELECT * FROM Info119 WHERE Gender = 'M';
   SELECT * FROM Info119 WHERE Gender = 'F';
   ```

2. **Join `Info119` and `Address119` on `Id`**  
   ```sql
   SELECT i.Id, i.Name, i.Age, i.Gender, a.City
   FROM Info119 i
   JOIN Address119 a ON i.Id = a.Id;
   ```

3. **Join with Filter**  
   Retrieve data for females in a specific city:
   ```sql
   SELECT i.Id, i.Name, i.Age, i.Gender, a.City
   FROM Info119 i
   JOIN Address119 a ON i.Id = a.Id
   WHERE i.Gender = 'F' AND a.City = 'K';
   ```

---

This README covers Hive table setup, data loading, and querying in a Hadoop environment using Docker. Adjust paths as needed based on your setup.