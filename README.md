# LiDAR Vegetation Filter Workflow (WilCo 2024 Example)

This repository contains a Python script for processing LiDAR data tiles.  
It demonstrates how to:

1. **Download** LiDAR tiles (zipped LAZ files).  
2. **Extract** LAZ files.  
3. **Convert** LAZ → LAS using [PDAL](https://pdal.io).  
4. **Clip** LAS tiles to city boundaries.  
5. **Extract vegetation** points (classification = 5, high vegetation).

The workflow is based on the 2024 StratMap LiDAR project for Williamson & Hays Counties (TX), but can be adapted to any LiDAR source.

---

## Requirements

- Python 3.8+  
- [PDAL](https://pdal.io) with LAZ support enabled (LASzip).  
- Python libraries:  
    ```bash
    pip install geopandas shapely requests
    ```

## Recommended installation with conda (ensures PDAL + dependencies):
```
conda create -n lidar python=3.11
conda activate lidar
conda install -c conda-forge pdal python-pdal geopandas shapely requests
```

## Usage
1. **Download and Extract LAZ**
The script downloads a list of LAZ tiles hosted by Williamson County GIS, saves them locally, and extracts the contents to a `lidar_raw_laz` folder.  
2. **Convert LAZ to LAS**
Using PDAL pipelines, each `.laz` file is converted into an uncompressed `.las` file in `lidar_raw_las`.  
3. **Clip to City Limits**
* A city boundary GeoJSON is required (e.g., `City_Limits_-_Round_Rock.geojson`).  
* The script reads the CRS from the LAS files and reprojects the city boundary if necessary.  
* Each LAS tile is cropped using PDAL’s `filters.crop` with the city polygon.  
Clipped output is written to `lidar_clipped`.  
4. **Extract Vegetation (Classification 5)**
* PDAL’s `filters.range` selects points classified as `5` (high vegetation).  
* Each clipped LAS is filtered and written to `lidar_high_vegetation`.  

## Directory Structure  
After running, you’ll have something like:  
```
.  
├── lidar_raw_laz/          # downloaded & extracted LAZ files  
├── lidar_raw_las/          # converted LAS files  
├── lidar_clipped/          # LAS clipped to city limits  
├── lidar_high_vegetation/  # vegetation-only LAS  
├── City_Limits_-_Round_Rock.geojson  
└── lidar_processing.py     # main script  
```

## Example Snippet
Vegetation extraction step:
```
pipeline_def = {
    "pipeline": [
        {"type": "readers.las", "filename": str(las_file)},
        {"type": "filters.range", "limits": "Classification[5:5]"},
        {"type": "writers.las", "filename": str(output_las)}
    ]
}
pipe = pdal.Pipeline(json.dumps(pipeline_def))
pipe.execute()
```

## Notes
Replace the urls list with your own LiDAR dataset links.  
Check the classification codes for your dataset — here, 5 = High Vegetation.  
If you see PDAL errors like “LASzip not enabled”, reinstall PDAL with LAZ support.  
```
conda install -c conda-forge pdal python-pdal laszip
```

## Resources
* [WILCO LiDAR, Contour, and Orthoimagery Request](https://wilcomaps.wilco.org/vertigisstudio/web/?app=890fe4cc2634486ba1cd03a552c54aab)  
* [City Limits - Round Rock](https://geohub.roundrocktexas.gov/datasets/CORR::city-limits-round-rock-1/about)
