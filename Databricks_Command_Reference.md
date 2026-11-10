# Databricks Command Reference

Format per entry follows: purpose, syntax, return value, permissions, common errors, performance/security, cleanup. Executable examples are in `Complete_Databricks_Zero_to_Expert.ipynb`. Written from training knowledge, not re-verified against live docs; verify preview features before production use.

## Index
- SHOW CATALOGS / SCHEMAS / TABLES / VOLUMES
- DESCRIBE TABLE [EXTENDED]
- DESCRIBE DETAIL / HISTORY
- CREATE CATALOG / SCHEMA
- CREATE VOLUME
- CREATE TABLE / CTAS / CREATE OR REPLACE
- INSERT INTO / OVERWRITE
- UPDATE / DELETE
- MERGE INTO
- RESTORE TABLE
- Time travel
- OPTIMIZE
- VACUUM
- CLONE
- ALTER TABLE
- Change Data Feed
- read_files()
- COPY INTO
- Auto Loader
- Structured Streaming
- DataFrame joins
- Window functions
- explode family
- GRANT / REVOKE / DENY / SHOW GRANTS
- Row filter / column mask
- ABAC policies
- dbutils.fs / secrets / widgets / notebook
- Databricks CLI
- Bundles (Declarative Automation Bundles)
- Jobs API / SDK
- MLflow


## SHOW CATALOGS / SCHEMAS / TABLES / VOLUMES
- **Purpose:** List UC objects
- **Syntax:** `SHOW SCHEMAS IN cat; SHOW TABLES IN cat.sch; SHOW VOLUMES IN cat.sch`
- **Return value:** Result set
- **Prerequisites/permissions:** USE CATALOG/USE SCHEMA; lists only what you can see
- **Common errors:** Catalog not found
- **Performance/security:** Metadata only, cheap
- **Cleanup:** n/a

## DESCRIBE TABLE [EXTENDED]
- **Purpose:** Columns, type MANAGED/EXTERNAL, location, owner
- **Syntax:** `DESCRIBE TABLE EXTENDED cat.sch.t`
- **Return value:** Result set
- **Prerequisites/permissions:** SELECT or ownership
- **Common errors:** Table not found
- **Performance/security:** Metadata only
- **Cleanup:** n/a

## DESCRIBE DETAIL / HISTORY
- **Purpose:** Delta file stats; transaction history
- **Syntax:** `DESCRIBE DETAIL t; DESCRIBE HISTORY t LIMIT n`
- **Return value:** Result set
- **Prerequisites/permissions:** SELECT
- **Common errors:** Not a Delta table
- **Performance/security:** History limited by log retention (30d default)
- **Cleanup:** n/a

## CREATE CATALOG / SCHEMA
- **Purpose:** Create UC containers
- **Syntax:** `CREATE CATALOG IF NOT EXISTS c [MANAGED LOCATION 'path']; CREATE SCHEMA IF NOT EXISTS c.s`
- **Return value:** None
- **Prerequisites/permissions:** CREATE CATALOG on metastore; CREATE SCHEMA on catalog
- **Common errors:** No default storage: add MANAGED LOCATION
- **Performance/security:** Separate storage per catalog for isolation
- **Cleanup:** DROP SCHEMA c.s CASCADE; DROP CATALOG c CASCADE

## CREATE VOLUME
- **Purpose:** Governed file storage
- **Syntax:** `CREATE VOLUME IF NOT EXISTS c.s.v; CREATE EXTERNAL VOLUME ... LOCATION 'path'`
- **Return value:** None
- **Prerequisites/permissions:** CREATE VOLUME on schema
- **Common errors:** Path under /Volumes/c/s/v/
- **Performance/security:** Use instead of mounts
- **Cleanup:** DROP VOLUME c.s.v

## CREATE TABLE / CTAS / CREATE OR REPLACE
- **Purpose:** Create Delta tables
- **Syntax:** `CREATE TABLE t (cols) [CLUSTER BY (c)] ; CREATE OR REPLACE TABLE t AS SELECT ...`
- **Return value:** None
- **Prerequisites/permissions:** CREATE TABLE on schema
- **Common errors:** REPLACE keeps history; DROP+CREATE does not
- **Performance/security:** CLUSTER BY for query keys
- **Cleanup:** DROP TABLE t

## INSERT INTO / OVERWRITE
- **Purpose:** Append or replace rows
- **Syntax:** `INSERT INTO t SELECT ...; INSERT OVERWRITE t SELECT ...`
- **Return value:** Num rows
- **Prerequisites/permissions:** MODIFY
- **Common errors:** Schema mismatch
- **Performance/security:** Batch writes, avoid tiny inserts
- **Cleanup:** n/a

## UPDATE / DELETE
- **Purpose:** Row changes (Delta)
- **Syntax:** `UPDATE t SET c=v WHERE p; DELETE FROM t WHERE p`
- **Return value:** Num rows
- **Prerequisites/permissions:** MODIFY
- **Common errors:** Constraint violations fail whole statement
- **Performance/security:** Deletion vectors reduce rewrite cost
- **Cleanup:** RESTORE if mistaken

## MERGE INTO
- **Purpose:** Upsert/CDC
- **Syntax:** `MERGE INTO t USING s ON cond WHEN MATCHED [AND c] THEN UPDATE SET * | DELETE WHEN NOT MATCHED THEN INSERT * [WHEN NOT MATCHED BY SOURCE THEN DELETE]`
- **Return value:** Row counts
- **Prerequisites/permissions:** MODIFY on target, SELECT on source
- **Common errors:** Multiple source rows match one target row
- **Performance/security:** Dedupe source; include partition/cluster predicate
- **Cleanup:** n/a

## RESTORE TABLE
- **Purpose:** Roll back Delta table
- **Syntax:** `RESTORE TABLE t TO VERSION AS OF n | TIMESTAMP AS OF 'ts'`
- **Return value:** Metrics
- **Prerequisites/permissions:** MODIFY
- **Common errors:** Files already vacuumed
- **Performance/security:** Creates a new commit
- **Cleanup:** n/a

## Time travel
- **Purpose:** Read old versions
- **Syntax:** `SELECT * FROM t VERSION AS OF n; ... TIMESTAMP AS OF 'ts'; spark.read.option('versionAsOf',n).table(t)`
- **Return value:** DataFrame
- **Prerequisites/permissions:** SELECT
- **Common errors:** Version/files removed
- **Performance/security:** Bounded by VACUUM and log retention
- **Cleanup:** n/a

## OPTIMIZE
- **Purpose:** Compact files; cluster
- **Syntax:** `OPTIMIZE t [WHERE p] [ZORDER BY (c)]`
- **Return value:** Metrics
- **Prerequisites/permissions:** MODIFY
- **Common errors:** ZORDER not allowed on liquid-clustered tables
- **Performance/security:** Run on schedule or enable predictive optimization
- **Cleanup:** n/a

## VACUUM
- **Purpose:** Delete unreferenced files
- **Syntax:** `VACUUM t [RETAIN n HOURS] [DRY RUN]`
- **Return value:** File list
- **Prerequisites/permissions:** MODIFY
- **Common errors:** Retention below 7 days is blocked by default
- **Performance/security:** Breaks time travel/shallow clones beyond retention
- **Cleanup:** n/a

## CLONE
- **Purpose:** Copy table
- **Syntax:** `CREATE TABLE x DEEP|SHALLOW CLONE y`
- **Return value:** None
- **Prerequisites/permissions:** SELECT on source, CREATE TABLE
- **Common errors:** Shallow clone breaks if source files are vacuumed
- **Performance/security:** Deep copies data
- **Cleanup:** DROP TABLE x

## ALTER TABLE
- **Purpose:** Modify structure/properties
- **Syntax:** `ADD COLUMNS; RENAME COLUMN (needs column mapping); SET TBLPROPERTIES; ADD CONSTRAINT CHECK; CLUSTER BY; SET ROW FILTER; ALTER COLUMN SET MASK; SET OWNER TO; SET TAGS`
- **Return value:** None
- **Prerequisites/permissions:** MODIFY or ownership
- **Common errors:** RENAME/DROP COLUMN without column mapping
- **Performance/security:** Property changes may upgrade protocol (irreversible-ish)
- **Cleanup:** n/a

## Change Data Feed
- **Purpose:** Row-level change stream
- **Syntax:** `TBLPROPERTIES (delta.enableChangeDataFeed=true); table_changes('t', start[, end]); readChangeFeed=true`
- **Return value:** Rows with _change_type
- **Prerequisites/permissions:** SELECT
- **Common errors:** Only changes after enablement
- **Performance/security:** Extra storage
- **Cleanup:** Disable property

## read_files()
- **Purpose:** SQL file reader (batch/streaming)
- **Syntax:** `SELECT * FROM read_files('path', format => 'csv', header => true, schema => 'a INT')`
- **Return value:** Table
- **Prerequisites/permissions:** READ VOLUME / external location access
- **Common errors:** Schema inference needs sample files
- **Performance/security:** Wrap with STREAM(...) in streaming tables
- **Cleanup:** n/a

## COPY INTO
- **Purpose:** Idempotent incremental file load
- **Syntax:** `COPY INTO t FROM 'path' FILEFORMAT=CSV [FILES=(...)|PATTERN='*.csv'] FORMAT_OPTIONS(...) COPY_OPTIONS('mergeSchema'='true','force'='false') ; add VALIDATE ALL for dry run`
- **Return value:** num_affected_rows, num_inserted_rows, num_skipped_correct_files
- **Prerequisites/permissions:** MODIFY on table, READ on path
- **Common errors:** Target table must exist; force reloads files
- **Performance/security:** Best for thousands, not millions, of files
- **Cleanup:** n/a

## Auto Loader
- **Purpose:** Incremental file ingestion
- **Syntax:** `spark.readStream.format('cloudFiles').option('cloudFiles.format','json').option('cloudFiles.schemaLocation',p).load(dir).writeStream.option('checkpointLocation',c).trigger(availableNow=True).toTable(t)`
- **Return value:** StreamingQuery
- **Prerequisites/permissions:** READ on path, MODIFY on table; notification mode needs cloud perms
- **Common errors:** Schema change stops stream once (addNewColumns)
- **Performance/security:** Notification mode for very large dirs
- **Cleanup:** Delete checkpoint+schema dirs to reset

## Structured Streaming
- **Purpose:** Incremental processing
- **Syntax:** `readStream / writeStream / trigger(processingTime|availableNow) / outputMode / withWatermark / foreachBatch / toTable`
- **Return value:** StreamingQuery
- **Prerequisites/permissions:** Source/sink access
- **Common errors:** Changing query with old checkpoint; unbounded state without watermark
- **Performance/security:** Monitor lastProgress
- **Cleanup:** query.stop(); remove checkpoint

## DataFrame joins
- **Purpose:** Combine data
- **Syntax:** `df.join(other, on, how) with inner,left,right,full,cross,left_semi,left_anti; F.broadcast(df)`
- **Return value:** DataFrame
- **Prerequisites/permissions:** SELECT
- **Common errors:** Ambiguous columns after join; accidental cross join
- **Performance/security:** Broadcast small side; AQE
- **Cleanup:** n/a

## Window functions
- **Purpose:** Ranking/running calcs
- **Syntax:** `F.row_number().over(Window.partitionBy(a).orderBy(b)); lag, lead, rank, dense_rank, ntile; rowsBetween`
- **Return value:** Column
- **Prerequisites/permissions:** n/a
- **Common errors:** Missing orderBy for ranking
- **Performance/security:** Wide transformation (shuffle)
- **Cleanup:** n/a

## explode family
- **Purpose:** Flatten arrays
- **Syntax:** `F.explode, explode_outer, posexplode; SQL: LATERAL VIEW explode / inline`
- **Return value:** DataFrame
- **Prerequisites/permissions:** n/a
- **Common errors:** explode drops null/empty arrays; use _outer
- **Performance/security:** Can multiply rows
- **Cleanup:** n/a

## GRANT / REVOKE / DENY / SHOW GRANTS
- **Purpose:** Access control
- **Syntax:** `GRANT priv ON securable TO principal; REVOKE ...; DENY ... (where supported); SHOW GRANTS ON ...`
- **Return value:** None
- **Prerequisites/permissions:** Owner or MANAGE
- **Common errors:** Missing USE CATALOG/USE SCHEMA
- **Performance/security:** Grant to groups, not users
- **Cleanup:** REVOKE

## Row filter / column mask
- **Purpose:** Fine-grained access
- **Syntax:** `ALTER TABLE t SET ROW FILTER fn ON (col); ALTER TABLE t ALTER COLUMN c SET MASK fn; DROP ROW FILTER; DROP MASK`
- **Return value:** None
- **Prerequisites/permissions:** Owner/MANAGE, EXECUTE on function
- **Common errors:** Function must be deterministic SQL/UDF returning BOOLEAN/column type
- **Performance/security:** Filters add per-query cost
- **Cleanup:** DROP ROW FILTER / DROP MASK

## ABAC policies
- **Purpose:** Tag-driven access
- **Syntax:** `Governed tags + policies (CREATE POLICY ...) [verify syntax, preview in some regions]`
- **Return value:** None
- **Prerequisites/permissions:** MANAGE on securable, tag assignment rights
- **Common errors:** Feature not enabled
- **Performance/security:** Central governance at scale
- **Cleanup:** DROP POLICY

## dbutils.fs / secrets / widgets / notebook
- **Purpose:** Notebook utilities
- **Syntax:** `dbutils.fs.ls(path); dbutils.secrets.get(scope,key); dbutils.widgets.text(n,default,label); dbutils.notebook.run(path,timeout,args)`
- **Return value:** varies
- **Prerequisites/permissions:** Volume/ secret ACL permissions
- **Common errors:** Mounts unsupported with UC; secrets redacted in output
- **Performance/security:** Never print secrets
- **Cleanup:** dbutils.fs.rm(p,True)

## Databricks CLI
- **Purpose:** Terminal automation
- **Syntax:** `databricks auth login|profiles; jobs|clusters|warehouses|catalogs|secrets|fs|repos|bundle ...; databricks api <verb> <path>`
- **Return value:** JSON
- **Prerequisites/permissions:** Per API permissions
- **Common errors:** Wrong profile/host
- **Performance/security:** Use OAuth
- **Cleanup:** n/a

## Bundles (Declarative Automation Bundles)
- **Purpose:** Deploy code+resources
- **Syntax:** `databricks bundle init|validate|deploy -t T|run job -t T|summary|generate|destroy`
- **Return value:** Deploy summary
- **Prerequisites/permissions:** Workspace permissions
- **Common errors:** Variable/target mismatch
- **Performance/security:** dev mode prefixes resources; prod uses run_as service principal
- **Cleanup:** bundle destroy -t dev

## Jobs API / SDK
- **Purpose:** Programmatic jobs
- **Syntax:** `POST /api/2.2/jobs/create|run-now|repair; w.jobs.create(...)`
- **Return value:** JSON
- **Prerequisites/permissions:** CAN_MANAGE on job
- **Common errors:** Invalid task keys/ cluster spec
- **Performance/security:** Retries, timeouts, notifications
- **Cleanup:** jobs delete

## MLflow
- **Purpose:** Tracking and registry
- **Syntax:** `mlflow.set_experiment; start_run; log_param/metric/model; mlflow.set_registry_uri('databricks-uc'); register_model`
- **Return value:** Run/model
- **Prerequisites/permissions:** Experiment and UC model permissions
- **Common errors:** Registry URI not set to UC
- **Performance/security:** Track data/model lineage
- **Cleanup:** mlflow.delete_run

## Deprecated to modern
- dbutils.library to %pip
- DBFS mounts to volumes/external locations
- Delta Live Tables to Lakeflow Spark Declarative Pipelines
- APPLY CHANGES INTO to AUTO CDC INTO
- trigger(once=True) to trigger(availableNow=True)
- Repos to Git folders; Workflows to Lakeflow Jobs; Asset Bundles to Declarative Automation Bundles

## Not covered exhaustively
This is not every Databricks command. Missing detail: full option lists for read_files/cloudFiles, every SQL privilege, full REST API surface, Model Serving/Vector Search APIs. Use the official docs for those.