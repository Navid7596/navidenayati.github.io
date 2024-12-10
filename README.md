# Memphis Blight and Litter 2024 – Final GIS Project

## Overview
The objective of this project is to investigate the **relationship between blighted areas** and **litter** in **Memphis, TN**, specifically for the year **2024**. The aim was to analyze the spatial distribution of **litter** and determine if areas with higher levels of **blight** are more prone to increased littering. This study involves the use of **geospatial analysis** techniques, primarily using **GeoPandas** and **ArcGIS Pro**, to explore correlations and visualize the spatial relationship between these two variables.

## Methodology

### Data Collection and Preparation
We utilized various datasets for **blighted areas** and **litter** (dumping locations), which were loaded from the **Memphis Data Hub** and **ArcGIS Pro**. The data was initially in a **geographic coordinate system** (WGS84), but to perform accurate analysis, it was reprojected into a **projected CRS** (EPSG:3857, Web Mercator). This re-projection was necessary for performing buffer analysis and other geometric operations.

```python
import geopandas as gpd

# Load the shapefiles for blighted areas and litter data
blighted_areas = gpd.read_file(r"C:\Users\simani\OneDrive - The University of Memphis\Attachments\Navid\OUTPUT\blighted_areas.shp")
litter_data = gpd.read_file(r"C:\Users\simani\OneDrive - The University of Memphis\Attachments\Navid\OUTPUT\litter_data.shp")

# Reproject to a projected CRS (EPSG:3857 for Web Mercator)
blighted_areas_projected = blighted_areas.to_crs(epsg=3857)
litter_data_projected = litter_data.to_crs(epsg=3857)
###
```
## Buffer Analysis
A buffer of 100 meters was created around each blighted area to determine the proximity of litter to these areas. Buffers were essential to examine how close litter was to the blighted areas, helping to visualize potential hotspots.
```python
# Create a buffer around each blighted area (100 meters) in the new CRS
blighted_areas_projected['buffer'] = blighted_areas_projected.geometry.buffer(100)

# Plot the original blighted areas and their buffers
import matplotlib.pyplot as plt
ax = blighted_areas_projected.plot(color='blue', alpha=0.5, edgecolor='black')
blighted_areas_projected['buffer'].plot(ax=ax, color='red', alpha=0.3)
plt.title("Blighted Areas with 100m Buffer")
plt.show()
```
![Buffer](https://github.com/user-attachments/assets/1b31e6d0-a32f-49f9-a2da-7db45ee6551f)

## Spatial Join
A spatial join was performed to identify litter points that fall within the blighted area buffers. This allowed us to assess how much of the litter overlapped with blighted areas.
```python
import geopandas as gpd

# Perform spatial join using 'within' operation to find litter points inside blighted area buffers
join_result = gpd.sjoin(litter_data_projected, blighted_areas_projected, how="inner", predicate="within")

# Display the first few rows of the joined data
print(join_result.head())
```
## Pearson Correlation Analysis
To quantify the relationship between blight and litter, a Pearson correlation was calculated. The result of this correlation indicated a weak negative correlation of -0.044, which suggested a weak relationship between litter and proximity to blighted areas.
```python
# Calculate Pearson correlation between litter count and proximity to blighted areas
correlation = join_result['geometry'].distance(blighted_areas_projected['buffer'].iloc[0]).corr(join_result['litter_id'])
print(f"Pearson correlation: {correlation}")
```
## Visualization
Visualizations were created using GeoPandas and ArcGIS Pro. Maps were generated showing:

Blighted Areas and their 100-meter buffers.
Litter Distribution overlaid on top of the blighted area buffers.
```python
# Plotting the litter points and their proximity to blighted areas using GeoPandas
ax = litter_data_projected.plot(color='green', alpha=0.5, edgecolor='black', label="Litter")
blighted_areas_projected.plot(ax=ax, color='blue', alpha=0.3, edgecolor='black', label="Blighted Areas")
plt.title("Litter and Blighted Areas in Memphis (2024)")
plt.legend()
plt.show()
```
![Spatial join](https://github.com/user-attachments/assets/2f390777-a018-4cbc-8f7a-b1140b608258)

ArcGIS Pro vs. Python Analysis
In ArcGIS Pro, we observed that around 84% of the litter occurred within blighted areas. This high percentage suggests that the relationship between blight and litter is strong at the local scale, where most litter tends to accumulate near blighted areas.
However, in the Python-based analysis, the Pearson correlation was much lower (-0.044), indicating a weak correlation.
![Picture1](https://github.com/user-attachments/assets/5dc7dafa-9563-4a47-a795-0b35510d818a)

## Why the Discrepancy?
The Pearson correlation from the Python code suggests a weak relationship between blight and litter, which could be attributed to several factors:

Spatial aggregation: The Pearson correlation analysis in Python might have averaged out the spatial variation across larger regions, while ArcGIS Pro's analysis likely focused on more localized areas.
Data granularity: ArcGIS Pro might have used more specific, localized data or spatial units (e.g., smaller blocks or parcels), which would capture the intensity of litter in blighted areas more effectively than a global statistical measure like Pearson's correlation.
Buffer Size: The buffer size of 100 meters used in the Python analysis may not have been optimal to detect a stronger relationship. Smaller or larger buffer sizes might produce different results.
Different methodologies: ArcGIS Pro and Python use different techniques for spatial analysis. ArcGIS Pro's tools for spatial analysis are optimized for GIS operations, which may yield more spatially sensitive results than the Python approach using GeoPandas.
## Conclusion and Next Steps
In conclusion, the project successfully visualized and analyzed the relationship between blight and litter in Memphis using both Python and ArcGIS Pro. While the ArcGIS Pro analysis showed a high percentage of litter occurring within blighted areas (84%), the Pearson correlation analysis in Python showed a much weaker correlation (-0.044). The discrepancy could be due to the difference in spatial scales and methodologies between the two approaches.
