# Postgresql Cheat Sheet

### Reference

### Version
```psql
select version();
```
### List all databases
```psql
select * from pg_database;
```

### Connection
```psql
psql -U username -h hostname dbname
```

### Run SQL From File
```psql
psql -f "filename.psql"
```

### See table details
```psql
\d table_name
```

### Dump & Restore
```psql
pg_dump dbname
psql dbname < infile

pg_dump dbname | gzip > filename.gz
gunzip -c filename.gz | psql dbname

pg_dump -h localhost -U localuser dbname | psql -h remotehost -U remoteuser dbname
pg_dump dbname | bzip2 | ssh remoteuser@remotehost "bunzip2 | psql dbname"
```

### Drop DB From Command Line
check db connections first
```psql
select * from pg_stat_activity where datname = 'dbname';
```

as superuser or the database owner
```shell
dropdb 'dbname'
```

### Snippets

#### Indexes
```psql
create index pm_alert_created_at_patient_idx_category on pm_alert(created_at, patient_id, category);
drop index pm_alert_created_at_patient_idx_category;
```

#### Use alias to avoid ambiguous columns

Both api_log and user_meter have column user_id
```psql
select distinct al.user_id, al.user_agent from api_log al join user_meter on al.user_id = user_meter.user_id where user_meter.last_date_time >= now() + interval '8.5 hours' order by al.user_id;
```

Both care_connect and user_meter have column user_id
```psql
select distinct clinic_id, cc.user_id, last_date_time from care_connect cc join user_meter on cc.user_id = user_meter.user_id where cc.role = 'user' and cc.status = 'active' and user_meter.last_date_time >= now() + interval '8.5 hours' and cc.clinic_id not in (1) order by clinic_id;
```

#### String search
```psql
select distinct user_id from api_log where user_agent like '%iOS%' order by user_id;
select distinct user_id from api_log where user_agent like '%Android%' order by user_id;
```

#### Compare datetime
```psql
select * from diary where user_id = 661 and recorded_at < '2014-01-01' order by recorded_at desc;
select * from user_meter where last_date_time >= now() and last_date_time <= now() + interval '1 day';
select * from user_meter where last_date_time >= now() + interval '8.5 hours';
```

#### Add time to datetime
```psql
with
    rec as (
        select d.id, d.recorded_at
        from diary d
    )
update diary
set recorded_at = rec.recorded_at + interval '1 year'
from rec
where diary.id = rec.id and user_id = 8073;
```

#### Time casts
```psql
select * from diary where state = 'after_meal' and meal_type = 'snacks' and recorded_at::time <= '05:00:00' and recorded_at::time >= '00:00:00';
```

#### Subquery
```psql
select distinct clinic_id from care_connect where role = 'user' and status = 'active' and user_id in (select user_id from user_meter where last_date_time >= now() + interval '8 hours');
```

#### Replication

On master (the cat one shouldn't be at the bottom of the file)
```shell
sudo -u postgres psql -c "CREATE USER rep REPLICATION LOGIN CONNECTION LIMIT 1 ENCRYPTED PASSWORD 'rep';"

cat >> /etc/postgresql/9.4/main/pg_hba.conf <<EOF
host   replication   rep   IP_address_of_slave/32   md5
EOF

cat >> /etc/postgresql/9.4/main/postgresql.conf <<EOF
listen_addresses = 'localhost,IP_address_of_THIS_host'
wal_level = 'hot_standby'
archive_mode = on
archive_command = 'cd .'
max_wal_senders = 1
hot_standby = on
EOF

service postgresql restart
```

On slave
```shell
service postgresql stop

cat >> /etc/postgresql/9.4/main/pg_hba.conf <<EOF
host   replication   rep   IP_address_of_master/32   md5
EOF

cat >> /etc/postgresql/9.4/main/postgresql.conf <<EOF
listen_addresses = 'localhost,IP_address_of_THIS_host'
wal_level = 'hot_standby'
archive_mode = on
archive_command = 'cd .'
max_wal_senders = 1
hot_standby = on
EOF
```
