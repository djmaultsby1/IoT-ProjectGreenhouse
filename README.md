# Greenhouse Data Integration Project

This project combines three data feeds from one greenhouse on 15 July 2026 into a single, dashboard-ready CSV. The sources come from different devices and use different formats, date styles, and units. Together they let an analyst relate soil conditions, outdoor weather, and irrigation activity, which no single file allows.

## Repository contents

| Path | What it is |
|------|-----------|
| `data/source/batch102.csv` | Soil sensor readings (CSV): 10 probes in 5 beds, every 2 hours, 120 records. |
| `data/source/feed102.xml` | Weather station WS-01 observations (XML): every 10 minutes, 144 records. |
| `data/source/stream102.json` | Irrigation pump log (JSON): 5-minute entries while a zone runs, 38 records. |
| `data/integrated_greenhouse.csv` | **The integrated dataset**: 80 rows × 10 fields, one row per soil reading with matching weather and irrigation context. |
| `data_dictionary.md` | Name, type, units, and meaning of every field in the integrated file. |
| `report.md` | Written report: sources, questions answered, format choice, record design, cleaning, and trade-offs. |
| `README.md` | This file. |


