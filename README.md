# Flight Movement Analysis — Aircraft Tracks Around Leeds

Visualising and analysing aircraft movement over time from raw ADS-B position
reports, and characterising how traffic builds up and disperses around a
regional airport.

**Author:** LU ZIYI (MC652428)
**Course:** CISC7204 Data Science and Data Visualization — University of Macau
**Status:** completed

---

## Problem / research question

> **How does aircraft traffic behave over time in the airspace around Leeds, and
> can the raw position reports be reduced to a small number of interpretable
> descriptors of that traffic?**

Sub-questions:

1. How many distinct aircraft are observed, and how are reports distributed
   across them and across days?
2. What does the spatial footprint of the traffic look like when each aircraft's
   positions are joined into a track?
3. Which altitude and ground-speed bands dominate the observed traffic?

## Dataset source and usage permissions

| Item | Detail |
|---|---|
| File | `flight_tracking.csv` (33,766 rows × 12 columns) |
| Content | ADS-B position reports: `hex_ident`, `altitude(feet)`, `latitude`, `longitude`, `date`, `time`, `angle`, `distance(nauticalmile)`, `squawk`, `ground_speed(knotph)`, `track`, `callsign` |
| Coverage | 2017/09/11 – 2017/09/16 (six calendar days), airspace around Leeds, United Kingdom |
| Rows per day | 09/11: 1,961 · 09/12: 6,772 · 09/13: 7,166 · 09/14: 7,970 · 09/15: 8,644 · 09/16: 1,253 |
| Source | Course dataset distributed through UMMoodle for CISC7204 Tutorial 08, itself derived from open ADS-B flight-tracking data |
| Permissions | Used here **only** for this course's educational exercise. ADS-B data is publicly broadcast; no personal data, credentials, API keys or restricted datasets are included. The dataset is redistributed in this repository solely as course material. |
| Sensitive data | None. `hex_ident` is an aircraft transponder address, not a person; no crew or passenger information exists in the file. |

## Methods and tools

**Tools:** Python 3.14 (Anaconda), pandas, NumPy, matplotlib, SciPy, Jupyter Notebook.

**Pipeline:**

1. **Load** — `pandas.read_csv` into a DataFrame.
2. **Tidy** — rename `latitude`/`longitude` to `lat`/`lon`; combine the separate
   `date` and `time` columns into a single Unix `timestamp` (the `to_epoch`
   helper, with a fallback for unparseable rows).
3. **Profile** — count distinct aircraft, inspect missing values
   (`squawk` 1,987 nulls, `callsign` 2,582 nulls, `ground_speed` 1,071 nulls,
   `track` 1,206 nulls), and summarise the numeric distributions.
4. **Track construction** — group by `hex_ident` and order by `timestamp` to
   recover one polyline per aircraft.
5. **Temporal windowing** — implement a custom `TrackLayer` that selects the
   reports inside a rolling 180-second window (`timestamp > t AND timestamp <=
   t + 180`), colours each aircraft separately, advances `t` by one second and
   wraps at the end of the data — the same layer semantics the course tutorial
   specifies.
6. **Visualise** — render the layer frame by frame, plus a per-aircraft route
   coloured by time, and density summaries.

> **Note on libraries.** The course tutorial specifies `geoplotlib` for this
> layer. That package could not be installed or run in this environment (its
> `setup.py` uses `use_2to3`, removed in setuptools 58; it is a Python-2 era
> release; and `geoplotlib.show()` requires an interactive OpenGL window).
> The layer is therefore reproduced with **matplotlib**, keeping the tutorial's
> step numbering and logic. This is stated in the notebook.

## Key findings

All figures below are computed by the notebook from the 33,766 reports.

1. **Many aircraft, few reports each.** The file contains reports from **805
   distinct transponder addresses** (`hex_ident`) and **699 distinct callsigns**.
   Reports per aircraft are heavily skewed: median **18**, mean **41.9**, maximum
   **564**. 197 aircraft have ≥50 reports and 91 have ≥100, so only a minority of
   aircraft are observed for long enough to build a long track.

2. **Tracks are spatially structured, not noise.** Joining each aircraft's reports
   in `timestamp` order yields smooth, coherent polylines rather than a scatter
   cloud — compare panels (a) and (b) of `figures/flight_tracks_frame.png`, which
   plot exactly the same 33,766 points before and after the join. This is direct
   evidence that the reconstructed `timestamp` ordering recovers real flight
   paths. `figures/single_aircraft_route.png` isolates one continuous flight
   (192 reports over seven minutes) and colours it by elapsed time.

3. **The rolling window is the right abstraction.** A 180-second window holds a
   small, readable number of aircraft at any instant — for example **29 reports
   from 2 aircraft** in the first window. Incrementing `t` by one second makes
   traffic enter and leave the window smoothly, so the output reads as motion,
   which a single static scatter of all 33,766 points cannot convey.

4. **High-altitude traffic dominates; terminal traffic is a distinct minor mode.**
   Altitude spans 2,325–44,975 ft (mean 25,983 ft), and the distribution is
   strongly bimodal in band terms:

   | Altitude band | Reports | Share |
   |---|---:|---:|
   | 0–1,000 ft | 0 | 0.0% |
   | 1,000–5,000 ft | 3,566 | 10.6% |
   | 5,000–10,000 ft | 799 | 2.4% |
   | 10,000–20,000 ft | 4,803 | 14.2% |
   | above 20,000 ft | 24,598 | **72.8%** |

   Nearly three quarters of all reports are above 20,000 ft (cruise/overflight),
   with a secondary low-altitude cluster at 1,000–5,000 ft around the airport.
   Ground speed supports the same split: 85–611 kn, mean 382 kn.

5. **The airspace is quiet at times.** Because reception is irregular, some
   sampled windows contain no reports at all, so traffic cannot be treated as
   continuous.

### Limitation

**Coverage is incomplete and receiver-dependent.** ADS-B reception depends on
ground-station coverage and line of sight, so the dataset records *what was
received*, not every aircraft that was actually flying. Reports are irregularly
spaced in time, 2,582 rows lack a `callsign`, and per-aircraft counts range from
1 to 564. Consequently every count above is a **lower bound** on real traffic, and
no conclusion about total air traffic volume should be drawn from this dataset.
It covers six days at one location, so nothing here speaks to seasonality or
longer-term trends either.

## Repository contents

```
data-science-project/
├── flight_tracking_analysis.ipynb   executed notebook (all outputs present)
├── flight_tracking.csv              the dataset analysed
├── requirements.txt                 exact package versions
├── figures/
│   ├── flight_tracks_frame.png      raw reports vs the same reports joined into tracks
│   └── single_aircraft_route.png    one continuous flight, coloured by elapsed time
└── README.md
```

## Instructions for running the analysis

```bash
# 1. create an isolated environment (Python 3.11+)
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 2. install the exact dependencies
pip install -r requirements.txt

# 3. launch Jupyter and run the notebook top to bottom
jupyter notebook flight_tracking_analysis.ipynb
# then: Kernel -> Restart & Run All
```

The notebook reads `flight_tracking.csv` from the repository root, so no path
configuration is required. Running all cells reproduces every figure and every
printed statistic in this README.

## Licence and attribution

Coursework produced for CISC7204 (University of Macau). Dataset provided as
course material; reused here for educational purposes only.
