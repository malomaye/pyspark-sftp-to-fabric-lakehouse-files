
# SFTP → Microsoft Fabric Lakehouse Hybrid Copy Notebook
## Parallel Copy • Structured Logging • Bandwidth Limiter • Azure Key Vault

This notebook ingests files from an **SFTP server** into a **Microsoft Fabric Lakehouse**, mirroring the directory tree under a target `/Files` folder.

---

## Features
- Hybrid copy engine (buffered + streaming)
- Parallel execution with retries
- Structured logging (JSON or text)
- Global bandwidth limiter
- Azure Key Vault integration (Fabric connection or SDK)
- Dry-run mode
- Optional verification step

---

## Quick Start
Follow these steps to run your first sync in just a few minutes.

### 1) Upload & Attach Lakehouse
- In Fabric: **Workspace → Notebook → Upload** the `SFTP_to_Lakehouse_Hybrid.ipynb`.
- Click **Add Lakehouse** so paths like `/lakehouse/default/Files/...` are available.

### 2) Install Packages (Cell 2)
Run the cell that installs `paramiko` (and Azure SDK if you use Key Vault SDK mode).

### 3) Pick Your Auth Method
- **Password**: set `password = "..."` (or load from Key Vault).
- **SSH key**: set `key_path = "/path/to/privatekey.pem"` (or load from Key Vault) and optionally `key_passphrase`.

### 4) Edit the CONFIG (Cell 4)
Provide the minimum:
```python
host = "sftp.example.com"
username = "myuser"
sftp_root = "/export/data"
lakehouse_base = "/lakehouse/default/Files/sftp_sync"
include_exts = [".csv"]

# Start in preview mode
dry_run = True
```
Optional but recommended:
```python
max_workers = 8
large_file_threshold_mb = 1024
chunk_size_mb = 8

# Logging
log_to_file = True
log_format = "json"
log_dir = "/lakehouse/default/Files/logs"

# Bandwidth limiter
bandwidth_limit_mbps = 50
burst_capacity_seconds = 2.0
```

### 5) (Optional) Load Secrets from Key Vault
**Fabric Connection mode**:
```python
use_key_vault = True
kv_mode = "fabric_connection"
kv_connection_name = "kv-sftp"
kv_secret_password_name = "sftp-password"  # OR use PEM secrets
```
**SDK mode**:
```python
use_key_vault = True
kv_mode = "sdk"
kv_vault_url = "https://<your-vault>.vault.azure.net/"
kv_secret_password_name = "sftp-password"  # OR use PEM secrets
```

### 6) Run in Preview (Dry Run)
Run Cell 4. Verify the summary (files seen / would copy / would skip).

### 7) Execute for Real
Flip:
```python
dry_run = False
```
Run Cell 4 again. Monitor output and (if enabled) the log file under `/lakehouse/default/Files/logs/`.

### 8) Verify
Run the optional verification cell to count ingested files.

---

## Sample Config File
If you prefer keeping settings in a separate file, use the JSON sample below as `sample_config.json`. You can paste it into your repo and copy/paste values into the CONFIG cell, or extend the notebook later to load it programmatically.

**`sample_config.json`**
```json
{
  "host": "sftp.example.com",
  "port": 22,
  "username": "myuser",
  "password": null,
  "key_path": null,
  "key_passphrase": null,

  "sftp_root": "/export/data",
  "lakehouse_base": "/lakehouse/default/Files/sftp_sync",
  "include_exts": [".csv", ".parquet"],

  "large_file_threshold_mb": 1024,
  "chunk_size_mb": 8,

  "max_workers": 8,
  "retries": 3,
  "retry_backoff_base": 1.5,

  "dry_run": true,
  "skip_if_same_size": true,
  "create_missing_dirs": true,

  "socket_timeout_sec": 60,
  "keepalive_secs": 30,

  "log_level": "INFO",
  "log_format": "json",
  "log_to_file": true,
  "log_dir": "/lakehouse/default/Files/logs",
  "log_max_bytes": 52428800,
  "log_backup_count": 2,

  "bandwidth_limit_mbps": 50,
  "burst_capacity_seconds": 2.0,
  "min_sleep_ms": 10,
  "force_streaming_under_limiter": true,

  "use_key_vault": true,
  "kv_mode": "fabric_connection",
  "kv_connection_name": "kv-sftp",
  "kv_vault_url": "https://<your-vault>.vault.azure.net/",
  "kv_secret_password_name": "sftp-password",
  "kv_secret_pem_name": null,
  "kv_secret_pem_passphrase_name": null
}
```

> **Tip**: Start with `dry_run = true`. For limiter smoothness, try `chunk_size_mb=2` when `bandwidth_limit_mbps` is low (e.g., 10–25 Mbps).

---

## Key Vault Notes
- **Fabric Connection** uses `mssparkutils.credentials.getSecret(<connection>, <secret>)`.
- **SDK mode** uses `DefaultAzureCredential` + `SecretClient` (ensure MI/SPN has `get` access to secrets).
- Secrets are **never logged**; PEM (if any) is written to `/tmp` with `chmod 600` by default.

---

## Troubleshooting (Short List)
| Issue | Likely Cause | Fix |
|---|---|---|
| Connection refused | Host/port or firewall | Verify DNS/port and allowlist Fabric egress |
| Auth failed | Bad password/PEM/passphrase | Verify secrets; test with an SFTP client |
| Many retries | Server throttling / limiter too strict | Bump `burst_capacity_seconds`; reduce `max_workers`; increase `socket_timeout_sec` |
| PEM errors | Wrong key format | Ensure raw PEM content; remove extra headers/footers |

---

## Extending
- Emit per-file metrics as CSV/Delta in Lakehouse
- Add checksum verification for idempotency
- Parameterize for Fabric Pipelines



---

## Flow Diagram
![SFTP to Lakehouse Flow](flow_diagram.png)

---

## Load Config from JSON (Optional)
Paste this cell **above the CONFIG cell** (or at the top of the CONFIG cell) in the notebook to load `sample_config.json` automatically if present:

```python
import json, os
config_path = 'sample_config.json'
if os.path.exists(config_path):
    with open(config_path, 'r', encoding='utf-8') as f:
        cfg = json.load(f)
    # Inject into global namespace so CONFIG variables are set
    globals().update(cfg)
    print(f"Loaded {len(cfg)} settings from {config_path}")
else:
    print("No sample_config.json found; using inline CONFIG values.")
```
