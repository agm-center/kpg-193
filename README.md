## Citation

If you use the KPG 193 test system in your work, please cite:

```bibtex
@article{song2024kpg,
  title={KPG 193: A Synthetic Korean Power Grid Test System for Decarbonization Studies},
  author={Song, Geonho and Kim, Jip},
  journal={arXiv preprint arXiv:2411.14756},
  year={2024}
}
```

# KPG 193 Test System

![License: ODbL](https://img.shields.io/badge/License-ODbL_1.0-blue.svg)
![Version: v2.0.0](https://img.shields.io/badge/Version-v2.0.0-green.svg)
![Status: Active](https://img.shields.io/badge/Status-Active-brightgreen.svg)

![KPG 193 System Overview](kpg193.png)
*Network topology of the KPG 193 test system showing 193 buses across the Korean peninsula.*

## What is KPG 193?

KPG 193 is a **synthetic Korean power grid test system** developed by the [AGM Center](https://agm.kentech.ac.kr/) at [KENTECH](https://www.kentech.ac.kr/) from publicly available Korean energy data. It provides a standardized 193-bus representation of the Korean transmission network for reproducible power system studies.

KPG 193 is part of the [**KPG Test System**](https://agm.kentech.ac.kr/docs/kpg-test-system/) module in the [**KPG (Korean Power Grid) Platform**](https://agm.kentech.ac.kr/docs/introduction/). It is designed to work with [**KPG Run**](https://agm.kentech.ac.kr/docs/kpg-run/user-interface/) for ED, UC, DC-OPF, and AC-OPF studies, and with [**KPG View**](https://agm.kentech.ac.kr/docs/kpg-view/overview/) for map-based visualization.

The dataset supports research on decarbonization, optimal power flow, resource adequacy, reliability, and renewable energy integration. It is released under the Open Database License (ODbL 1.0) to support transparent and reproducible research.

## Key Features

- **MATPOWER-compatible**: MATPOWER v2 case files in `.m` and `.mat` formats.
- **Annual datasets for 2022–2025**: A separate 193-bus base system and hourly inputs for each year.
- **Network metadata**: Bus, line, conventional generator, hydro, and pumped-storage metadata.
- **Hourly profiles**: Demand, unit commitment, renewable generation ratios, and weather data for the corresponding year.
- **Hydro and pumped-storage inputs**: Unit and group metadata, capacity data, hourly output ratios, and daily group generation energy constraints.
- **Open and reproducible**: Released under ODbL 1.0 with documented data sources.

## System Overview

| Year | Buses | Conventional generators | AC branch rows | DC links | Hydro units | Pumped-storage units | Hourly periods |
|---|---:|---:|---:|---:|---:|---:|---:|
| 2022 | 193 | 183 | 724 | 1 | 41 | 16 | 8,760 |
| 2023 | 193 | 188 | 734 | 2 | 39 | 16 | 8,760 |
| 2024 | 193 | 193 | 742 | 2 | 39 | 16 | 8,784 |
| 2025 | 193 | 190 | 746 | 2 | 39 | 16 | 8,760 |

The AC branch count is the number of independent circuit rows in `mpc.branch`. The base system includes the highest voltage layer for each route. Conventional generators are represented in `mpc.gen`; hydro and pumped-storage units are supplied separately in metadata and profiles.

## Repository Structure

```text
├── kpg193_v2_0_0_2022/
├── kpg193_v2_0_0_2023/
├── kpg193_v2_0_0_2024/
├── kpg193_v2_0_0_2025/
├── README.md
├── LICENSE
└── kpg193.png
```

Each year folder has the following structure. File names containing `<year>` use that folder's year; daily profile files use the day of year.

```text
network/
├── m/kpg193_v2_0_0_<year>.m
├── mat/kpg193_v2_0_0_<year>.mat
└── metadata/
    ├── bus_metadata/
    ├── line_metadata/
    ├── generator_metadata/
    └── hydropower_metadata/
profile/
├── demand/demand_<day>.csv
├── commitment_decision/commitment_decision_<day>.csv
├── renewables/renewables_<day>.csv
└── weather/weather_<day>.csv
constraints/
└── hydropower_group_energy_constraints_<year>.csv
renewables_capacity/
├── solar_generators_<year>.csv
├── wind_generators_<year>.csv
└── hydropower_generators_<year>.csv
mustoff/
└── mustoff.csv
```

## Data Description

### Network Case Files

The `.m` and `.mat` files contain MATPOWER cases with `baseMVA = 100`. Conventional generator parameters and costs are in `mpc.gen`, `mpc.gencost`, and `mpc.genthermal`. AC circuits are in `mpc.branch`, and DC links are in `mpc.dcline`.

The initial operating values correspond to January 1, hour 1 of the relevant year. The case's `Pd` and `Qd` are net loads at that operating point, after renewable, hydro, and pumped-storage injections. Annual demand and generation profiles are provided separately for time-series studies.

### Metadata and Capacity

Bus and line metadata describe the network, and conventional generator metadata corresponds to `mpc.gen`. Hydro and pumped-storage metadata is in `hydropower_metadata/`, with separate unit and group tables. `unit_id` identifies a unit, and `group_id` links units to their group. These IDs belong to the corresponding year.

Solar and wind capacities are provided by bus. `hydropower_generators_<year>.csv` provides bus-level hydro and pumped-storage capacities, identified by `Type` (`hydro` or `psh`). Hydro and pumped-storage units are not additional rows in `mpc.gen`.

### Hourly Profiles

Each daily CSV contains hourly observations, with days numbered 1–365, or 1–366 for 2024.

| Folder | Contents |
|---|---|
| `profile/demand/` | Active and reactive demand by hour and bus: `demandP`, `demandQ` |
| `profile/commitment_decision/` | Unit status by hour and `generator_id` |
| `profile/renewables/` | Solar, wind, hydro, and pumped-storage ratios by hour and bus |
| `profile/weather/` | Longwave flux, temperature, wind components, and wind speed by hour and bus |

Commitment IDs 1–N correspond to the N conventional generator rows. IDs N+1–N+16 correspond to pumped-storage units in their metadata order.

For hydro and pumped-storage profiles, output in MW equals the profile ratio times the corresponding bus-level turbine `Pmax`. Pumped-storage ratios represent net injection (generation minus pumping) and may be negative, including below −1 when pump capacity exceeds turbine capacity.

### Energy Constraints and Outage Schedules

`hydropower_group_energy_constraints_<year>.csv` contains daily group generation energy limits in MWh. The group number in each column refers to `group_id` in that year's group metadata. Pumping power limits, efficiency, and storage energy limits are supplied in the hydro and pumped-storage metadata.

`mustoff/mustoff.csv` provides outage schedules for conventional generators, hydro units, and pumped-storage units.

## Getting Started

Clone this repository:

```bash
git clone https://github.com/agm-center/kpg-193.git
cd kpg-193
```

Load a case in MATLAB or Octave with MATPOWER on the path:

```matlab
addpath('kpg193_v2_0_0_2025/network/m');
mpc = kpg193_v2_0_0_2025;
```

Read CSV data in Python:

```python
from pathlib import Path
import pandas as pd

year = 2025
root = Path(f'kpg193_v2_0_0_{year}')
demand = pd.read_csv(root / 'profile/demand/demand_1.csv', encoding='utf-8-sig')
generators = pd.read_csv(
    root / f'network/metadata/generator_metadata/generator_metadata_{year}.csv',
    encoding='utf-8-sig',
)
```

## Data Sources

For the data construction methodology, see the accompanying paper [KPG 193: A Synthetic Korean Power Grid Test System for Decarbonization Studies](https://arxiv.org/abs/2411.14756).

## Version History

| Version | Updated | Major changes |
|---|---|---|
| **v2.0.0** | 2026-10-03 | Base-system datasets for 2022–2025; updated network cases, metadata, hourly profiles, and hydro/pumped-storage inputs |
| **v2.0.0-preview** | 2026-06 | Previous public 2025 dataset with system-demand-based profiles, refined generator parameters, and pumped-storage metadata |
| **v1.5** | 2025-12 | Generator locations corrected |
| **v1.4** | 2025-09 | Bus numbering standardized; clustering improved |
| **v1.3** | 2025-07 | Renewable profiles fixed |
| **v1.2** | 2025-02 | Generator costs and capacities updated |
| **v1.1** | 2024-12 | Slack bus changed |
| **v1.0** | 2024-11 | Initial public release |

The previous public dataset is available as [v2.0.0-preview](https://github.com/agm-center/kpg-193/releases/tag/v2.0.0-preview).

## Documentation

For documentation, tutorials, and additional resources, visit the [AGM Center Documentation](https://agm.kentech.ac.kr/docs/kpg-test-system/).

## License

The test system is made available under the [Open Database License (ODbL 1.0)](http://opendatacommons.org/licenses/odbl/1.0/).

## Contact

For questions or suggestions, contact: egolab.kpg@gmail.com
