# Databricks Zero-to-Expert Package

## Files
| File | Purpose |
|---|---|
| `Databricks_Environment_Setup.ipynb` | Run first. Creates catalog `databricks_learning`, schemas, volume, sample tables. Idempotent. |
| `Complete_Databricks_Zero_to_Expert.ipynb` | Main notebook: Modules 1-16, appendices, final validation. |
| `Databricks_Command_Reference.md` | Searchable command reference. |
| `Databricks_Compatibility_Matrix.md` | Runtime, cloud and compute compatibility. |
| `README.md` | This file. |

## Import and run
1. Workspace > Import > upload both `.ipynb` files.
2. Attach to serverless or Unity Catalog compute (Runtime 13.3 LTS+).
3. Run the Setup notebook. If a step fails, it prints the SQL an administrator must run; it never grants permissions to itself.
4. Run the main notebook top to bottom. Each risky cell is guarded, so unsupported features show as SKIPPED or FAILED in the final summary and the run continues.

## Needs your input
- Cells with `<placeholders>` (job spec, bundle YAML, JDBC) are examples; replace the values before use.
- Grants use a placeholder group `analysts` and are commented out.
- Secret scopes, warehouses, service principals, storage credentials, external locations: CLI/UI/admin (see Setup section 10).

## Limits (read this)
- Not run on a live workspace by me; code cells pass a Python syntax check only. Expect to fix small issues depending on runtime, cloud and permissions.
- Written from training knowledge without live doc verification; items tagged [verify] are preview or fast-changing.
- It is broad, not exhaustive: not every Databricks command or option is covered.

## Cleanup
Set `CONFIRM_CLEANUP = True` in the last cell of the Setup notebook to drop catalog `databricks_learning` (guarded to that name only).
