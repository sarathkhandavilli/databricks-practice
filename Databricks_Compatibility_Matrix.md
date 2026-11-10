# Databricks Compatibility Matrix

Based on training knowledge, not live docs. Entries marked [verify] are preview or fast-changing; confirm in release notes.

## Compute vs feature
| Feature | Serverless notebook/jobs | Classic dedicated | Classic standard (shared) | SQL warehouse |
|---|---|---|---|---|
| Python/PySpark DataFrames | Yes | Yes | Yes | No (SQL only) |
| SQL | Yes | Yes | Yes | Yes |
| Scala / R | Limited [verify] | Yes | Scala yes, R limited [verify] | No |
| SparkContext / RDD / broadcast vars / accumulators | No | Yes | No | No |
| Arbitrary Spark conf changes | Small allow-list | Yes | Mostly | Limited |
| Init scripts / custom cluster libs | No (use environment) | Yes | Limited | No |
| Unity Catalog | Yes | Yes | Yes | Yes |
| Streaming (availableNow) | Yes | Yes | Yes | Streaming tables/MVs only |
| Photon | Built in | Selectable | Selectable | Built in (Pro/Serverless) |

## Feature vs minimum runtime (approximate)
| Feature | Minimum | Notes |
|---|---|---|
| Unity Catalog volumes | DBR 13.3 LTS | |
| Liquid clustering (CLUSTER BY) | DBR 13.3 LTS+ [verify] | Replaces partitioning/ZORDER |
| Deletion vectors | DBR 12.2 LTS+ | Default on newer runtimes for some tables [verify] |
| read_files() | DBR 13.3+/DBSQL | |
| Row filters / column masks | DBR 12.2 LTS+ [verify] | |
| Identity columns | DBR 10.4+ [verify] | |
| DataFrame.unpivot | Spark 3.4 (DBR 13+) | |
| Lakeflow Spark Declarative Pipelines | Pipeline runtime | Not runnable in a plain notebook |
| ABAC policies | Preview [verify] | |

## Cloud differences
| Area | AWS | Azure | GCP |
|---|---|---|---|
| Storage URI | s3:// | abfss:// | gs:// |
| Storage credential | IAM role | Managed identity / service principal | Service account |
| File notification (Auto Loader) | SNS + SQS | Event Grid + Queue Storage | Pub/Sub |
| Account console | accounts.cloud.databricks.com | accounts.azuredatabricks.net | accounts.gcp.databricks.com |
| Private connectivity | PrivateLink | Private Link | Private Service Connect |
| Instance types | EC2 | Azure VM | GCE |
| Feature availability | Broadest | Broad | Some features lag [verify] |

## Needs admin, CLI, UI or another product
| Item | Where |
|---|---|
| Metastore, workspace assignment | Account console |
| Storage credentials, external locations | UI/SQL with cloud IAM set up |
| Secret scopes | CLI/API |
| SQL warehouses, service principals | UI/CLI/API (account rights for SPs) |
| Lakeflow Connect | UI/API |
| Model Serving, Vector Search, Genie, AutoML, Agent Framework | Separate enablement, region-dependent |
| System tables | Metastore admin enables schemas |
| Delta Sharing | Sharing enabled on metastore; CREATE SHARE/RECIPIENT |
