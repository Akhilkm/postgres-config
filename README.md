## Install psql client local

```
sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
apt-get update
apt install postgresql-client-15
```

### Master

1. cd /opt/postgresql
2. Update IP in the last line with replica server ip
3. docker compose up -d
4. Create user
   ```
   CREATE USER replicator REPLICATION LOGIN ENCRYPTED PASSWORD 'password';
   ```

### Replica

1. cd /opt/postgresql
2. docker run -it -v $PWD/data:/data postgres:15 pg_basebackup -h <master_ip> -p <master_port> -U replicator -X stream -Fp -Xs -P -R -D ./data
3. Update IP in the last line with master server ip
4. docker compose up -d

### Testing in master

```
\x
CREATE TABLE test (id serial PRIMARY KEY, data text);
INSERT INTO test (data) VALUES ('test data');
select * from test;

select * from pg_stat_replication;
```

Testing in Replica

```
\x
select * from test;

select * from pg_stat_wal_receiver;
```

Failover


### Backup 
```
pg_dump -h 192.168.102.247 -p 5432 -U postgres -Fc -b -v -f staging_dump.sql -d wfms_db
```

### Restore
```
create database wfms_db
pg_restore -v -h 192.168.102.249 -p 5438 -U postgres -d wfms_db -j 2 staging_dump.sql
```
