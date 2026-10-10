# Blue Carbon & Mangrove Health Monitoring

**Arab Youth Space Hackathon 2026 · 813 Challenge**

**Team Leader:** Reham Abdulraouf · 
**Country:** Egypt
**Theme:** Ecosystem Health, Biodiversity & Blue Carbon
**Repository:** https://github.com/starlettremaa/blue-carbon-monitoring

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/starlettremaa/blue-carbon-monitoring/blob/main/notebooks/main_analysis.ipynb)

---

## 1. Title & Summary

A satellite-based **screening tool** that tracks mangrove vegetation condition and estimates blue carbon stock ranges, using Sentinel-2 imagery and a change-detection threshold calibrated on a reference site. It is a proof of concept (PoC) built on Nabq (Egyptian Red Sea) with Abu Dhabi (UAE) as a larger comparison site. Its job is to tell reserve managers **which patches to inspect first**, not to prove degradation or measure carbon loss.

## 2. Business Use Case

- **Who uses this:** Environmental protection agencies (e.g., the Egyptian Environmental Affairs Agency, EEAA), protected-area rangers, and blue carbon project teams.
- **Decision supported:** Which mangrove patches to visit first, and where field effort and protection should be prioritised.
- **What they use today:** Infrequent, costly field surveys in remote coastal zones, with no continuous coverage.
- **Why satellite data:** Sentinel-2 is free, revisits every ~5 days at 10 m, and allows routine, repeatable screening at low cost.
- **What this tool does not do:** It does not decide whether a site qualifies for carbon-credit programmes. That requires field measurements and a formal measurement, reporting and verification (MRV) methodology.

## 3. Problem Statement

Mangroves store large amounts of carbon in a small area. The Nabq stand alone covers 38.8 ha (ESA WorldCover class 95) and holds an estimated **1,670–2,619 t C** under literature-based density scenarios. Egypt's Red Sea mangroves (including Nabq and Wadi El Gemal) have no routine spectral monitoring, so vegetation stress is typically noticed only after it is visible on the ground.

For small, isolated stands like Nabq, a localised stress event can matter. A cheap, repeatable screening layer helps decide where a ranger or survey team should look.

## 4. Data Used

| Dataset | Provider | Dates | Processing level, resolution | Licence | Role |
|---|---|---|---|---|---|
| Sentinel-2 L2A | ESA Copernicus via Microsoft Planetary Computer | Jan–Mar 2019 and Jan–Mar 2025 | L2A surface reflectance; 10 m (Nabq), 20 m (Abu Dhabi and Nabq reference) | Free and open (Copernicus) | NDVI / NDRE composites |
| ESA WorldCover 2021 | ESA via Planetary Computer | 2021 | Land-cover map; 10 m | CC BY 4.0 | Mangrove mask (class 95) |
| Global Mangrove Watch v4 | JAXA / Zenodo | 1996–2024 | Vector extent | Open | Used only to select candidate sites, not in the pipeline |

- **Bands:** B04 (Red), B05 (Red-Edge), B08 (NIR), plus the SCL scene-classification layer.
- **Indices:** NDVI and NDRE (both reported in `results/summary.csv`).
- **Scene screening:** scene cloud cover < 20 %, then per-pixel SCL classes 4, 5, 6 (vegetation, bare soil, water) are kept. The Nabq composites use 15 scenes for 2019 and 15 for 2025; the counts are printed by the notebook and saved in `data/sample_input/params.json`.
- **Carbon densities (t C/ha):** low 43.0 (Red Sea soil only, 1 m; Almahasheer et al. 2017), mid 60.35 and high 67.44 (Qatar *Avicennia marina* stands, tree + soil; see `CARBON` in the notebook). These come from other sites and are not measured at the study areas.

## 5. Technical Approach

1. **Discover:** query the Planetary Computer STAC API for Sentinel-2 L2A scenes over each area of interest (AOI), filtered by date and cloud cover.
2. **Load and correct:** stream B04, B05, B08 and SCL with `stackstac`. Scenes acquired on or after 25 Jan 2022 carry a +1000 DN offset (processing baseline 04.00), which is subtracted before converting to reflectance.
3. **Inspect:** print B08 quantiles for both years and assert the median is in a plausible 0–0.7 reflectance range, as a radiometric sanity check.
4. **Analyse:**
   - Build Jan–Mar median composites for 2019 (baseline) and 2025 (recent); compute NDVI = (B08 − B04)/(B08 + B04) and NDRE = (B08 − B05)/(B08 + B05).
   - Mask to WorldCover class 95, then erode by one pixel to get a **core mask** that removes edge pixels.
   - Use Nabq (20 m, core mask) as a **reference site**: decline threshold = median change − 3 × MAD-sigma of its per-pixel NDVI change, instead of an arbitrary cut-off.
   - Report the share of core pixels below that threshold at each site.
5. **Communicate:** NDVI-change maps (PNG), summary tables (CSV), and a carbon range = mask area × literature density. "Exposed carbon" is the carbon under pixels showing a decline signal, which is **not** carbon that was lost.

## 6. Installation

Tested on **Python 3.13** (Google Colab). All package versions are pinned in `requirements.txt`.

```bash
git clone https://github.com/starlettremaa/blue-carbon-monitoring.git
cd blue-carbon-monitoring
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install jupyterlab           # only needed to open the notebook locally
```

No API keys or environment variables are needed. The analysis is deterministic (medians and thresholds, no random sampling), so no random seeds are required.

## 7. How to Run

```bash
jupyter lab notebooks/main_analysis.ipynb
```

In Jupyter choose **Kernel → Restart & Run All** (in Google Colab: **Runtime → Run all**). No credentials are needed: data comes from the public Planetary Computer STAC endpoint. Runtime was roughly 8–12 minutes in Colab and depends on network speed.

The notebook writes to:

- `results/`: `summary.csv`, `calibrated_threshold.csv`, `carbon_range.csv`, and the two change-map PNGs.
- `data/sample_input/`: Nabq 10 m mangrove mask and NDVI composites (GeoTIFF) plus `params.json` with the exact parameters used.
- `requirements.lock.txt`: exact package versions of the run.

The files already committed in `results/` come from the author's run. Change maps are exported as PNG only; GeoTIFFs are exported for the Nabq sample inputs, not for the change maps.

## 8. Example Input & Output

**Input:** Sentinel-2 L2A scenes are retrieved automatically via STAC. A small Nabq sample (10 m) is committed in `data/sample_input/`:

- `nabq_worldcover_mangrove_mask_10m.tif`: WorldCover class-95 mask
- `nabq_ndvi_2019_JanMar_10m.tif` and `nabq_ndvi_2025_JanMar_10m.tif`: the two NDVI composites
- `params.json`: exact site box, dates, cloud limit, SCL classes, scenes used, calibrated threshold

**Output:** NDVI change 2019 → 2025 inside the core mangrove mask. Blue = increase, red = decrease, white = little change.

Nabq, Egypt (10 m):

![Nabq NDVI change](results/nabq_egypt_10m_ndvi_change.png)

Abu Dhabi, UAE (20 m):

![Abu Dhabi NDVI change](results/abu_dhabi_ndvi_change.png)

## 9. Results & Limitations

### Results

Core-mask values. Nabq at 10 m, Abu Dhabi at 20 m. Source: `results/*.csv`.

| Site | Mangrove area (ha) | Carbon stock (t C) | Median NDVI 2019 → 2025 | Median per-pixel NDVI change | Median NDRE 2019 → 2025 | Core pixels below threshold |
|---|---|---|---|---|---|---|
| Nabq, Egypt | 38.8 | 1,670 – 2,619 | 0.590 → 0.597 | +0.012 | 0.355 → 0.340 | 1.4 % (3,121 px) |
| Abu Dhabi, UAE | 3,332 | 143,286 – 224,726 | 0.286 → 0.355 | +0.042 | 0.159 → 0.190 | 10.9 % (64,252 px) |

Calibrated decline threshold: −0.108 NDVI (Nabq median change 0.011 − 3 × 0.040, saved in `params.json`). Exposed carbon (carbon under pixels with a decline signal, across the −0.05 / calibrated / −0.20 thresholds): Nabq 0–161 t C, Abu Dhabi 6,462–29,674 t C. This is an exposure indicator, not a loss estimate.

**Nabq.** NDVI is slightly higher in 2025 and only 1.4 % of core pixels fall below the threshold, so no strong NDVI decline signal is seen. Note that NDRE moves the other way (−0.015), which is a small divergence worth checking with more years, and that the 1.4 % is measured on the same site the threshold was calibrated on, so it is not an independent validation.

**Abu Dhabi.** NDVI rose (median per-pixel change +0.042). 10.9 % of core pixels fall below the Nabq-derived threshold, and they are spatially clustered (see map), which makes them good candidates for inspection. However, Abu Dhabi's natural change spread (MAD-sigma 0.085) is about twice Nabq's (0.040) and its baseline NDVI is much lower (0.29 vs 0.59), so the Nabq threshold does not transfer cleanly. Treat 10.9 % as a ranking signal, not a stress rate. The cause of the increase (for example planting, tide state or phenology) is not tested here.

### Limitations

- Carbon values are literature proxies from the Red Sea and Qatar, not field-validated.
- One season (Jan–Mar) and two years only. This cannot separate real stress from phenology or year-to-year variation.
- Tide state at acquisition is not controlled and SCL class 6 (water) is retained, which can shift NDVI in mangrove areas and may explain part of any change.
- The decline threshold comes from a single small reference site (649 core pixels at 20 m) and is evaluated partly on that same site.
- The +1000 DN offset is applied by acquisition date (≥ 25 Jan 2022), not read from each scene's processing-baseline metadata.
- Abu Dhabi is processed at 20 m and Nabq at 10 m, so pixel-share figures are not strictly comparable.
- No field validation and no hyperspectral data in this PoC. Future work: multi-year same-season series, uncertainty intervals, tide-aware scene selection, and Red-Edge/hyperspectral sensors (e.g., EnMAP) for biochemical stress.

## 10. Team, Licence & Attribution

**Team Leader:** Reham Abdulraouf: project design, EO pipeline, analysis, documentation.

Built on:

- Sentinel-2 data: ESA Copernicus Programme
- STAC access: Microsoft Planetary Computer
- ESA WorldCover: ESA / Brockmann Consult / Google / Mundi Web Services
- Carbon density reference: Almahasheer et al. (2017), mangrove forests in the Arabian Peninsula
- Global Mangrove Watch v4: JAXA Earth Observation Research Center (site selection)

**Licence:** MIT (see `LICENSE`).

*Arab Youth Space Hackathon 2026 · 813 Challenge · Ecosystem Health, Biodiversity & Blue Carbon*
