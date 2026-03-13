
# SFTP → Microsoft Fabric Lakehouse Hybrid Copy Notebook
## Parallel Copy • Structured Logging • Bandwidth Limiter • Azure Key Vault

This notebook ingests files from an **SFTP server** into a **Microsoft Fabric Lakehouse**, mirroring the directory tree under a target `/Files` folder.

## Features
- Hybrid copy engine (buffered + streaming)
- Parallel execution with retries
- Structured logging (JSON or text)
- Global bandwidth limiter
- Azure Key Vault integration (Fabric connection or SDK)
- Dry-run mode
- Optional verification step

## How to Use
### 1. Upload in Fabric
Workspace → Notebook → Upload → select the `.ipynb` file.

### 2. Attach a Lakehouse
Enables `/lakehouse/default/Files/...` paths.

### 3. Configure (Cell 4)
Set values for:
- SFTP (`host`, `port`, `username`, `password` or `key_path`)
- Paths (`sftp_root`, `lakehouse_base`)
- Filters (`include_exts`)
- Hybrid thresholds
- Parallelism
- Logging
- Bandwidth limiter
- Optional Key Vault secrets

### 4. Dry Run
Set:
```
dry_run = True
```
Then run all cells.

### 5. Execute Copy
Set:
```
dry_run = False
```
Run Cell 4.

### 6. Logs
If enabled:
```
/lakehouse/default/Files/logs/sftp_hybrid_<run_id>.log
```

### 7. Verification
Run Cell 5 to count ingested files.

## Key Vault
Supports:
- Password
- Private key PEM
- Optional passphrase

Modes:
- Fabric Connection
- Azure SDK

## Notes
- Begin with dry-run
- Tune workers and chunk size
- Never logs secret values

