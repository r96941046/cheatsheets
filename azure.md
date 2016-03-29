## Azure Cheatsheet

### Azure CLI

- [MS Azure CLI Guide](https://azure.microsoft.com/zh-tw/documentation/articles/virtual-machines-command-line-tools/)

First switch to the correct npm environment

Help
```
azure help
azure help config
azure help account list
```

Set to azure resource manager mode
```
azure config mode arm
```

Login
```
azure login
```

Info
```
azure resource list
azure account list
```

Set subscription to use
```
azure account set <subscription-id>
```

Connect to storage account
```
azure storage account connectionstring show <storage-name> -g <resource-group-name-for-the-storage>
export AZURE_STORAGE_CONNECTION_STRING="<the-connection-string>"
azure storage blob upload test.txt <container-name> example/test.txt
azure storage blob download <container-name> <blob-name> <target-folder-name>
```

### HDInsight

Shared file system commands

List
```
hdfs dfs -ls /
hdfs dfs -ls /example
```

Print
```
hdfs dfs -text /example/data/test.txt
hdfs dfs -cat
```

File
```
hdfs dfs -rm
hdfs dfs -cp
hdfs dfs -mv
```

Common task jars
```
ls /usr/hdp/current/hadoop-mapreduce-client
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount
hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount <source-data-on-shared-file-system> <result-path-on-shared-file-system>
```

### HDInsight - Hive

First ssh into HDInsight head node

Enter Hive shell
```
hive
```

Leave Hive shell
```
EXIT;
```

Info
```
SHOW TABLES;
```

Default table location
```
hdfs dfs -ls /hive/warehouse
```

Select
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

Select from views
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

Create views
```
CREATE VIEW v_system_log
AS
SELECT from_unixtime(unix_timestamp(datetime, 'dd/MM/yyyy hh:mm:ss')) AS datetime, level, event_id, source, details
FROM system_log;
```

Create external table
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

Create internal table
```
CREATE TABLE system_log
(level STRING,
 datetime STRING,
 source STRING,
 event_id STRING,
 category STRING,
 details STRING);
```

Create table as select
```
CREATE TABLE error_log
ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE LOCATION '/data/error_log'
AS
SELECT datetime, source, event_id, details
FROM system_log
WHERE level = 'Error';
```

Load data into table
```
LOAD DATA INPATH '/data/logs.txt' INTO TABLE staged_log;
```

Insert data from select
```
INSERT INTO TABLE system_log
SELECT * FROM staged_log
WHERE event_id IS NOT NULL;
```

Partition
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

Run hive script
```
hive -i wasb:///data/create_table.hql
```

UDF with python (stdin/stdout)
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

Use UDF in hive script
```
add file wasb:///data/convert_rain.py;

SELECT TRANSFORM (year, month, max_temp, min_temp, frost_days, rainfall, sunshine_hours)
  USING 'python convert_rain.py' AS
  (year INT, month INT, max_temp FLOAT, min_temp FLOAT, frost_days INT, rainfall FLOAT, sunshine_hours FLOAT)
FROM weather;
```

### HDInsight - Pig

Enter grunt shell
```
pig
```

Leave grunt shell
```
QUIT;
```

Load
```
Source = LOAD '/data/heathrow.txt' USING PigStorage('\t') AS (year:chararray, month:chararray, maxtemp:float, mintemp:float, frost:int, rainfall:float, sunshine:float);
```

Filter (remove null values from schema shaping)
```
Data = FILTER Source BY maxtemp IS NOT NULL AND mintemp IS NOT NULL;
Readings = FILTER Data BY year != 'yyyy';
CleanReadings = FILTER DataVals BY INDEXOF(sunshinehours, '#', 0) <= 0;
DirtyReadings = FILTER DataVals BY INDEXOF(sunshinehours, '#', 0) > 0;
```

Dump (to see results on-the-fly)
```
DUMP Data;
DUMP SortedResults;
```

Save
```
STORE SortedReadings INTO '/data/scrubbedweather' USING PigStorage(' ');
```

Group
```
YearGroups = GROUP Readings BY year;
```

Foreach
```
AggTemps = FOREACH YearGroups GENERATE group AS year, AVG(Readings.maxtemp) AS avghigh, AVG(Readings.mintemp) AS avglow;
CleanedReadings = FOREACH DirtyReadings GENERATE year, month, maxtemp, mintemp, frostdays, rainfall, SUBSTRING(sunshinehours, 0, INDEXOF(sunshinehours, '#', 0)) AS sunshinehours;
```

Sort
```
SortedResults = ORDER AggTemps BY year;
```

Replace
```
DataVals = FOREACH Data GENERATE year, month, maxtemp, mintemp, frostdays, rainfall, REPLACE(sunshinehours, '---', '') AS sunshinehours;
```

Union
```
Readings = UNION CleanReadings, CleanedReadings;
```

Run pig scripts
```
pig wasb:///data/scrub_weather.pig
```

UDF with python
```
@outputSchema("f_readings: {(year:chararray, month:int, maxtemp:float, mintemp:float, frostdays:int, rainfall:float, sunshinehours:chararray)}")
def fahrenheit(c_reading):
  year, month, maxtemp, mintemp, frostdays, rainfall, sunshine = c_reading.split(' ')
  maxtemp_f = float(maxtemp) * 9/5 + 32
  mintemp_f = float(mintemp) * 9/5 + 32
  return year, int(month), maxtemp_f, mintemp_f, frostdays, float(rainfall), sunshine
```

Use UDF in pig script
```
REGISTER 'wasb:///data/convert_temp.py' using jython as convert_temp;

Source = LOAD '/data/scrubbedweather' AS (celsius_readings:chararray);

ConvertedReadings = FOREACH Source GENERATE FLATTEN(convert_temp.fahrenheit(celsius_readings));

STORE ConvertedReadings INTO '/data/convertedweather';
```

### PowerShell Getting started

- [How to install and configure Azure PowerShell](https://azure.microsoft.com/en-us/documentation/articles/powershell-install-configure/)

Open PowerShell ISE with admin
```
Install-Module AzureRM
Install-AzureRM
Install-Module Azure
```

Enable running scripts on the system
```
Set-ExecutionPolicy UnRestricted
```

Import and update the module to install necessary functions
```
Import-Module Azure
Import-Module AzureRM
Update-AzureRM
```

Login resource manager
```
Import-AzureRM
Login-AzureRmAccount
```

### Azure site-to-site VPN specs

- [About VPN devices for Site-to-Site VPN Gateway connections](https://azure.microsoft.com/en-us/documentation/articles/vpn-gateway-about-vpn-devices/)

### Azure site-to-site VPN

- [7 Steps to Building Site-to-Site VPN Connections for V2 VNETs using Azure Resource Manager in the NEW Azure Portal](https://blogs.technet.microsoft.com/keithmayer/2015/12/22/7-steps-to-building-site-to-site-vpn-connections-for-v2-vnets-using-azure-resource-manager-in-the-new-azure-portal/)

### Use PowerShell to dump site-to-site VPN logs to storage account

- [Step-by-Step: Capturing Azure Resource Manager (ARM) VNET Gateway Diagnostic Logs](https://blogs.technet.microsoft.com/keithmayer/2015/12/07/step-by-step-capturing-azure-resource-manager-arm-vnet-gateway-diagnostic-logs/)
- [Diagnose Azure Virtual Network VPN connectivity issues with PowerShell](https://blogs.technet.microsoft.com/keithmayer/2014/12/18/diagnose-azure-virtual-network-vpn-connectivity-issues-with-powershell/)

### VPN Routers

- [Draytek Vigor IPsec VPN configuration](http://maxding.blogspot.tw/2014/07/draytek-vigor-2920n-ipsec-vpn.html)
- [Draytek syslog access](https://www.draytek.com/index.php?option=com_k2&view=item&id=2062&Itemid=293&lang=en)
