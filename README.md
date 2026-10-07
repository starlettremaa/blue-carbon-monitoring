# Blue Carbon & Mangrove Health Monitoring: Wadi El Gemal Case Study

**Team Leader:** Reham Abdulraouf  
**Country:** Egypt  
**Theme:** Ecosystem Health, Biodiversity & Blue Carbon  
**Challenge Track:** Arab Youth Space Hackathon - 813 Challenge  

---

## 1. Executive Summary
This Proof of Concept (PoC) provides an automated geospatial pipeline to monitor, map, and assess the vitality of coastal mangrove ecosystems in the Red Sea region (focused on Wadi El Gemal National Park, Egypt). By leveraging Sentinel-2 multispectral satellite imagery, the system computes the Normalized Difference Vegetation Index (NDVI) to delineate dense vegetation canopies from arid surroundings and screen coastal ecosystem health.

---

## 2. Business Use Case
* **Target Users:** Environmental protection agencies (e.g., Egyptian Environmental Affairs Agency - EEAA), national park rangers, and blue carbon project managers.
* **Decision Support:** Enables decision-makers to identify degraded mangrove stands, prioritize field inspection/reforestation efforts, and establish baseline measurements for blue carbon offset programs.
* **Current Gap:** Traditional manual field surveying is labor-intensive, logistically challenging in remote coastal zones, and lacks continuous temporal coverage.

---

## 3. Problem Statement
Coastal mangrove forests are exceptionally high-capacity carbon sinks (Blue Carbon), yet they face severe risks from climate change and coastal developments. In the Red Sea coastline, mangrove stands are highly localized and fragmented. Monitoring their health and spatial dynamics requires scalable, high-resolution satellite Earth Observation (EO) tools.

---

## 4. Datasets Used
* **Sentinel-2 L2A (Copernicus / Microsoft Planetary Computer):**
  * **Bands:** Band 4 (Red, 10m resolution) and Band 8 (Near-Infrared, 10m resolution).
  * **Area of Interest (AOI):** Wadi El Gemal Coastal Zone [35.03°E, 24.63°N, 35.12°E, 24.72°N].
  * **Processing Level:** Surface Reflectance (L2A), Cloud Cover < 5%.
  * **CRS:** EPSG:32636 (UTM Zone 36N).

---

## 5. Technical Approach
1. Catalog Query: Query STAC catalog via pystac_client and planetary_computer for cloud-free imagery.
2. Data Stacking & Reprojection: Use stackstac to stream raster assets into an xarray DataArray aligned with EPSG:32636 at 10m spatial resolution.
3. Spectral Index Calculation: NDVI = (NIR - Red) / (NIR + Red)
4. Visualization & Output Generation: Generate high-resolution spatial maps isolating high-NDVI vegetation clusters (0.3 <= NDVI <= 0.5) representing mangrove communities.

---

## 6. Installation

Requires Python 3.10+.

```bash
git clone [https://github.com/starlettremaa/blue-carbon-monitoring.git](https://github.com/starlettremaa/blue-carbon-monitoring.git)
cd blue-carbon-monitoring
python -m venv .venv
pip install -r requirements.txt
