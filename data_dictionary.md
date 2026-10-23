# Data Dictionary: `data/integrated_greenhouse.csv`

One row = one soil-sensor reading, refined with weather conditions and irrigation activity for that moment. 80 rows, 17 fields. Blank cells mean "not applicable / not available".

| # | Field | Data type | Units | Source | Description |
|---|-------|-----------|-------|--------|-------------|
| 1 | `timestamp_local` | datetime | local time, `YYYY-MM-DDTHH:MM:SS` | Soil CSV | Time of the soil reading. |
| 2 | `sensor_id` | string | n/a | Soil CSV | Soil probe identifier. 10 distinct sensors. |
| 3 | `bed_id` | string | n/a | Soil CSV | Growing bed the probe sits in. Two probes per bed. |
| 4 | `soil_moisture_vwc_pct` | float | % volumetric water content | Soil CSV | Soil moisture at the reading time. |
| 5 | `soil_temp_f` | float | °F | Soil CSV | Soil temperature at the reading time. |
| 6 | `air_temp_f` | float | °F | Weather XML | Air temperature from station WS-01 at the same minute as the soil reading. |
| 7 | `humidity_pct` | float | % relative humidity | Weather XML | Humidity from station WS-01 at the reading time. |
| 8 | `wind_speed_mph` | float | miles per hour | Weather XML | Wind speed from station WS-01 at the reading time. |
| 9 | `pump_on_at_reading` | boolean (`True`/`False`) | n/a | Irrigation JSON | `True` if the irrigation log shows the pump running at exactly the reading time. |
| 10 | `active_zone` | integer | zone number (1–3) | Irrigation JSON | Zone being watered at the reading time. Blank when the pump is off. |

