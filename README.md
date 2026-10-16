NCMRWF RDS Downloader for Windows
A native Windows tool for downloading climate and reanalysis data from the NCMRWF Research Data Server portal.

📦 Downloads
File	Size	Notes
RDS-Downloader-Tk.exe	18 MB	tkinter/ttk — zero extra runtime needed
No Python, bash, curl, or jq required on the target machine.

✨ Features
Date selection

Inline month‑grid calendar — click start date, click end date to select a range
Season presets: All year, MAM, JJAS, Oct‑Nov, DJF, Summer, Winter
“Advanced options” still opens the classic checkbox dialogs if you want to hand‑pick non‑contiguous months or days
Live caption shows exactly what will be requested (e.g., "10 Jun 2005 → 20 Sep 2005 | requesting 4 month(s) × all 31 days")
Dataset selection

Pick from 9 available datasets:
NGFS Daily Single Levels (1999‑2018)
NGFS 6‑Hourly Single Levels (1999‑2018)
NGFS 6‑Hourly Pressure Levels (1999‑2018)
IMDAA Daily Single Level (1979‑2020)
IMDAA Hourly Single Levels (1979‑2022)
IMDAA 3‑Hourly Pressure Levels (1979‑2022)
S2S Sub‑seasonal Re‑forecast (1993‑2015) ← new
NEMO Ocean Analysis (2016‑2026) ← new
MERA Hourly Rainfall over India (2020‑2025) ← new
Variable and time options

Per‑dataset variable lists (so you only see variables that the dataset actually supports)
Time selectors: hourly steps (00‑23), 3‑hourly steps, or forecast leads for S2S
Pressure‑level selectors for pressure‑level datasets
Variable groups are properly scoped per dataset — no more collision bugs
Spatial domain

Region presets: India, Asia, Global
Manual bounding‑box editor (north/south/east/west)
Datasets respect their natural domain (e.g., ocean uses 0‑360° longitude, S2S is global)
Output

Choose download folder (defaults to a downloads/ folder next to the exe)
Progress log with colour‑coded messages
Stop button responds immediately (no waiting for poll intervals)


Auto download in the background if data is processed.
Themes

Light and dark colour schemes
Theme settings are saved and restored between sessions
