# OlmosCity: Medellín Landslide Risk Analysis

## Project Overview

**OlmosCity** is a data-driven geospatial analysis project focused on assessing and visualizing landslide risk in Medellín, Colombia. The project combines population density, historical landslide events, topographic data, and precipitation patterns to provide integrated risk assessment and prevention strategies.

This initiative was developed as part of the **NASA Space Apps Challenge 2025** and is a finalist in the **Gen N Gen Eco** category.

---

## Key Features

- **Landslide Risk Mapping**: Creates normalized risk scores (0-1) combining multiple risk factors
- **Spatial Analysis**: Grid-based analysis at 10×10 resolution for regional risk assessment
- **Statistical Analysis**: Comprehensive descriptive statistics and correlation analysis
- **Geospatial Processing**: Handles EPSG:4326 (geographic) and EPSG:3857 (projected) coordinate systems
- **Interactive Visualization**: Folium heatmaps for web-based exploration
- **Automated Risk Propagation**: Cellular automaton approach for spatial risk diffusion
- **GeoTIFF Export**: High-resolution raster output (500m resolution, LZW compressed)

---

## Project Structure

```
OlmosCity/
├── README.md                                          # This file
├── requirements.txt                                   # Python dependencies
├── nasa.ipynb                                         # Main refactored landslide analysis
├── OlmosCity_V1.ipynb                                # Original analysis version
├── integrated_analysis.ipynb                          # Statistical integration workflow
├── content/                                           # Data files
│   ├── col_pd_2020_1km_ASCII_XYZ.csv                 # Population density grid (1km cells)
│   ├── Colombia+Antioquia_Monthly.csv                # Monthly precipitation data
│   └── Inventario_de_movimientos_en_masa_*.csv       # Historical landslide inventory
├── outputs/                                           # Generated analysis outputs
│   ├── medellin_risk_map.tif                         # Risk raster (GeoTIFF)
│   ├── risk_distribution.png                         # Risk value histogram
│   ├── medellin_risk_heatmap.png                     # Spatial heatmap
│   ├── 01_distributions.png                          # Data distribution plots
│   ├── 02_spatial_analysis.png                       # Spatial analysis maps
│   └── population_heatmap.html                       # Interactive Folium map
└── evidencia/                                         # Reference links
    └── LinksInteres.txt                              # Project links
```

---

## Data Sources

### 1. **Population Data** (`col_pd_2020_1km_ASCII_XYZ.csv`)

- **Source**: Colombian population grid 2020 (1km × 1km cells)
- **Format**: X (longitude), Y (latitude), Z (population density)
- **Coverage**: Medellín metropolitan area

### 2. **Landslide Inventory** (`Inventario_de_movimientos_en_masa_*.csv`)

- **Type**: Historical landslide events with coordinates
- **Attributes**: Location (x, y), event type, date information
- **Use**: Historical hazard analysis and proximity-based risk calculation

### 3. **Precipitation Data** (`Colombia+Antioquia_Monthly.csv`)

- **Type**: Monthly precipitation records for Antioquia department
- **Format**: Wide table with years as columns
- **Use**: Climate risk factor in combined risk assessment

### 4. **Digital Elevation Model (DEM)**

- **Reference**: ASTGTMV003 30m resolution (optional input)
- **Use**: Slope calculation for terrain-based risk assessment

---

## Installation & Setup

### Prerequisites

- Python 3.8+
- pip or conda package manager

### 1. Clone or download the repository

```bash
cd OlmosCity
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Verify installation

```bash
python -c "import geopandas, rasterio, folium; print('✓ All core packages installed')"
```

---

## Core Dependencies

| Package          | Version | Purpose                          |
| ---------------- | ------- | -------------------------------- |
| **geopandas**    | 1.1.1   | Geospatial data handling         |
| **rasterio**     | 1.4.3   | GeoTIFF I/O operations           |
| **scipy**        | 1.16.2  | Interpolation & spatial analysis |
| **scikit-learn** | 1.7.2   | Data normalization               |
| **folium**       | 0.20.0  | Interactive web maps             |
| **matplotlib**   | 3.10.6  | Static visualizations            |
| **seaborn**      | 0.13.2  | Statistical plots                |
| **pandas**       | 2.3.3   | Data manipulation                |
| **numpy**        | 2.3.3   | Numerical computing              |

---

## Workflow & Usage

### Quick Start: Run Complete Analysis

```bash
jupyter notebook nasa.ipynb
```

Then execute cells sequentially:

1. **Configuration** - Set CRS, bounds, and output paths
2. **Data Loading** - Import population, landslides, precipitation
3. **Preprocessing** - CRS transformations and validation
4. **Slope Calculation** - Create terrain grid from elevation data
5. **Risk Analysis** - Run LandslidePreventionAutomaton
6. **Export** - Generate GeoTIFF and visualizations

### Running Individual Notebooks

#### `nasa.ipynb` (Recommended)

Refactored, production-ready analysis with:

- Centralized CRS handling (EPSG:3857 for metrics, EPSG:4326 for display)
- Modular risk calculation pipeline
- Efficient raster export with adaptive resolution
- Comprehensive error handling

**Key Output**: `medellin_risk_map.tif` (GeoTIFF raster)

#### `integrated_analysis.ipynb`

Statistical deep-dive including:

- Descriptive statistics for all variables
- Normality testing (Shapiro-Wilk)
- Correlation analysis (Pearson r)
- 95% confidence intervals
- Grid-based spatial clustering

**Key Output**: `01_distributions.png`, `02_spatial_analysis.png`

#### `OlmosCity_V1.ipynb`

Original analysis version (reference implementation)

---

## Risk Calculation Pipeline

### LandslidePreventionAutomaton Class

The core risk engine combines multiple factors:

```python
automaton = LandslidePreventionAutomaton(
    population_gdf,      # Population points
    historical_gdf,      # Historical landslide locations
    slope_gdf,          # Elevation/slope data
    precip_df           # Precipitation records
)
risk_gdf, alerts = automaton.run()
```

#### Risk Factors & Weights

| Factor                   | Weight | Calculation                                      |
| ------------------------ | ------ | ------------------------------------------------ |
| **Slope**                | 50%    | Normalized mean slope from k-NN neighbors        |
| **Historical Proximity** | 25%    | Exponential decay with distance (500m influence) |
| **Precipitation**        | 15%    | Categorical (low/moderate/high/critical)         |
| **Elevation**            | 10%    | Min-max normalization of Z values                |

#### Risk Levels

- **Low** (0.00 - 0.25): Minimal hazard
- **Moderate** (0.25 - 0.45): Moderate concern
- **High** (0.45 - 0.65): Significant risk
- **Critical** (0.65 - 1.00): Immediate hazard (alert threshold)

#### Spatial Propagation

Risk values propagate locally across 3 iterations with 150m influence radius, simulating cellular automaton behavior to account for spatial correlation.

---

## Output Files

### Raster Output

- **`medellin_risk_map.tif`**: Single-band GeoTIFF
  - CRS: EPSG:3857 (Web Mercator, meters)
  - Resolution: 500m (adaptive for large datasets)
  - Compression: LZW
  - Data type: Float32 (values 0-1)

### Visualizations

- **`risk_distribution.png`**: Histogram of risk values (30 bins)
- **`medellin_risk_heatmap.png`**: Spatial heatmap visualization
- **`01_distributions.png`**: Data distributions (6 subplots)
- **`02_spatial_analysis.png`**: Spatial analysis maps
- **`population_heatmap.html`**: Interactive Folium map (zoomable, explorable)

### Alerts

- High-risk zones (risk ≥ 0.65) with coordinates, risk scores, and slope values
- Format: GeoDataFrame with columns: `lat`, `lon`, `risk_current`, `risk_level`, `slope_mean`

---

## Coordinate Reference Systems

The project uses two CRS strategically:

| CRS           | Name                     | Use Case                                       |
| ------------- | ------------------------ | ---------------------------------------------- |
| **EPSG:4326** | WGS84 (Geographic)       | Maps, GeoJSON, Folium visualization            |
| **EPSG:3857** | Web Mercator (Projected) | Distance calculations, gridding, raster export |

**Medellín Bounding Box** (lat/lon):

- Latitude: 6.14722° to 6.38402° N
- Longitude: -75.74590° to -75.44199° W

---

## Key Functions & Classes

### LandslidePreventionAutomaton

```python
# Main methods:
.compute_slope_features(k_neighbors=8)      # K-NN slope analysis
.compute_historical_risk(influence_meters=500)  # Distance decay
.compute_precipitation_risk()                # Climate factor
.combine_risks()                             # Weighted combination
.propagate_risk(iterations=3, radius_m=150) # Spatial diffusion
.classify_zones()                            # Label risk levels
.generate_alerts(threshold=0.65)            # Extract high-risk zones
.run()                                       # Execute full pipeline
```

### Export Functions

```python
export_risk_geotiff(risk_gdf, out_path, resolution=500, smooth_sigma=1.5)
plot_risk_distribution(risk_gdf, out_png)
create_folium_heatmap(risk_gdf, center, zoom_start)
```

### Preprocessing Utilities

```python
reproject_to_projected(gdf)              # → EPSG:3857
reproject_to_geographic(gdf)             # → EPSG:4326
get_projected_bounds(latlon_bounds)      # Convert bounds
create_slope_grid(points_gdf, grid_resolution=100, buffer_m=500)
```

---

## Performance Considerations

### Memory Optimization

- **Large datasets (>1M points)**: Resolution automatically increases to 1000m
- **Medium datasets (>100k)**: Resolution ≥ 500m recommended
- **Interpolation method**: Linear griddata (cubic for smaller datasets)
- **Adaptive smoothing**: Gaussian filter (σ=1.5) to reduce noise

### Processing Time

- Population data loading: ~1-2 seconds
- Risk calculation: ~5-15 seconds (depends on point count)
- GeoTIFF export: ~10-30 seconds (including smoothing)
- Visualization generation: ~5-10 seconds

---

## Examples & Use Cases

### 1. Generate Risk Map

```python
# Load data
pop_gdf = gpd.read_file('population.gpkg')
hist_gdf = gpd.read_file('landslides.gpkg')

# Run analysis
automaton = LandslidePreventionAutomaton(pop_gdf, hist_gdf, slope_gdf, precip_df)
risk_gdf, alerts = automaton.run()

# Export
export_risk_geotiff(risk_gdf, 'risk_map.tif')
```

### 2. Identify High-Risk Zones

```python
high_risk = alerts[alerts['risk_level'] == 'Critical']
print(f"Found {len(high_risk)} critical zones")
# Deploy resources to coordinates in high_risk[['lat', 'lon']]
```

### 3. Create Interactive Map

```python
m = create_folium_heatmap(risk_gdf)
m.save('medellin_risks.html')
# Open in web browser for exploration
```

### 4. Statistical Analysis

```python
from integrated_analysis import compute_descriptive_stats
stats = compute_descriptive_stats(risk_gdf['risk_current'].values, "Risk Score")
```

---

## Project Recognition

- 🌍 **NASA Space Apps Challenge 2025** - Participant
- 🏆 **Gen N Gen Eco Category** - Finalist
- 🎙️ **El Colombiano Podcast** - Featured in Gen N finalist series

### Reference Links

- [NASA Space Apps Challenge Team Profile](https://www.spaceappschallenge.org/2025/find-a-team/olmoscity/?tab=details)
- [Project Presentation (Canva)](https://www.canva.com/design/DAG08naFj3E/qZfy-PG2KJdl2gw5SdO0zg/view?utm_content=DAG08naFj3E&utm_campaign=designshare&utm_medium=link&utm_source=viewer)
- [Podcast Feature (Spotify)](https://open.spotify.com/episode/1xrrwtsJPh7QQv0tRn29XD?si=f7BvE7M-QxOg9f3-88gc5w&pi=__BOJp_FRpKSt&nd=1&dlsi=303fe34f9d5f4a8c)

---

## Troubleshooting

| Issue                            | Solution                                                                    |
| -------------------------------- | --------------------------------------------------------------------------- |
| `ModuleNotFoundError: geopandas` | Run `pip install -r requirements.txt`                                       |
| Empty GeoDataFrame after loading | Verify CSV has X,Y or x,y columns; check file encoding (UTF-8)              |
| GeoTIFF export fails             | Ensure `outputs/` directory exists; check disk space                        |
| Folium map not displaying        | Use `m.save('output.html')` and open in browser; check internet for tiles   |
| Slow rasterization               | Reduce `resolution` parameter; increase `grid_resolution` in slope function |
| NaN values in risk map           | Filter data: `gdf = gdf[gdf.geometry.is_valid]`                             |

---

## Contributing & Future Work

### Potential Enhancements

- [ ] Integration of satellite imagery (NDVI, slope from optical DEM)
- [ ] Machine learning model for risk prediction (Random Forest, XGBoost)
- [ ] Real-time precipitation data API integration
- [ ] Multi-hazard analysis (floods, earthquakes, landslides)
- [ ] Impact assessment (buildings, infrastructure at risk)
- [ ] Early warning system with push notifications

### Data Integration Opportunities

- Regional geological maps (Servicio Geológico Colombiano)
- Soil stability indices
- Infrastructure location data
- Historical precipitation patterns (longer time series)

---

## License & Citation

If you use OlmosCity in research or development, please cite:

```bibtex
@project{olmoscity2025,
  title={OlmosCity: Medellín Landslide Risk Analysis},
  author={OlmosCity Team},
  year={2025},
  url={https://github.com/yourusername/olmoscity},
  note={NASA Space Apps Challenge 2025 Finalist}
}
```

---

## Contact & Support

For questions, issues, or collaboration inquiries:

- 📧 Open an issue on the project repository
- 🌐 Visit the NASA Space Apps Challenge team profile
- 🎙️ Listen to our podcast feature on Spotify

---

## Acknowledgments

- **NASA Space Apps Challenge** for the platform and inspiration
- **Colombian Geographic Institute (IGAC)** for population grid data
- **National Disaster Risk Management Unit (UNGRD)** for landslide inventory
- **IDEAM** for precipitation data
- **USGS ASTER** for global elevation model access

---

**Last Updated**: December 2025  
**Status**: Active Development  
**Version**: 2.0 (Refactored)
