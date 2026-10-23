# Greenhouse Data Integration Report

## 1. The three datasets

| | Soil sensors | Weather station | Irrigation log |
|---|---|---|---|
| **File** | `batch102.csv` | `feed102.xml` | `stream102.json` |
| **Device** | 10 soil probes, two in each of five beds | Weather station `WS-01` | Irrigation controller / pump with flow meter, serving zones 1–3 |
| **Measures** | Soil moisture and soil temperature | Air temperature, humidity, wind speed | Flow rate and pump status per zone |
| **Format** | CSV, one header row | XML, `<observation>` elements with attributes and child elements carrying a `unit` attribute | JSON, an object with a `Log` array of flat objects |
| **One record is** | One sensor's reading at one time | One observation at one time | One pump-log entry for one zone at one time |
| **Time style** | `MM/DD/YYYY`, 12-hour with AM/PM | `DD/MM/YYYY`, 24-hour | `MM/DD/YYYY`, 12-hour with AM/PM |

This assignment refers to the "Greenhouse data"; I used the soil-sensor file for that because it is the only one recording the conditions within the beds. The sources have no specific designation as to whether WS-01 was indoors or out; with the air temperature from around 74 F to 94 F, humidity drifting from 93% down to 44% during the day, it appears to be recording conditions outside or nearby, so I refer to it simply as "the weather station".

## 2. Questions the integrated file can answer

1. **How fast does soil dry out as the day heats up?** On average, readings from all probes dropped from roughly 34.1% at 08:00 to 28.6% at 16:00. During this period, air temperature increased from 24.4 C to 34.5 C, and humidity fell from 87% to 46%. By aligning soil moisture data with air temperature, humidity, and wind information, an analyst can observe how soil drying corresponds to weather changes. The irrigation log identifies periods without watering, allowing the drying effect to be distinguished from irrigation.
2. **Which beds correspond to which irrigation zone??** As the log, zone 1 operated from 05:00 to 05:30, and beds 1 and 2 increased by about 7 moisture points by 06:00. Zone 3 ran from 06:30 to 07:00, and bed 5 increased by about 7 points by 08:00. Zone 2 was active from 13:05 to 13:25, and only beds 3 and 4 showed a rise by 14:00. This implies zone 1 is linked to beds 12, zone 2 to beds 34, and zone 3 to bed 5. This is an inference rather than a confirmed mapping, as neither the soil file nor the irrigation log gives this information directly.
3. **How dry do the beds become before watering?** The 16:00 measurement matches the start of a zone 1 cycle. At that time, beds 1 and 2 reached their lowest moisture levels of the day, with an air temperature of 34.5 C and 46% humidity. An analyst can compare soil moisture and weather conditions at the beginning of each cycle to assess whether the schedule aligns with soil needs.
4. **How closely does soil temperature track air temperature?** Average soil temperature increased from 71.4 F at 06:00 to 91.3 F at 18:00, and continued rising at 18:00, even though air temperature had reached its maximum of 34.5 C at 16:00. Comparing soil temperature with air temperature indicates that soil temperature lags behind air temperature. An analyst may also examine whether temperature patterns change around watering times.
5. **Do the two probes in each bed agree, and can you rule out a probe failure?** The paired probes agreed by only 0.0 to 0.6 points in bed 1 and 0.1 to 0.8 points in bed 5, but by 0.7 to 1.2 points in bed 2, as much as 1.2 points in bed 3, and as much as 1.7 points in bed 4. Since each row also has a weather and irrigation field, an analyst can tell whether the difference between a pair of sensors is caused by uneven water application or by a calibration or installation problem.

## 3. Format choice: CSV

I chose **CSV** because the integrated file is a flat table: every record has the same 17 fields and no repeating or nested groups, which is exactly what CSV expresses. Dashboard tools open it directly, and it is compact and easy to diff in Git.

- **Why not JSON:** JSON's strength is nested or variable-shaped records, such as the irrigation log with lists of events per zone. Our data has no nesting after integration, so JSON would repeat every field name on every record, enlarge the file, and need flattening before most dashboards could use it.
- **Why not XML:** XML is the most verbose option and has the weakest native dashboard support. Its advantages, such as attributes and schema validation, are not needed for a flat numeric table.

CSV does lose self-described types and units, which is why the units appear in field names (`_f`, `_pct`, `_mph`, `_gal`) and in the separate data dictionary.

## 4. The integrated dataset

**One record = one soil-sensor reading**, joined to the weather observation and the irrigation log entry with the same timestamp. The file has 80 records and 11 fields in three groups, all taken directly from the sources:

- **Soil (CSV):** `timestamp_local`, `sensor_id`, `bed_id` , `soil_moisture_vwc_pct`, `soil_temp_f`. `sensor_id` nests within `bed_id`.
- **Weather (from the XML):** `air_temp_f`, `humidity_pct`, `wind_speed_mph`.
- **Irrigation (from the JSON):** `pump_on_at_reading`, `active_zone`.

**How the sources are joined:** All three sources have a time axis; that is the join key. Soil samples are on the even hours, so they match the weather station exactly on the 10-minute interval. The irrigation log is summarized over the 2 hours before the soil sampling, giving the same sampling interval and this way telling us 'what happened since the last sample?'.

**How I chose the sample:** The file is for **04:00 to 18:00 on 15 July 2026, a total of 14 hours'** duration, exceeding the minimum of 6 hours. I chose this time window because the irrigation log only recorded activity from 05:0017:15, so it contains all cycles; it started with a pre-irrigation baseline reading and finished with a reading after the final cycle, and for a long dry period plus a hot afternoon. The midnight 02:00 and 20:0022:00 readings are omitted don't record irrigation and don't offer much for questions that connect water to soil response.

## 5. Data cleaning and normalization

The three sources had no missing values, no unparseable values, and no duplicate keys (checked on load), so cleaning was mainly standardization:

- **Date and time:** the sources use three conventions. Each was parsed with an explicit format so that day and month cannot be swapped, and all were converted to ISO 8601, which sorts correctly and is understood by dashboards. No timezone is given in any source, so none was invented.
- **Units:** air temperature was converted from °C to °F so soil temperature and air temperature are directly comparable and a single temperature unit is used in the file. Wind stays in mph, humidity in percent; units were already uniform within each source.
- **Field names:** renamed to one consistent `snake_case`. The XML's separate `unit` attributes were folded into field names, and the XML's nested elements and the JSON's `Log` array were flattened to tables before joining.
- **Missing values created by the integration:** when the irrigation log has no entry at a reading time, the three irrigation fields are left blank rather than filled with a guess. A blank means "no log entry", not "pump off".
- **Assumption:** `SoilMoisture_VWC` is assumed to be a percentage of volumetric water content; the source does not state the unit.

## 6. Trade-offs

**What was gained:** a single table in which each soil reading sits next to the weather at that moment and the irrigation log entry, so questions that span devices can be answered with one filter or chart instead of matching three files with different timestamp styles. Everything is in one consistent format with documented fields.

**What was lost or simplified:**
**Irrigation coverage is thin:** an exact-timestamp join means only 2 of 8 timestamps carry irrigation values. Cycles that start and end between soil readings do not appear in the file, so the irrigation history has to be read from the source log.
**Resolution:** weather is reported every 10 minutes and irrigation every 5 minutes, but the file is at the soil sensors' 2-hour resolution, so only the weather value at the reading time is kept.
**No calculated fields:** values such as change since the last reading, total water delivered, or soil-minus-air temperature are not stored. This keeps the file strictly source data, but an analyst must compute them, and the mixed temperature units must be converted first.
**Irrigation is whole-greenhouse:** with no documented zone-to-bed mapping, irrigation fields are not specific to a bed. The mapping suggested in section 2 is only an inference.
**Repetition:** weather and irrigation values repeat on all 10 rows of a timestamp, which suits a flat table but makes naive sums over the file overcount.
**Coverage:** only 14 hours of a single day, so multi-day patterns cannot be seen.
**CSV metadata:** types and units live in the data dictionary rather than in the file itself.
