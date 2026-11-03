---
license: cc-by-4.0
tags:
  - buildings
  - hvac
  - brick-schema
  - building-automation
  - time-series
  - semantic-modeling
pretty_name: "Purdue Building Operational Dataset"
---

# Purdue Building Operational Dataset

Operational time-series data and [Brick](https://brickschema.org/) semantic
models from two commercial buildings with VAV-AHU systems, released
as an open benchmark for AI-enabled building operation research.

## Overview

This dataset pairs 15-minute building automation system (BAS) trend data with
a Brick semantic model for each of two buildings. Every data column is linked
to a typed Brick entity (equipment, point class, and equipment relationships),
so the dataset can be used to benchmark automated semantic-model extraction,
point tagging, and equipment-relationship inference, and to develop
metadata-driven analytics such as fault detection and control.

It was collected to validate the Point2Brick and equipment-relationship
inference methods listed below. **Status:** v1.0, maintained by the Ray W.
Herrick Laboratories, Purdue University.

| | Building 1 | Building 2 |
|---|---|---|
| **Period** | 2025-06-25 to 2025-07-29 (35 days, cooling season) | 2023-01-01 to 2023-01-31 (heating season) |
| **Resolution** | 15 minutes | 15 minutes |
| **Timestamps** | 3,360 | 2,958 (data end 2023-01-31 19:15) |
| **Time-series points** | 378 (55 AHU-level, 323 VAV-level) | 201 (66 AHU-level, 135 VAV-level) |
| **Air-handling units** | 10 | 2 |
| **VAV terminal units** | 53 | 15 |
| **Missing values** | ~0.1% | ~8.2% |
| **Brick model entities** | 547 | 248 |

## Repository Structure

```
building1/
  building1_model.ttl                              # Brick model (Turtle)
  building1_15min_2025-06-25_to_2025-07-29.csv     # 15-min time series
building2/
  building2_model.ttl
  building2_15min_2023-01-01_to_2023-01-31.csv
LICENSE         # CC BY 4.0
CITATION.cff    # machine-readable citation
```

## Data Format

**Time series (`*.csv`).** Wide format: the first column is `Timestamp`
(`YYYY-MM-DD HH:MM:SS`, 15-minute intervals); every other column is one BAS
point. Empty cells are missing values. [Confirm: timestamps are local time
without a UTC offset.]

**Semantic model (`*_model.ttl`).** Brick Turtle files (namespace
`urn:building1/` or `urn:building2/`). Each point entity carries a
`ref:hasExternalReference` whose value is **exactly the CSV column name**
(all 378 and 201 columns match their model one-to-one). The models describe:

- **Equipment:** `brick:Air_Handling_Unit`, `brick:Variable_Air_Volume_Box`,
  and, for each VAV, a `brick:Damper` and a `brick:Heating_Valve` (`hasPart`)
- **Relationships:** `brick:feeds` (AHU to VAV), `brick:hasPoint`,
  `brick:hasPart`
- **Point classes:** e.g., `Zone_Air_Temperature_Sensor`,
  `Supply_Air_Flow_Sensor`, `Zone_Air_Heating/Cooling_Temperature_Setpoint`,
  `Damper_Position_Command`, `Valve_Position_Command`,
  `Supply_Air_Static_Pressure_Sensor`, and fan, coil, and damper commands
  for Building 2's AHUs

The models are equipment-and-point level. They do not include a spatial
hierarchy (building, floor, zone) or the central plant. In Building 2, 30
status and alarm points are typed only as `brick:Sensor`.

**Naming and units.** The two buildings use different point-naming
conventions (e.g., `VAV1_ROOM_TEMP` in Building 1 vs. `VAV1_01_Zone_Temp` in
Building 2), which makes the pair useful for testing cross-building
generalization. Values are as exported from the BAS: temperatures appear to
be in °F and airflow in cfm. 

## Quick Start

```python
import pandas as pd
from rdflib import Graph, Namespace, RDF

REF = Namespace("https://brickschema.org/schema/Brick/ref#")

g = Graph().parse("building2/building2_model.ttl")
df = pd.read_csv("building2/building2_15min_2023-01-01_to_2023-01-31.csv",
                 parse_dates=["Timestamp"], index_col="Timestamp")

# map each CSV column to its Brick class
brick_class = {str(g.value(s, REF.hasExternalReference)): str(g.value(s, RDF.type))
               for s in g.subjects(REF.hasExternalReference, None)}
zone_temps = [c for c in df.columns
              if brick_class[c].endswith("Zone_Air_Temperature_Sensor")]
print(df[zone_temps].describe())
```

## Intended Uses

- Semantic-model extraction and point tagging from point names and time series
- Equipment-relationship inference (e.g., AHU to VAV)
- Fault detection and diagnostics, and metadata-driven analytics
- Benchmarking control and optimization methods on real operational data

## Anonymization

Building names, addresses, tenant and organization names, and room names have been removed. Equipment and point labels in the Brick models use generic identifiers only (e.g., AHU1, VAV1_01). The datasets contain building automation system measurements only, with no occupant-identifying data. Time-series values are provided as exported from the BAS and have not been modified.

## Known Limitations

- Two buildings and short windows: 35 days (cooling season) and 31 days
  (heating season)
- Building 2 contains about 8% missing values, and its series ends before
  midnight on the last day
- No ground-truth fault labels
- Point units are not encoded in the models
- Brick models describe equipment and points only (no spatial or plant model)

## Related Publications

- Wang, Z., Ma, J., Cai, J., & Qu, M. (2025). Unsupervised inference of
  equipment relationships in building automation systems using LSTM-based
  feature importance. *Proceedings of the 12th ACM International Conference
  on Systems for Energy-Efficient Buildings, Cities, and Transportation*,
  252–256.
- Wang, Z., Ma, J., Cai, J., & Qu, M. (2026). Point2Brick: Automating
  building semantic model creation via multi-stage confidence-aware large
  language model agents. *Energy and Buildings*, 117516.

## License

Released under the
[Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) license.
You may share and adapt the data for any purpose, including commercially,
provided you give appropriate credit.

## Citation

Use GitHub's "Cite this repository" button (from `CITATION.cff`) and cite the
Point2Brick paper above.

## Contact

Jie Cai, Ray W. Herrick Laboratories, Purdue University (cai40@purdue.edu)
