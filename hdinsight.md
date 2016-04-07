## HDInsight Cheatsheet

### Basic

- Shared file system commands

- List
```
hdfs dfs -ls /
hdfs dfs -ls /example
```

- Print
```
hdfs dfs -text /example/data/test.txt
hdfs dfs -cat
```

- File
```
hdfs dfs -rm
hdfs dfs -cp
hdfs dfs -mv
```

- Remove folder
```
hdfs dfs -rm -r /data/iislogs/scripts
```

- Copy file to local storage
```
hdfs dfs -get /data/iislogs/job.properties
```

- Common task jars
```
ls /usr/hdp/current/hadoop-mapreduce-client
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount <source-data-on-shared-file-system> <result-path-on-shared-file-system>
```

### Hive

- First ssh into HDInsight head node

- Enter Hive shell
```
hive
```

- Leave Hive shell
```
EXIT;
```

- Info
```
SHOW TABLES;
```

- Default table location
```
hdfs dfs -ls /hive/warehouse
```

- Select
```
SELECT * FROM staged_log;

SELECT datetime, level, event_id
FROM system_log
LIMIT 10;

SELECT unix_timestamp(datetime, 'dd/MM/yyyy hh:mm:ss') AS time_stamp, level, event_id
FROM system_log;

SELECT from_unixtime(unix_timestamp(datetime, 'dd/MM/yyyy hh:mm:ss')) AS datetime, level, event_id
FROM system_log;
```

- Select from views
```
SELECT datetime, level, event_id
FROM v_system_log
LIMIT 10;

SELECT CAST(SUBSTR(datetime, 1, 10) AS DATE) AS event_date, level, event_id
FROM v_system_log;

SELECT CAST(SUBSTR(datetime, 1, 10) AS DATE) AS event_date, level, event_id
FROM v_system_log
ORDER BY event_date;

SELECT CAST(SUBSTR(datetime, 1, 10) AS DATE) AS event_date, COUNT(*) AS events
FROM v_system_log
GROUP BY CAST(SUBSTR(datetime, 1, 10) AS date)
ORDER BY event_date;

SELECT CAST(SUBSTR(datetime, 1, 10) AS DATE) AS event_date, level, COUNT(*) AS events
FROM v_system_log
GROUP BY CAST(SUBSTR(datetime, 1, 10) AS date), level
ORDER BY event_date, level;
```

- Create views
```
CREATE VIEW v_system_log
AS
SELECT from_unixtime(unix_timestamp(datetime, 'dd/MM/yyyy hh:mm:ss')) AS datetime, level, event_id, source, details
FROM system_log;
```

- Create external table
```
CREATE EXTERNAL TABLE staged_log
(level STRING,
 datetime STRING,
 source STRING,
 event_id INT,
 category STRING,
 details STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE LOCATION '/data/staged_log';
```

- Create internal table
```
CREATE TABLE system_log
(level STRING,
 datetime STRING,
 source STRING,
 event_id STRING,
 category STRING,
 details STRING);
```

- Create table as select
```
CREATE TABLE error_log
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE LOCATION '/data/error_log'
AS
SELECT datetime, source, event_id, details
FROM system_log
WHERE level = 'Error';
```

- Load data into table
```
LOAD DATA INPATH '/data/logs.txt' INTO TABLE staged_log;
```

- Insert data from select
```
INSERT INTO TABLE system_log
SELECT * FROM staged_log
WHERE event_id IS NOT NULL;
```

- Partition
```
CREATE TABLE part_log
(event_date DATE,
 source STRING,
 event_id INT,
 details STRING)
PARTITIONED BY (level STRING)
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE LOCATION '/data/part_log';

SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT INTO TABLE part_log PARTITION(level)
SELECT CAST(SUBSTR(datetime, 1, 10) AS DATE), source, event_id, details, level
FROM v_system_log;

SELECT event_date, event_id
FROM part_log
WHERE level = 'Error';
```

- Run hive script
```
hive -i wasb:///data/create_table.hql
```

- UDF with python (stdin/stdout)
```
#!/usr/bin/env python

import sys
import string

while True:
  line = sys.stdin.readline()
  if not line:
    break

  row = string.strip(line, "\n")
  year, month, max_temp, min_temp, frost_days, rainfall_mm, sunshine = string.split(row, "\t")
  rainfall_inches = float(rainfall_mm) / 25.4
  print "\t".join([year, month, max_temp, min_temp, frost_days, str(rainfall_inches), sunshine])
```

- Use UDF in hive script
```
add file wasb:///data/convert_rain.py;

SELECT TRANSFORM (year, month, max_temp, min_temp, frost_days, rainfall, sunshine_hours)
  USING 'python convert_rain.py' AS
  (year INT, month INT, max_temp FLOAT, min_temp FLOAT, frost_days INT, rainfall FLOAT, sunshine_hours FLOAT)
FROM weather;
```

### Pig

- Enter grunt shell
```
pig
```

- Leave grunt shell
```
QUIT;
```

- Load
```
Source = LOAD '/data/heathrow.txt' USING PigStorage('\t') AS (year:chararray, month:chararray, maxtemp:float, mintemp:float, frost:int, rainfall:float, sunshine:float);
```

- Filter (remove null values from schema shaping)
```
Data = FILTER Source BY maxtemp IS NOT NULL AND mintemp IS NOT NULL;
Readings = FILTER Data BY year != 'yyyy';
CleanReadings = FILTER DataVals BY INDEXOF(sunshinehours, '#', 0) <= 0;
DirtyReadings = FILTER DataVals BY INDEXOF(sunshinehours, '#', 0) > 0;
```

- Dump (to see results on-the-fly)
```
DUMP Data;
DUMP SortedResults;
```

- Save
```
STORE SortedReadings INTO '/data/scrubbedweather' USING PigStorage(' ');
```

- Group
```
YearGroups = GROUP Readings BY year;
```

- Foreach
```
AggTemps = FOREACH YearGroups GENERATE group AS year, AVG(Readings.maxtemp) AS avghigh, AVG(Readings.mintemp) AS avglow;
CleanedReadings = FOREACH DirtyReadings GENERATE year, month, maxtemp, mintemp, frostdays, rainfall, SUBSTRING(sunshinehours, 0, INDEXOF(sunshinehours, '#', 0)) AS sunshinehours;
```

- Sort
```
SortedResults = ORDER AggTemps BY year;
```

- Replace
```
DataVals = FOREACH Data GENERATE year, month, maxtemp, mintemp, frostdays, rainfall, REPLACE(sunshinehours, '---', '') AS sunshinehours;
```

- Union
```
Readings = UNION CleanReadings, CleanedReadings;
```

- Run pig scripts
```
pig wasb:///data/scrub_weather.pig
```

- UDF with python
```
@outputSchema("f_readings: {(year:chararray, month:int, maxtemp:float, mintemp:float, frostdays:int, rainfall:float, sunshinehours:chararray)}")
def fahrenheit(c_reading):
  year, month, maxtemp, mintemp, frostdays, rainfall, sunshine = c_reading.split(' ')
  maxtemp_f = float(maxtemp) * 9/5 + 32
  mintemp_f = float(mintemp) * 9/5 + 32
  return year, int(month), maxtemp_f, mintemp_f, frostdays, float(rainfall), sunshine
```

- Use UDF in pig script
```
REGISTER 'wasb:///data/convert_temp.py' using jython as convert_temp;

Source = LOAD '/data/scrubbedweather' AS (celsius_readings:chararray);

ConvertedReadings = FOREACH Source GENERATE FLATTEN(convert_temp.fahrenheit(celsius_readings));

STORE ConvertedReadings INTO '/data/convertedweather';
```

### Comparison

Ref: [Stackoverflow](https://stackoverflow.com/questions/10279942/what-is-the-difference-between-apache-pig-and-apache-hive/35965805#35965805)

- Hive is used as **declarative SQL** for reporting in structured data 
- Pig is used as **procedural language (script)** for programming in semi-structured data 
- Both are slower than traditional MapReduce jobs, because they _all have to be converted to MapReduce jobs_

### Oozie

- [Oozie Demo](https://github.com/r96941046/demo_oozie)

- Make sure following is configured in the config file (job.properties)
```
user.name=<ssh_user_name>
nameNode=wasb://<container>@<storage_account>.blob.core.windows.net
jobTracker=jobtrackerhost:9010
oozie.wf.application.path=wasb:///data/iislogs/oozie
oozie.wf.rerun.failnodes=true
oozie.use.system.libpath=true
queueName=default
```

- Upload to azure HDInsight (login to azure is required)
```
azure storage blob upload <local_file_name> <container_name> data/iislogs/<remote_file_name>
```

- Download config file to local storage
```
hdfs dfs -get /data/iislogs/job.properties
```

- Submit job and get job ID (not run yet)
```
oozie job -oozie http://localhost:11000/oozie -config job.properties -submit
```

- Run job
```
oozie job -oozie http://localhost:11000/oozie -config job.properties -run
```

- Job info
```
oozie job -oozie http://localhost:11000/oozie -info <job_ID>
```

- Start job
```
oozie job -oozie http://localhost:11000/oozie -start <job_ID>
```

- Rerun job
  + Note: [Need to set failed nodes in config](http://mail-archives.apache.org/mod_mbox/incubator-oozie-users/201203.mbox/%3CCAJs-t7OTEf8usUZLu=1b26Bx7Av87mDGowRNkVJ4E12f5Czd6A@mail.gmail.com%3E)
  + Note: [Remove existing folders before rerunning pig jobs](http://stackoverflow.com/questions/11110403/how-to-force-store-overwrite-to-hdfs-in-pig)

```
oozie job -oozie http://localhost:11000/oozie -config job.properties -rerun <job_ID>
```

- Error log
```
oozie job -oozie http://localhost:11000/oozie -log <job_ID>
```

### sqoop

- Export
  + Note: escape special characters like @ with backslash
  + Note: configure sql server firewall to allow connection from azure service
  + Note: create the table first in sql database before export
```
sqoop export --connect "jdbc:sqlserver://<sql_server_name>.database.windows.net;username=<sql_server_user_name>@<sql_server_name>;password=<password>;database=<sql_db_name>" --table <table_name> --export-dir /data/iislogs/summarized --input-fields-terminated-by \\t
```
