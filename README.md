# CockroachDB + PGWeb

This repository contains files to run CockroachDB (CRDB).

`start.sh` - Starts cockroachdb

`start_web_admin.sh` - Starts PGWeb

`docker-compose-single.yml` - runs both, with persistent volume in host's ./data folder. CockroachDB is single node in this case.

`docker-compose.yml` - runs cockroachdb 3 node cluster, persistent volume in /data, /data2
and /data3 directories.

> This documentation assumes that the CRDB cluster docker-compose is used.

> Remember to delete the /data directory & use `--remove-orphans` if you are changing from single node to clustered.

## Usage

> This documentation assumes that modifications are being done on the "AuditLog" table in the "postgres" database.

These are the basic commands:

- Perform SQL Queries to roach1 node (denoted as `SQL>`)

  `$ docker exec -it cockroachdb_roach1_1 ./cockroach sql --insecure --database=postgres`

- Run bash in the container (denoted as `#`)

  `$ docker exec -it cockroachdb_roach1_1 /bin/bash`

### Configuration

#### Garbage Collection

By default, gc.ttlseconds is 25hours (90000).

1. `SQL> SHOW ZONE CONFIGURATIONS;` to see current zone configs
2. `SQL> SHOW ZONE CONFIGURATION FOR DATABASE postgres;` to see configs for a specific database
3. `SQL> ALTER DATABASE postgres CONFIGURE ZONE USING gc.ttlseconds = 600;` to set to 10min TTL

Reference: https://www.cockroachlabs.com/docs/stable/configure-zone.html

### Logging

The raw log files can be found in `/data/logs/cockroach*.log`. It follows [this format](https://www.cockroachlabs.com/docs/stable/debug-and-error-logs.html#write-to-file).

#### Enable Per-node Logging Display in GUI

1. `SQL> SHOW ALL CLUSTER SETTINGS;`, check for `server.remote_debugging.mode`
2. `SQL> SET CLUSTER SETTING server.remote_debugging.mode="any";`
3. To check, `SQL> SHOW CLUSTER SETTING server.remote_debugging.mode;`
4. View logs by going to GUI -> Live Nodes -> Logs

#### Enable Audit Logging

https://www.cockroachlabs.com/docs/stable/sql-audit-logging.html

> Note: This is an experimental feature

1. `SQL> ALTER TABLE "AuditLog" EXPERIMENTAL_AUDIT SET READ WRITE;`
2. Do some select query
3. Check the audit log at `/data/logs/cockroach-sql-audit.log`

### Backup & Restore

#### Backup Data (Dump to local as .sql)

https://www.cockroachlabs.com/docs/v19.1/sql-dump.html#dump-a-tables-schema-and-data

1. `$ docker exec -it cockroachdb_roach1_1 ./cockroach dump postgres AuditLog --insecure > dump-file.sql`

#### Restore Data (Using .sql dump)

https://www.cockroachlabs.com/docs/v19.1/sql-dump.html#restore-a-table-from-a-backup-file

1. `$ docker container cp ~/dump-file.sql cockroachdb_roach1_1:/cockroach/dump-file.sql`
2. After ensuring that the table has been dropped, `# ./cockroach sql --insecure --database=postgres < dump-file.sql`

#### Backup Data (Dump as CSV format)

TODO

#### Restore Data (Using CSV dump)

TODO

### Deleting Data

After deleting data, garbage collection will be performed after the specified interval to reclaim space. This can be controlled with `gc.ttlseconds`.

> Do not confuse the "Compaction Status" logs as compaction occurring. These logs are set to appear every 10m (see CRDB src code storage/store.go).

#### Delete Records

1. `SQL> DELETE FROM "AuditLog" WHERE 1=1;`

   Need to use `WHERE 1=1` since `sql_safe_updates = true`.

#### Truncate Table

1. Navigate to PGWeb
2. Right click table and select `Truncate Table`

#### Drop Table

1. Navigate to PGWeb
2. Right click table and select `Drop Table`
