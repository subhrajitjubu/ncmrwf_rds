# NCMRWF RDS Downloader

Download data from the [NCMRWF Research Data Store](https://rds.ncmrwf.gov.in).

There are three ways to run it, all built on the same request model — pick a dataset, a date
range and the variables you want, submit the request, wait for the portal to process it, and
download the resulting archive:

| Way to run it | Platform | Needs |
|---|---|---|
| **Desktop app** — tkinter or Qt | Windows 10+ | nothing else |
| **Desktop app** — Debian package | Ubuntu 22.04+ / Debian | `apt` installs what it needs |
| **Shell client** — CDS-style script | any Unix | `bash`, `curl`, `jq` |

The desktop app is the easiest route and is what most people want; the shell client is the
original, scriptable interface and is documented in full further down.

---

## Contents

- [What you get](#what-you-get)
- [Requirements](#requirements)
- [The desktop app](#the-desktop-app)
  - [Install](#install)
  - [Getting started](#getting-started)
  - [The window](#the-window)
  - [Features](#features)
- [Datasets](#datasets)
- [The shell client](#the-shell-client)
  - [File structure](#file-structure)
  - [Setup](#setup)
  - [How to run](#how-to-run)
  - [How to define a request](#how-to-define-a-request)
  - [Request parameters](#request-parameters)
  - [Dataset reference](#dataset-reference)
  - [Looping over multiple years](#looping-over-multiple-years)
  - [Token and authentication](#token-and-authentication)
  - [Job tracking](#job-tracking)
  - [Download URL pattern](#download-url-pattern)
- [Where your files live](#where-your-files-live)
- [Security and privacy](#security-and-privacy)
- [Building from source](#building-from-source)
- [Tests](#tests)
- [Project layout](#project-layout)
- [Troubleshooting](#troubleshooting)
- [Licence](#licence)

---

## What you get

| Deliverable | Size | Notes |
|---|---|---|
| `win/dist/RDS-Downloader-Tk.exe` | 18.3 MB | tkinter / ttk. No optional dependencies at all — the safe choice for a locked-down machine. |
| `win/dist/RDS-Downloader-Qt.exe` | 41.6 MB | PySide6 / Qt 6. Same features; visually closest to the original GTK client. |
| `linux/dist/rds-downloader_0.5_all.deb` | 50 KB | Ships both front-ends; `rds-downloader` is tkinter, `rds-downloader --qt` is Qt. |
| `rds_client.sh` + `rds_config.sh` | — | The original shell client, driven by `download.sh`. |

Both desktop builds are **v0.5** and at feature parity. All of them share the same download
engine and the same dataset definitions, so a request means the same thing whichever one you use.

## Requirements

- Windows 10 or later (64-bit) for the `.exe` files — nothing to install
- Ubuntu 22.04+ or a comparable Debian release for the `.deb` — `apt` pulls in
  `python3`, `python3-tk`, `python3-requests`, and `python3-pyside6.qtwidgets` only if you use `--qt`
- `bash`, `curl` and `jq` for the shell client (`sudo apt install jq`)
- An NCMRWF RDS account, and outbound HTTPS to `rds.ncmrwf.gov.in`

---

## The desktop app

### Install

**Windows.** Run `RDS-Downloader-Tk.exe` (smaller) or `RDS-Downloader-Qt.exe`. Windows
SmartScreen may warn that the file is unsigned — choose *More info → Run anyway*. That is
expected for a locally built `.exe`.

**Ubuntu / Debian.**

```bash
sudo apt install ./linux/dist/rds-downloader_0.5_all.deb
```

Use `apt`, not bare `dpkg -i`, so the dependencies get resolved. Then launch **NCMRWF RDS
Downloader** from the applications menu, or:

```bash
rds-downloader            # tkinter front-end
rds-downloader --qt       # Qt front-end
man rds-downloader        # options, environment variables, files
```

`sudo apt remove rds-downloader` removes the program but leaves your credentials, preferences
and downloads alone.

### Getting started

1. **Enter your credentials.** *Settings* tab → email and password. They are saved to
   `~/.ncmrwfapirc` (on Windows: `%USERPROFILE%\.ncmrwfapirc`), readable only by you, and
   reused on later runs. The app logs in for you and refreshes the session token automatically.
2. **Pick a dataset** from the *Dataset* dropdown. The time, variable and pressure-level lists
   underneath are rebuilt for that dataset.
3. **Pick the dates.** Click a start day on the calendar, then an end day — the range
   highlights. Or take a season preset. The caption below the calendar states exactly what will
   be requested.
4. **Tick the variables** you want (at least one).
5. **Set the download folder and the target file name.** The target understands `{dataset}` and
   `{year}` placeholders; with a multi-year range it writes one file per year.
6. **Submit & Download.** The request is submitted, polled to completion, and the archive is
   saved, with progress reported in the job log.

> **Generate Request** saves the exact request as a `request.txt` — a CDS-style block of
> `key = value` lines — so a run can be reproduced or archived outside the GUI.

### The window

The app opens at **1180×660** (minimum 960×540), which fits a 1366×768 14" laptop. Three tabs
sit in the middle of the window:

| Tab | Holds |
|---|---|
| **Download** | dataset, calendar + season presets, region, variables, target and the live request preview |
| **Presets** | saved request presets |
| **Settings** | credentials, download folder, theme, font size, and a button to open the config folder |

Above the tabs is the title bar — theme dropdown, **Cycle**, font-size dropdown — and below them
is the action bar and the job log, both **pinned outside the scroll area** so neither can scroll
off-screen however tall the panels grow or how large the font step.

### Features

**A real calendar.** Click once for the start date, click again for the end; `‹` / `›` move
between months, clamped to the dataset's real year span. Season presets: **All year, MAM, JJAS,
Oct-Nov, DJF, Summer, Winter**. **Advanced months/days…** opens checkbox dialogs if you want
non-contiguous months or days.

> The RDS API takes `month[]` and `day[]` as *separate* lists and requests their cartesian
> product, so a range like 05 Mar – 20 Jul cannot be expressed exactly. The app therefore always
> shows the resulting month × day set after you pick a range, so what will actually be fetched
> is never a surprise.

**15 themes.** Seven hand-built (Light, Dark, Midnight, Solarized, Amber, Ocean, High Contrast)
plus eight imported verbatim from the opencode `replit-deck` reference (Helix, Holm, Vance,
Bevel, World Dark, World Mint, Atlas, Bluehouse). Every theme is held to **WCAG AA (4.5:1)** for
body, muted and preview text — the lowest measured ratio is 4.66:1.

**Font sizes as roles.** Four steps — **Small / Normal / Large / X-Large**, default
**Large (1.2×)**. Font families are chosen per platform (Segoe UI / Consolas on Windows, DejaVu
Sans / DejaVu Sans Mono on Linux), so text renders in a font that is actually installed rather
than being silently substituted.

**A live request preview.** The exact payload that will be POSTed is shown beside the calendar,
built by the engine's own `build_payload()`, so what you see is what is sent — including the
dataset quirks (no `day` field for S2S, bare-number pressure levels).

**Download Existing — pick what you fetch.** The button fetches the list of completed jobs and
opens a picker (Select All / Clear All / Invert / search) with everything preselected, then
downloads only what you keep. Rows read `job type · short id · created`. This matters — the live
account held **31** completed jobs, so the alternative is a bulk download of 31 archives with no
way to say no.

**Collapsible panels.** The calendar and the request preview each fold away with a `▸` / `▾`
button — folding the calendar alone saves **228 px**. The fold state persists between runs.

**Presets and regions.** Save a whole request as a preset. Region presets India / Asia / Global
set the bounding box, or type it by hand.

**One prefs file.** Theme, font step and both fold states live in `~/.ncmrwf_gui/ui.cfg`. An
older `theme.cfg` holding just a theme name is still read.

---

## Datasets

The engine and the shell client define the same nine datasets; both GUIs generate their pickers
from those tables.

| Key | Label | Years | Time steps |
|---|---|---|---|
| `ngfs-daily-single` | NGFS Daily Single Levels | 1999–2018 | daily mean/sum/max/min |
| `ngfs-hourly-single` | NGFS 6-Hourly Single Levels | 1999–2018 | 4 (`00 06 12 18`) |
| `ngfs-hourly-pressure` | NGFS 6-Hourly Pressure Levels | 1999–2018 | 4 (`00 06 12 18`) |
| `imdaa-daily-single` | IMDAA Daily Single Level | 1979–2020 | daily sum/mean/max/min |
| `imdaa-hourly-single` | IMDAA Hourly Single Levels | 1979–2022 | 24 (`00`–`23`) |
| `imdaa-hourly-pressure` | IMDAA 3-Hourly Pressure Levels | 1979–2022 | 8 (`00 03 …21`) |
| `s2s-daily` | S2S Sub-seasonal Re-forecast | 1993–2015 | forecast lead, **no clock time** |
| `ocean-data` | NEMO Ocean Analysis | 2016–2026 | 8 (`00 03 …21`) |
| `mera` | MERA Hourly Rainfall over India | 2020–2025 | 24 (`00`–`23`) |

> **S2S pressure levels are bare numbers, not the `_hpa` form.** Other pressure datasets want
> `500_hpa`; S2S wants `500`. Both satisfy the server's schema (it only checks that they are
> strings), so the wrong one does not fail validation — it fails later with *"Unknown processing
> error"*. Both the engine and the shell client derive the correct form per dataset, so you never
> have to think about it.

---

## The shell client

The original interface: plain shell scripts, no GUI. Everything the desktop app does can be
scripted, which makes it the right tool for bulk or scheduled downloads.

### File structure

```
project/
├── rds_config.sh          ← credentials, bbox defaults, dataset definitions
├── rds_client.sh          ← core engine (never edit)
├── download.sh            ← worked requests for the six documented datasets
├── imda_parallel.sh       ← parallel multi-year submission template
├── imda_tp.sh             ← a single two-temperature request
├── test_down.sh           ← quick connectivity / smoke test
├── logs/
│   └── submitted_jobs.log ← auto-maintained job log
└── downloads/             ← output folder
```

Also in the tree, but unrelated to the packaged builds: `rds_gui3.py` and its `run.sh` backend
(the original GTK3 desktop GUI) and `rds_gui.py` / `rds_gui_2.py` (earlier drafts).

### Setup

```bash
# 1. jq is required for JSON parsing
sudo apt install jq        # Ubuntu/Debian
sudo yum install jq        # CentOS/RHEL
brew install jq            # macOS

# 2. set your credentials in rds_config.sh
RDS_EMAIL="your@email.com"
RDS_PASSWORD="yourpassword"

# 3. make the scripts executable
chmod +x rds_config.sh rds_client.sh download.sh imda_parallel.sh imda_tp.sh test_down.sh
```

### How to run

Edit `download.sh` with the requests you want, then:

```bash
./download.sh
```

There is no shell script for downloading already-completed jobs — the desktop app's **Download
Existing** button is the supported way to do that. On the command line you can fetch a finished
job directly from its URL (see [Download URL pattern](#download-url-pattern)).

### How to define a request

All requests are made with the `retrieve` function. The syntax mirrors the CDS API style.

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

Several requests in one script simply run one after another:

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

### Request parameters

Common to every dataset:

| Parameter | Required | Values | Description |
|---|---|---|---|
| `--dataset` | ✓ | see [Datasets](#datasets) | Dataset to download from |
| `--year` | ✓ | `"1999"` | Year (a range like `"2000-2005"` is also accepted) |
| `--month` | ✓ | `"all"` or `"01,06,12"` | Month(s) — `all` expands to 01–12 |
| `--day` | ✓ | `"all"` or `"01,15,31"` | Day(s) — `all` expands to 01–31 |
| `--target` | ✓ | `"./downloads/out.zip"` | Output file path |
| `--north` | optional | `"45"` | Bounding box north (overrides the dataset default) |
| `--south` | optional | `"-15"` | Bounding box south |
| `--east` | optional | `"120"` | Bounding box east |
| `--west` | optional | `"30"` | Bounding box west |

Dataset-specific groups: `--time`, `--pressure_level`, `--variables`, `--daily_mean`,
`--daily_sum`, `--daily_maximum`, `--daily_minimum`, `--soil`, `--wind`, `--cloud`,
`--rain_and_snow`, `--radiation_and_heat`, `--temperature_and_pressure`, `--others`, and
`--init_day` / `--forecast_day` for S2S.

**Bounding box priority.** `--north/south/east/west` in `retrieve()` beats the dataset default in
`rds_config.sh`, which beats the global fallback. The default for every dataset is
`North 45  South -15  East 120  West 30`.

### Dataset reference

The six datasets below are the ones with worked examples in `download.sh`. `s2s-daily`,
`ocean-data` and `mera` are defined too — S2S takes `--init_day` / `--forecast_day` and no
`--day`, and the desktop app is the easiest way to build those requests, since it enumerates each
dataset's variables for you.

<details>
<summary><strong>1. <code>ngfs-daily-single</code></strong> — NGFS daily, single levels, 1999–2018</summary>

```bash
retrieve \
    --dataset       "ngfs-daily-single" \
    --year          "1999" \
    --month         "all" \
    --day           "all" \
    --daily_mean    "Tmean" \
    --target        "./downloads/ngfs_daily_1999.zip"
```

| Parameter | Values (at least one group required) |
|---|---|
| `--daily_mean` | `Tmean` |
| `--daily_sum` | `APCP-sfc` |
| `--daily_maximum` | `Tmax` |
| `--daily_minimum` | `Tmin` |

</details>

<details>
<summary><strong>2. <code>ngfs-hourly-single</code></strong> — NGFS 6-hourly, single levels, 1999–2018</summary>

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

| Parameter | Values |
|---|---|
| `--time` | `00` `06` `12` `18` |
| `--soil` | `TSOIL-L1` `TSOIL-L2` `TSOIL-L3` `TSOIL-L4` |
| `--wind` | `GUST-sfc` `VGRD-10m-agl` `UGRD-10m-agl` |
| `--temperature_and_pressure` | `TMP-sfc` `PRES-sfc` `TMP-2m-agl` `PRMSL-msl` |
| `--others` | `RH-2m-agl` `LAND-sfc` `WEASD-sfc` `ICEC-sfc` `VIS-sfc` `HPBL-sfc` `ICETK-sfc` |

</details>

<details>
<summary><strong>3. <code>ngfs-hourly-pressure</code></strong> — NGFS 6-hourly, pressure levels, 1999–2018</summary>

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

| Parameter | Values |
|---|---|
| `--time` | `00` `06` `12` `18` |
| `--pressure_level` | `1_hpa` `2_hpa` `3_hpa` `5_hpa` `7_hpa` `10_hpa` `20_hpa` `30_hpa` `50_hpa` `70_hpa` `100_hpa` `125_hpa` `150_hpa` `175_hpa` `200_hpa` `225_hpa` `250_hpa` `275_hpa` `300_hpa` `325_hpa` `350_hpa` `375_hpa` `400_hpa` `425_hpa` `450_hpa` `475_hpa` `500_hpa` `525_hpa` `550_hpa` `575_hpa` `600_hpa` `625_hpa` `650_hpa` `675_hpa` `700_hpa` `725_hpa` `750_hpa` `775_hpa` `800_hpa` `825_hpa` `850_hpa` `875_hpa` `900_hpa` `925_hpa` `950_hpa` `975_hpa` `1000_hpa` |
| `--variables` | `UGRD-prl` `VGRD-prl` `TMP-prl` `RH-prl` `HGT-prl` |

</details>

<details>
<summary><strong>4. <code>imdaa-daily-single</code></strong> — IMDAA daily, single level, 1979–2020</summary>

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

| Parameter | Values |
|---|---|
| `--daily_sum` | `APCP-sfc` |
| `--daily_mean` | `Tmean` `T2m_mean` |
| `--daily_maximum` | `Tmax` `T2m_max` |
| `--daily_minimum` | `Tmin` `T2m_min` |

</details>

<details>
<summary><strong>5. <code>imdaa-hourly-single</code></strong> — IMDAA hourly, single levels, 1979–2022</summary>

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

| Parameter | Values |
|---|---|
| `--time` | `00` through `23` |
| `--soil` | `TSOIL-L1` `TSOIL-L2` `TSOIL-L3` `TSOIL-L4` `CISOILM-L1` `CISOILM-L2` `CISOILM-L3` `CISOILM-L4` |
| `--wind` | `GUST-10m` `VGRD-10m` `VGRD-50m` `UGRD-10m` `UGRD-50m` |
| `--cloud` | `LCDC-atmc` `HCDC-atmc` `VLCDC-atmc` `MCDC-atmc` `TCDCRO-atmc` |
| `--rain_and_snow` | `APCP-sfc` `SNOL-sfc` `CWP-sfc` `SNOC-sfc` `WEASD-sfc` `LSWP-sfc` |
| `--radiation_and_heat` | `DSWRF-toa` `NLWRF-sfc` `NSWRF-sfc` `ULWRF-toa` `DLWRF-sfc` `DSWRF-sfc` `SHTFL-sfc` `LHTFL-sfc` `NSWRF-toa` |
| `--temperature_and_pressure` | `TMP-sfc` `PRES-sfc` `TMP-2m` `PRMSL-msl` |
| `--others` | `RH-2m` `LAND-sfc` `ICEC-sfc` `EVARSS-sfc` `VIS-2m` `EVARCA-sfc` `HPBL-sfc` `ICETK-sfc` `ROHLENABL-sfc` |

</details>

<details>
<summary><strong>6. <code>imdaa-hourly-pressure</code></strong> — IMDAA 3-hourly, pressure levels, 1979–2022</summary>

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

| Parameter | Values |
|---|---|
| `--time` | `00` `03` `06` `09` `12` `15` `18` `21` |
| `--pressure_level` | `10_hpa` `20_hpa` `30_hpa` `50_hpa` `70_hpa` `100_hpa` `150_hpa` `200_hpa` `250_hpa` `300_hpa` `400_hpa` `500_hpa` `600_hpa` `700_hpa` `800_hpa` `850_hpa` `925_hpa` `950_hpa` `1000_hpa` |
| `--variables` | `UGRD-prl` `VGRD-prl` `TMP-prl` `RH-prl` `HGT-prl` |

</details>

### Looping over multiple years

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

For parallel submission — every year submitted at once and polled together — use
`imda_parallel.sh` as the template.

### Token and authentication

- The token is a JWT stored as a cookie, valid for **15 days**.
- The client checks expiry before every request and **logs in automatically** if it has expired,
  using the credentials in `rds_config.sh`.
- On `HTTP 401` mid-run it re-logs in and retries once.
- You never manage the token by hand.

### Job tracking

Every submitted job is appended to `logs/submitted_jobs.log`:

```
job_id|dataset_key|job_type_label|year|status|target_file
abc123...|ngfs_daily_single|ngfsdaily_1999_all|1999|DOWNLOADED|./downloads/ngfs_daily_1999.zip
def456...|imdaa_daily_single|daily_2000_06_07_08_09|2000|RUNNING|./downloads/precip_2000.zip
```

Status values: `SUBMITTED` → `COMPLETED` → `DOWNLOADED` (or `FAILED` / `DOWNLOAD_FAILED`).

### Download URL pattern

Every completed job is downloadable at:

```
https://rds.ncmrwf.gov.in/downloads/{job_id}/{job_id}.zip
```

Files are downloaded as `{job_id}.zip` and renamed to your `--target` filename.

---

## Where your files live

| What | Shell client | Desktop app |
|---|---|---|
| Credentials | `RDS_EMAIL` / `RDS_PASSWORD` in `rds_config.sh` | `~/.ncmrwfapirc` (mode `600`) |
| Preferences / theme | — | `~/.ncmrwf_gui/ui.cfg` |
| Request presets | — | `~/.ncmrwf_gui/presets.ini` |
| Job log | `logs/submitted_jobs.log` | `~/.ncmrwf_gui/submitted_jobs.log` |
| Downloads | wherever `--target` points (by convention `downloads/`) | the folder chosen in the GUI |

On Windows the app's files are under `%USERPROFILE%` instead of `~`.

The app's default download folder is `~/Downloads/rds_data` (`%USERPROFILE%\Downloads\rds_data`
on Windows). When you run the app from the source tree rather than the packaged build, the
default is the project's own `downloads/` directory.

## Security and privacy

A full source audit lives in [`SECURITY-AUDIT.md`](SECURITY-AUDIT.md). The points that matter
when running the app:

- **TLS verification is off by default.** The deployment network performs TLS interception, so
  the RDS certificate does not chain to a trusted CA and verification would fail every request.
  The trade-off is stated plainly: with verification off, your password and the downloaded
  archive are only as private as the network path to `rds.ncmrwf.gov.in`. On a network
  **without** interception, set `RDS_VERIFY_TLS=1` to require a trusted certificate. The app
  logs a warning at startup whenever verification is off.
- **Credentials are stored with owner-only permissions** — `~/.ncmrwfapirc` at mode `600` on
  Linux, or locked to your account with `icacls` on Windows. The password is never written to the
  job log.
- **A downloaded body must really be a ZIP.** A response that is too small, that is not a ZIP
  archive, or that exceeds the size cap is refused rather than saved as if it were your data, so
  a blocked or error response cannot be mistaken for a finished download.
- **A single download is capped at 8 GiB.** Change it with `RDS_MAX_DOWNLOAD_MB`; `0` turns the
  cap off. Raise it only if a legitimate archive is ever larger.
- **Job ids received from the server are validated** before they are used in a URL or a filename.

The app never unpacks archives (it downloads the `.zip` as-is), never runs a shell, and never
evaluates code from the server. The `md5sums` in the Debian package cover every installed file,
so `sudo debsums rds-downloader` can verify them.

## Building from source

**Windows executables.** The tkinter build needs only PyInstaller; the Qt build also needs
PySide6.

```powershell
python -m pip install pyinstaller
cd win
python -m PyInstaller RDS-Downloader-Tk.spec --noconfirm --clean

python -m pip install PySide6
python -m PyInstaller RDS-Downloader-Qt.spec --noconfirm --clean
```

Both `.spec` files carry the exclusion list that keeps the Qt build at 42 MB — the big wins are
dropping `QtQml`, `QtQuick`, `QtWebEngine*` and `Qt3D*`.

> **Kill a running copy first.** A onefile exe runs as a bootloader parent *and* a child, and the
> parent holds `dist\<name>.exe` open: `Get-Process | Where-Object { $_.ProcessName -like "*RDS*" } | Stop-Process -Force`

**Ubuntu / Debian package.** Build on Ubuntu/Debian, or in WSL Ubuntu on Windows:

```bash
bash linux/build_deb.sh
```

The output lands in `linux/dist/`. The two fields you may want to set are overridable without
editing the script:

```bash
MAINTAINER="Your Name <you@example.com>" bash linux/build_deb.sh
VERSION=0.6                              bash linux/build_deb.sh
```

Debian policy requires a `Maintainer` field with an email address, and it defaults to the name
shown in the app's About box plus the address from `~/.ncmrwfapirc` — override it if you would
rather not publish that address. The package's `copyright` file states `License: proprietary`,
because the tree has no licence file; replace that block with real licence text before you
redistribute the package.

The build stages the tree in `/tmp` rather than in the repo, because `dpkg-deb` records the file
modes it finds and the Windows filesystem (`/mnt/c`) does not represent them faithfully. It
forces `-Zxz` compression, because `dpkg-deb` now defaults to zstd, which `dpkg` older than
roughly 1.21 cannot read; the payload is ~50 KB either way.

## Tests

**Windows** — the suites resolve their own paths, so run them from the project root or `win/`:

```powershell
python win\tests\test_tk.py                 # 15 themes x 9 datasets x 4 font steps
python win\tests\test_qt.py                 # the same matrix, plus Qt layout invariants
python win\tests\test_security.py           # audit findings F1-F5 and F9-F14
python win\tests\test_gate.py               # hostile input rejected inside the real _gather()
python win\tests\test_existing_picker.py    # fetch/subset split, both pickers, Qt Run path
```

Each prints one line per check and exits non-zero on failure. They redirect the config directory
to a temp folder, so a run never touches your real preferences.

**Debian package** — needs no root and installs nothing:

```bash
bash linux/tests/test_deb.sh                     # newest package in linux/dist/
bash linux/tests/test_deb.sh path/to/some.deb    # or a specific one
```

It extracts the package and checks the archive structure, every control field, every file mode,
the md5sums, the docs (gzip integrity, man-page rendering with `groff`, icon XML), the `.desktop`
entry, the maintainer scripts, and the launcher's argument handling. Then it imports the
installed modules and **opens the installed GUI on `$DISPLAY`**, applying every theme, and execs
the real launcher against a simulated install prefix. Expect `72 passed, 0 failed`;
`desktop-file-validate` is skipped when that tool is absent. The GUI checks need an X display —
on a headless machine, `xvfb-run -a bash linux/tests/test_deb.sh`.

## Project layout

```
.
├── README.md                     ← this file
├── UI-WORKLOG.md                 ← design decisions D-01..D-27, with evidence
├── SECURITY-AUDIT.md             ← findings F1-F14 and their verification
├── rds_config.sh  rds_client.sh  download.sh      ← shell client
├── imda_parallel.sh  imda_tp.sh  test_down.sh     ← shell client extras
├── downloads/  logs/                              ← shell client output and job log
├── win/
│   ├── rds_engine.py             ← shared engine: auth, payload, submit, poll, download
│   ├── rds_themes.py             ← shared 15-theme palette + font-role tables
│   ├── rds_gui_tk.py             ← tkinter GUI (the reference build)
│   ├── rds_gui_qt.py             ← PySide6 GUI
│   ├── RDS-Downloader-Tk.spec    ← PyInstaller specs
│   ├── RDS-Downloader-Qt.spec
│   ├── tests/                    ← regression + security suites
│   └── dist/                     ← the two .exe files
└── linux/
    ├── build_deb.sh              ← build the package
    ├── files/                    ← .desktop, icon, man page, copyright, maintainer scripts
    ├── tests/test_deb.sh         ← validate the built package
    └── dist/                     ← the .deb
```

`win/rds_engine.py` and `win/rds_themes.py` are imported by both front-ends and by the Debian
package, so the three builds cannot drift apart on request semantics, colour or type.
`rds_themes.py` deliberately imports neither `tkinter` nor `PySide6`, which lets the regression
tests load it headlessly.

## Troubleshooting

| Problem | Applies to | Fix |
|---|---|---|
| SmartScreen "Windows protected your PC" | Windows app | The binary is unsigned. *More info → Run anyway*. |
| Antivirus quarantines the exe | Windows app | PyInstaller onefile false positive. The full source is here; rebuild it yourself if required. |
| `command not found: rds-downloader` | Debian app | The package is not installed, or `/usr/bin` is not on your `PATH`. |
| `Unmet dependencies` from `dpkg -i` | Debian app | Install with `apt install ./…deb` instead. |
| `--qt` says it needs the PySide6 bindings | Debian app | `sudo apt install python3-pyside6.qtwidgets`. |
| No menu entry appears after install | Debian app | Log out and back in, or run `update-desktop-database /usr/share/applications`. |
| `HTTP 000` | Shell client | TLS interception on your network — `-k` is already set. |
| `HTTP 000` from the app | Desktop app | Same cause. Set `RDS_VERIFY_TLS=1` only on a network without interception. |
| `HTTP 401` / `Login failed` | all | Token expired or wrong credentials — re-enter them in *Settings*, or check `rds_config.sh`. |
| `HTTP 403` | Shell client | Wrong endpoint or missing cookie. |
| `Download refused: too small` | Desktop app | The server returned an error document instead of an archive — the job may still be processing, or the request was rejected. |
| `Download refused: not a ZIP archive` | Desktop app | Same cause: an HTML/JSON error page rather than data. |
| Download is `< 4 KB` | Shell client | Server returned an error JSON — check the job status on the portal. |
| `Unknown processing error` from the portal | all | Only reachable with a malformed pressure-level form; the built-in lists are always correct. |
| The log stalls on "Waiting 60s" | Desktop app | Normal — the portal processes asynchronously and the client polls. **Stop** interrupts within a second. |
| `jq: command not found` | Shell client | `sudo apt install jq`. |
| `Unknown dataset` | Shell client | Check the spelling against the [Datasets](#datasets) table. |

## Licence

No licence file ships with this source tree, so no licence is granted by it: all rights are
reserved and the terms for redistributing these builds are undefined until the copyright holder
states a licence. The Debian package's `copyright` file records the same caveat in
machine-readable form.
