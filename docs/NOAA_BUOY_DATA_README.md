# NOAA Buoy Data Analysis Notebook

Comprehensive analysis of sea surface temperature (SST) and partial pressure of CO2 (pCO2) measurements from 7 NOAA OCADS buoys across different ocean regions.

## Overview

This notebook explores ocean chemistry data from the NOAA Ocean Carbon and Acidification Data System (OCADS). The analysis includes:
- **Time series visualization** showing SST and pCO2 trends over time for each location
- **Distribution analysis** using violin plots to show typical ranges and variability
- **Comparative analysis** across all locations using box plots
- **Geographic context** with locations sorted north-to-south by latitude

## Data Source

**NOAA OCADS Portal**: https://www.ncei.noaa.gov/products/ocean-carbon-acidification-data-system

The data comes from moored buoys that continuously measure ocean surface conditions. Each CSV file contains:
- Date and time stamps
- Sea Surface Temperature (SST) in °C
- pCO2 SW (saturated) in μatm
- Additional water chemistry parameters (salinity, pH, nutrients, etc.)

## Study Locations

Seven buoy sites provide geographic coverage across the Pacific Ocean and Atlantic coastal waters:

| Location | Typical Role |
|----------|--------------|
| **Bering Sea** | Cold arctic waters |
| **First Landing** | US East Coast |
| **Grey's Reef** | Washington State coast |
| **LA buoy** | Southern California |
| **La Push** | Washington State coast |
| **South Pacific** | Tropical Pacific |
| **Southern Cali** | Southern California |

## Notebook Structure

### 1. Data Exploration
- Inspects the data directory structure
- Counts CSV files per location
- Examines data format and column names
- Checks for missing values and data ranges

**Key findings:**
- NOAA uses `-999.0` to indicate missing data
- Date formats vary (MM/DD/YYYY vs MM/DD/YY)
- Data spans multiple years per location

### 2. Time Series Analysis
*Individual cells for each location (7 cells total)*

Scatter plot visualization showing:
- **Blue points**: Sea Surface Temperature (left y-axis)
- **Red points**: pCO2 (right y-axis)
- **White gaps**: Periods with missing data

This approach clearly shows data availability without interpolating across gaps.

### 3. Distribution Analysis
*Individual cells for each location (7 cells total)*

Side-by-side violin plots showing:
- **SST Distribution**: Temperature ranges, median, quartiles
- **pCO2 Distribution**: CO2 saturation levels

Summary statistics printed: mean and standard deviation for each variable.

### 4. Comparative Analysis

**Box plots across all locations** (2 subplots):
- Left: SST comparison (Blue palette)
- Right: pCO2 comparison (Red palette)

Shows which locations are warmest/coldest and have highest/lowest ocean acidification levels.

### 5. Geographic Sorting

Locations reordered **north-to-south by latitude** with coordinates displayed on plot axes.

**Required cells to run in order:**
1. **Coordinate Extraction** - Searches NOAA data for lat/lon columns, builds mapping
2. **Geographic Sorting** - Reorders plots by latitude, displays coordinates

## Data Cleaning Pipeline

Applied consistently to all location data:

```python
# Handle NOAA's missing value encoding
data['SST (C)'] = data['SST (C)'].replace(-999.0, np.nan)
data['pCO2 SW (sat) uatm'] = data['pCO2 SW (sat) uatm'].replace(-999.0, np.nan)

# Parse mixed date formats
data['Date'] = pd.to_datetime(data['Date'], format='mixed')

# Sort chronologically
data = data.sort_values('Date')

# For distributions: remove NaN for clean comparisons
data_clean = data.dropna(subset=['SST (C)', 'pCO2 SW (sat) uatm'])
```

## How to Run

### Prerequisites
- Python 3.8+
- pandas, numpy, matplotlib, seaborn
- Jupyter/VS Code notebook environment

### Setup
1. Ensure data files are in: `./Data/noaa water chem data/[location]/`
2. Each location folder should contain CSV files from NOAA
3. CSV files must have identical headers with `Date`, `SST (C)`, `pCO2 SW (sat) uatm`, `Latitude`, `Longitude`

### Execution
- Run cells sequentially from top to bottom
- Time series and distribution cells can run independently after data exploration
- **Important**: Run "Coordinate Extraction" cell before "Geographic Sorting" cell

## Key Insights

The analysis enables investigation of:
- **Seasonal patterns**: Do SST and pCO2 vary predictably through the year?
- **Geographic trends**: Are warmer locations also more acidified?
- **Data availability**: Which locations have complete vs. sporadic measurements?
- **Spatial variance**: How do ocean conditions differ between regions?

## Troubleshooting

**NameError: 'location_coords' not defined**
- Ensure the Coordinate Extraction cell runs before Geographic Sorting cell

**Date parsing errors**
- The `format='mixed'` parameter handles both MM/DD/YY and MM/DD/YYYY
- Check if date columns have unexpected formatting

**Missing data columns**
- Verify latitude/longitude columns exist in CSV files
- NOAA data should include these columns by default

## Extended Analysis Ideas

- Correlation between SST and pCO2 at each location
- Seasonal decomposition of time series
- Trend analysis (are conditions changing over time?)
- Outlier detection for anomalous measurements
- Integration with other oceanographic parameters available in the data

## References

- NOAA OCADS: https://www.ncei.noaa.gov/products/ocean-carbon-acidification-data-system
- Data Download: https://www.ncei.noaa.gov/data/oceans/ocads/data/0156562/
- Ocean Acidification: https://www.usgs.gov/faqs/what-ocean-acidification

---

**Last Updated**: February 2026  
**Status**: Analysis template - ready for expansion and interpretation
