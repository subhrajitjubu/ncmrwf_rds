# NCMRWF RDS Downloader

A CDS-style shell script client for downloading data from the
[NCMRWF Research Data Store](https://rds.ncmrwf.gov.in).

---

## File Structure

```
project/
├── rds_config.sh          ← credentials, bbox defaults, dataset definitions
├── rds_client.sh          ← core engine (never edit)
├── download.sh            ← your download requests
├── download_existing.sh   ← download already-completed jobs from portal
├── logs/
│   └── submitted_jobs.log ← auto-maintained job log
└── downloads/             ← output folder
```

---

## Setup

### 1. Install dependencies

```bash
# jq is required for JSON parsing
sudo apt install jq        # Ubuntu/Debian
sudo yum install jq        # CentOS/RHEL
brew install jq            # macOS
```

### 2. Configure credentials

Edit `rds_config.sh` and set your email and password:

```bash
RDS_EMAIL="your@email.com"
RDS_PASSWORD="yourpassword"
```

### 3. Make scripts executable

```bash
chmod +x rds_config.sh rds_client.sh download.sh download_existing.sh
```

---

## How to Run

### Download new data
Edit `download.sh` with your requests, then run:
```bash
./download.sh
```

### Download already-completed jobs from portal
```bash
./download_existing.sh
```

---

## How to Define a Request

All requests are made using the `retrieve` function inside `download.sh`.
The syntax mirrors the CDS API style.

```bash
#!/bin/bash
source ./rds_client.sh

retrieve \
    --dataset   "<dataset-name>" \
    --year      "<year>" \
    --month     "<months>" \
    --day       "<days>" \
    --time      "<times>" \
    --<vargroup> "<variables>" \
    --target    "<output-file.zip>"
```

### Multiple requests in one script

```bash
#!/bin/bash
source ./rds_client.sh

# Request 1
retrieve \
    --dataset  "imdaa-daily-single" \
    --year     "2000" \
    --month    "all" \
    --day      "all" \
    --daily_sum "APCP-sfc" \
    --target   "./downloads/precip_2000.zip"

# Request 2 — runs after Request 1 completes
retrieve \
    --dataset  "ngfs-hourly-single" \
    --year     "1999" \
    --month    "06,07,08,09" \
    --day      "all" \
    --time     "00,06,12,18" \
    --soil     "TSOIL-L1,TSOIL-L2" \
    --target   "./downloads/ngfs_jjas_1999.zip"
```

---

## Request Parameters

### Common Parameters (all datasets)

| Parameter  | Required | Values | Description |
|------------|----------|--------|-------------|
| `--dataset` | ✓ | see Dataset Names | Dataset to download from |
| `--year`    | ✓ | `"1999"` | Single year as string |
| `--month`   | ✓ | `"all"` or `"01,06,12"` | Month(s) — `all` expands to 01–12 |
| `--day`     | ✓ | `"all"` or `"01,15,31"` | Day(s) — `all` expands to 01–31 |
| `--target`  | ✓ | `"./downloads/out.zip"` | Output file path |
| `--north`   | optional | `"45"` | Bounding box north (overrides dataset default) |
| `--south`   | optional | `"-15"` | Bounding box south |
| `--east`    | optional | `"120"` | Bounding box east |
| `--west`    | optional | `"30"` | Bounding box west |

### Bounding Box Priority
```
--north/south/east/west in retrieve()   ← highest priority
    └── dataset default in rds_config.sh
            └── global default in rds_config.sh  ← fallback
```

### Default Bounding Box (all datasets)
```
North: 45   South: -15   East: 120   West: 30
```

---

## Dataset Names and Parameters

---

### 1. `ngfs-daily-single`
**NGFS daily data on single levels — 1999 to 2018**

```bash
retrieve \
    --dataset       "ngfs-daily-single" \
    --year          "1999" \
    --month         "all" \
    --day           "all" \
    --daily_mean    "Tmean" \
    --target        "./downloads/ngfs_daily_1999.zip"
```

| Parameter | Required | Available Values |
|-----------|----------|-----------------|
| `--daily_mean`    | at least one group | `Tmean` |
| `--daily_sum`     | at least one group | `APCP-sfc` |
| `--daily_maximum` | at least one group | `Tmax` |
| `--daily_minimum` | at least one group | `Tmin` |

---

### 2. `ngfs-hourly-single`
**NGFS 6-hourly data on single levels — 1999 to 2018**

```bash
retrieve \
    --dataset                   "ngfs-hourly-single" \
    --year                      "1999" \
    --month                     "all" \
    --day                       "all" \
    --time                      "00,06,12,18" \
    --soil                      "TSOIL-L1,TSOIL-L2,TSOIL-L3,TSOIL-L4" \
    --wind                      "GUST-sfc,VGRD-10m-agl,UGRD-10m-agl" \
    --temperature_and_pressure  "TMP-sfc,PRES-sfc,TMP-2m-agl,PRMSL-msl" \
    --others                    "RH-2m-agl,LAND-sfc,WEASD-sfc,ICEC-sfc,VIS-sfc,HPBL-sfc,ICETK-sfc" \
    --target                    "./downloads/ngfs_hourly_single_1999.zip"
```

| Parameter | Available Values |
|-----------|-----------------|
| `--time` | `00` `06` `12` `18` |
| `--soil` | `TSOIL-L1` `TSOIL-L2` `TSOIL-L3` `TSOIL-L4` |
| `--wind` | `GUST-sfc` `VGRD-10m-agl` `UGRD-10m-agl` |
| `--temperature_and_pressure` | `TMP-sfc` `PRES-sfc` `TMP-2m-agl` `PRMSL-msl` |
| `--others` | `RH-2m-agl` `LAND-sfc` `WEASD-sfc` `ICEC-sfc` `VIS-sfc` `HPBL-sfc` `ICETK-sfc` |

---

### 3. `ngfs-hourly-pressure`
**NGFS 6-hourly data on pressure levels — 1999 to 2018**

```bash
retrieve \
    --dataset        "ngfs-hourly-pressure" \
    --year           "1999" \
    --month          "01" \
    --day            "all" \
    --time           "00,06,12,18" \
    --pressure_level "100_hpa,200_hpa,500_hpa,850_hpa,1000_hpa" \
    --variables      "UGRD-prl,VGRD-prl,TMP-prl,RH-prl,HGT-prl" \
    --target         "./downloads/ngfs_hourly_pressure_1999_01.zip"
```

| Parameter | Available Values |
|-----------|-----------------|
| `--time` | `00` `06` `12` `18` |
| `--pressure_level` | `1_hpa` `2_hpa` `3_hpa` `5_hpa` `7_hpa` `10_hpa` `20_hpa` `30_hpa` `50_hpa` `70_hpa` `100_hpa` `125_hpa` `150_hpa` `175_hpa` `200_hpa` `225_hpa` `250_hpa` `275_hpa` `300_hpa` `325_hpa` `350_hpa` `375_hpa` `400_hpa` `425_hpa` `450_hpa` `475_hpa` `500_hpa` `525_hpa` `550_hpa` `575_hpa` `600_hpa` `625_hpa` `650_hpa` `675_hpa` `700_hpa` `725_hpa` `750_hpa` `775_hpa` `800_hpa` `825_hpa` `850_hpa` `875_hpa` `900_hpa` `925_hpa` `950_hpa` `975_hpa` `1000_hpa` |
| `--variables` | `UGRD-prl` `VGRD-prl` `TMP-prl` `RH-prl` `HGT-prl` |

---

### 4. `imdaa-daily-single`
**IMDAA daily data on single level — 1979 to 2020**

```bash
retrieve \
    --dataset        "imdaa-daily-single" \
    --year           "1979" \
    --month          "all" \
    --day            "all" \
    --daily_sum      "APCP-sfc" \
    --daily_mean     "Tmean,T2m_mean" \
    --daily_maximum  "Tmax,T2m_max" \
    --daily_minimum  "Tmin,T2m_min" \
    --target         "./downloads/imdaa_daily_1979.zip"
```

| Parameter | Available Values |
|-----------|-----------------|
| `--daily_sum`     | `APCP-sfc` |
| `--daily_mean`    | `Tmean` `T2m_mean` |
| `--daily_maximum` | `Tmax` `T2m_max` |
| `--daily_minimum` | `Tmin` `T2m_min` |

---

### 5. `imdaa-hourly-single`
**IMDAA hourly data on single levels — 1979 to 2020**

```bash
retrieve \
    --dataset                   "imdaa-hourly-single" \
    --year                      "1979" \
    --month                     "all" \
    --day                       "all" \
    --time                      "00,01,02,03,04,05,06,07,08,09,10,11,12,13,14,15,16,17,18,19,20,21,22,23" \
    --soil                      "TSOIL-L1,TSOIL-L2,TSOIL-L3,TSOIL-L4,CISOILM-L1,CISOILM-L2,CISOILM-L3,CISOILM-L4" \
    --wind                      "GUST-10m,VGRD-10m,VGRD-50m,UGRD-10m,UGRD-50m" \
    --cloud                     "LCDC-atmc,HCDC-atmc,VLCDC-atmc,MCDC-atmc,TCDCRO-atmc" \
    --rain_and_snow             "APCP-sfc,SNOL-sfc,CWP-sfc,SNOC-sfc,WEASD-sfc,LSWP-sfc" \
    --radiation_and_heat        "DSWRF-toa,NLWRF-sfc,NSWRF-sfc,ULWRF-toa,DLWRF-sfc,DSWRF-sfc,SHTFL-sfc,LHTFL-sfc,NSWRF-toa" \
    --temperature_and_pressure  "TMP-sfc,PRES-sfc,TMP-2m,PRMSL-msl" \
    --others                    "RH-2m,LAND-sfc,ICEC-sfc,EVARSS-sfc,VIS-2m,EVARCA-sfc,HPBL-sfc,ICETK-sfc,ROHLENABL-sfc" \
    --target                    "./downloads/imdaa_hourly_single_1979.zip"
```

| Parameter | Available Values |
|-----------|-----------------|
| `--time` | `00` through `23` (all 24 hours) |
| `--soil` | `TSOIL-L1` `TSOIL-L2` `TSOIL-L3` `TSOIL-L4` `CISOILM-L1` `CISOILM-L2` `CISOILM-L3` `CISOILM-L4` |
| `--wind` | `GUST-10m` `VGRD-10m` `VGRD-50m` `UGRD-10m` `UGRD-50m` |
| `--cloud` | `LCDC-atmc` `HCDC-atmc` `VLCDC-atmc` `MCDC-atmc` `TCDCRO-atmc` |
| `--rain_and_snow` | `APCP-sfc` `SNOL-sfc` `CWP-sfc` `SNOC-sfc` `WEASD-sfc` `LSWP-sfc` |
| `--radiation_and_heat` | `DSWRF-toa` `NLWRF-sfc` `NSWRF-sfc` `ULWRF-toa` `DLWRF-sfc` `DSWRF-sfc` `SHTFL-sfc` `LHTFL-sfc` `NSWRF-toa` |
| `--temperature_and_pressure` | `TMP-sfc` `PRES-sfc` `TMP-2m` `PRMSL-msl` |
| `--others` | `RH-2m` `LAND-sfc` `ICEC-sfc` `EVARSS-sfc` `VIS-2m` `EVARCA-sfc` `HPBL-sfc` `ICETK-sfc` `ROHLENABL-sfc` |

---

### 6. `imdaa-hourly-pressure`
**IMDAA 3-hourly data on pressure levels — 1979 to 2020**

```bash
retrieve \
    --dataset        "imdaa-hourly-pressure" \
    --year           "1979" \
    --month          "all" \
    --day            "all" \
    --time           "00,03,06,09,12,15,18,21" \
    --pressure_level "10_hpa,20_hpa,30_hpa,50_hpa,70_hpa,100_hpa,150_hpa,200_hpa,250_hpa,300_hpa,400_hpa,500_hpa,600_hpa,700_hpa,800_hpa,850_hpa,925_hpa,950_hpa,1000_hpa" \
    --variables      "UGRD-prl,VGRD-prl,TMP-prl,RH-prl,HGT-prl" \
    --target         "./downloads/imdaa_hourly_pressure_1979.zip"
```

| Parameter | Available Values |
|-----------|-----------------|
| `--time` | `00` `03` `06` `09` `12` `15` `18` `21` |
| `--pressure_level` | `10_hpa` `20_hpa` `30_hpa` `50_hpa` `70_hpa` `100_hpa` `150_hpa` `200_hpa` `250_hpa` `300_hpa` `400_hpa` `500_hpa` `600_hpa` `700_hpa` `800_hpa` `850_hpa` `925_hpa` `950_hpa` `1000_hpa` |
| `--variables` | `UGRD-prl` `VGRD-prl` `TMP-prl` `RH-prl` `HGT-prl` |

---

## Looping Over Multiple Years

```bash
#!/bin/bash
source ./rds_client.sh

# Sequential — one year at a time
for year in $(seq 1999 2018); do
    retrieve \
        --dataset    "ngfs-daily-single" \
        --year       "${year}" \
        --month      "all" \
        --day        "all" \
        --daily_mean "Tmean" \
        --target     "./downloads/ngfs_daily_${year}.zip"
done
```

For parallel submission (all years submitted at once, polled together)
use the dedicated `download_precip_jjas.sh` as a template.

---

## Token / Authentication

- Token is a JWT stored as a cookie, valid for **15 days**
- The client **auto-checks** token expiry before every request
- If expired, it **auto-logs in** using credentials from `rds_config.sh`
- On `HTTP 401` mid-run it re-logs in and retries once automatically
- You never need to manually manage the token

---

## Job Tracking

Every submitted job is logged to `logs/submitted_jobs.log`:

```
job_id|dataset_key|job_type_label|year|status|target_file
abc123...|ngfs_daily_single|ngfsdaily_1999_all|1999|DOWNLOADED|./downloads/ngfs_daily_1999.zip
def456...|imdaa_daily_single|daily_2000_06_07_08_09|2000|RUNNING|./downloads/precip_2000.zip
```

Status values: `SUBMITTED` → `COMPLETED` → `DOWNLOADED` (or `FAILED` / `DOWNLOAD_FAILED`)

---

## Download URL Pattern

All completed job downloads follow this URL structure:
```
https://rds.ncmrwf.gov.in/downloads/{job_id}/{job_id}.zip
```

Files are downloaded as `{job_id}.zip` and renamed to your `--target` filename.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `HTTP 000` | SSL interception on your network — `-k` flag is already set |
| `HTTP 401` | Token expired — client auto-refreshes, or check credentials in `rds_config.sh` |
| `HTTP 403` | Wrong endpoint or missing cookie |
| Download is `< 4KB` | Server returned an error JSON — check job status on portal |
| `jq: command not found` | Install jq: `sudo apt install jq` |
| `Unknown dataset` | Check dataset name spelling against Dataset Names section above |

