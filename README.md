# Ecotron Sensor Data: CSV → JSON-LD Demo

This repository shows how time-series data from a soil/plant **Ecotron** (a controlled-environment chamber with a lysimeter) can be transformed from a flat sensor table into **linked data (JSON-LD)** using standard W3C and domain ontologies.

It contains a small sample only: about **12 minutes** of data from **25 June 2026**. The goal is to demonstrate the transformation, not to publish a complete dataset.

## Files

| File | What it is |
|---|---|
| `merged_preview.csv` | The input: 34 timestamps × 103 signals, merged from the facility's monitoring dashboard exports |
| `merged_preview.jsonld` | The output: the same data as a JSON-LD graph (1,047 nodes) |
| `merged_preview_preview3.jsonld` | A small excerpt of the output, covering the first 3 timestamps, for quick reading |

## What's in the data

| Group | Examples |
|---|---|
| Chamber air | air temperature, relative and absolute humidity, fresh/exhaust air flow |
| Greenhouse gases | CO₂ (ppm), CH₄ and N₂O (ppb) |
| Lighting | Red, Far Red, UV, Cold White and Warm White channel intensities |
| Soil | soil temperature (upper/middle/lower zones), matric potential and temperature (T21, T32), volumetric water content, bulk electrical conductivity and temperature (TP32), heat flux plate |
| Lysimeter | lysimeter and drainage weight, lower boundary condition (LBC) pumping, irrigation |
| Barometer | atmospheric pressure, sensor temperature |
| Controller state | PID setpoints and outputs, heating/cooling/humidification/dehumidification modes |

## How the transformation works

Each non-empty cell in the CSV (one signal at one time) becomes one node in the graph:

- **Measurements** (sensor readings) become `sosa:Observation` nodes. **Control actions** (setpoints, controller outputs, modes) become `sosa:Actuation` nodes.
- Each **timestamp** becomes a shared `time:Instant` node, and observations point to it with `sosa:resultTime`.
- Each observation links to:
  - the **property** it measures (`sosa:observedProperty`), using a standard vocabulary term where one exists
  - the **feature of interest** (chamber air or soil column)
  - the **sensor** or **actuator** that produced it
  - a **unit** (QUDT)
- The graph also describes the Ecotron itself: sensors, actuators, soil samples at depth, a soil profile, and the platform.

### Example

CSV cell (`CO2_PID_PV` at 03:36:30 = `454.0`):

```json
{
  "@id": "ecotron:observation/2026-06-25T03_36_30/CO2_PID_PV",
  "@type": "sosa:Observation",
  "resultTime": "ecotron:time/2026-06-25T03_36_30",
  "hasFeatureOfInterest": "ecotron:foi/chamber-air",
  "observedProperty": "cf:mole_fraction_of_carbon_dioxide_in_air",
  "hasSimpleResult": { "@value": 454.0, "@type": "xsd:double" },
  "unit": "qudt:PPM"
}
```

### Vocabularies used

| Prefix | Vocabulary | Used for |
|---|---|---|
| `sosa:` / `ssn:` | [W3C SOSA/SSN](https://www.w3.org/TR/vocab-ssn/) | observations, actuations, sensors, samples, platform |
| `time:` | [W3C OWL-Time](https://www.w3.org/TR/owl-time/) | timestamps |
| `glosis:` | [GLOSIS](http://w3id.org/glosis/model/) | soil profile and soil properties (e.g. electrical conductivity) |
| `sweet:` | [SWEET](http://sweetontology.net/) | matric potential, soil temperature |
| `cf:` | [CF Standard Names](http://vocab.nerc.ac.uk/standard_name/) | air temperature, humidity, pressure, gas concentrations, soil heat flux |
| `qudt:` | [QUDT units](http://qudt.org/vocab/unit/) | units |
| `ecotron:` | local extension | facility-specific properties with no standard term (LBC, light channels, modes, …) |

## Validation

The output was checked against the input in both directions:

- **All 1,030 non-empty CSV cells are accounted for.**
  - 982 appear directly as observations or actuations, with the same time, signal and value.
  - The other 48 are duplicate columns (see below), whose values are identical to the copy that was kept.
- **Every node traces back to the CSV.** No observation lacks a matching CSV cell.
- **Values match** to within 5 × 10⁻⁷, because results are rounded to 6 decimal places.
- **References resolve:** all 34 timestamps are present, every `resultTime` points to an existing `time:Instant`, and there are no duplicate `@id`s.

## Notes and limitations

- **Time zone:** timestamps are in the dashboard's local time (believed to be US Pacific, UTC−7 in June). No offset is recorded in the files.
- **`_x` / `_y` columns:** some signals appeared in more than one dashboard panel. Merging those panels produced pairs such as `Temperature_air_PID_PV_x` and `_y`. Where both copies have a value they are identical, and the JSON-LD keeps only one.
- **Placeholder URIs:** `ecotron:` currently expands to `http://example.org/ecotron/`. It is not yet a resolvable, permanent namespace.
- **Resolution:** values are at 30-second intervals as exported from the dashboard. They may be dashboard-aggregated rather than raw sensor readings.
- **Sample only:** this is a short preview window. The facility collects data continuously.

## License

<!-- Choose a license, e.g. CC-BY-4.0 for data, and add it here and as a LICENSE file. -->

## Contact

<!-- Your name, lab, and institution -->

