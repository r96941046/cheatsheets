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

### Dump & Restore
```psql
pg_dump dbname
psql dbname < infile

pg_dump dbname | gzip > filename.gz
gunzip -c filename.gz | psql dbname

pg_dump -h localhost -U localuser dbname | psql -h remotehost -U remoteuser dbname
pg_dump dbname | bzip2 | ssh remoteuser@remotehost "bunzip2 | psql dbname"
```

### Snippets

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
