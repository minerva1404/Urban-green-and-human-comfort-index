# Gold
Aggregates Silver data to compute urban metrics per planning area.  
Calculates total greenery, greenery coverage %, population, and density.  
Performs spatial joins to assign metrics accurately.  
Produces unified analytics-ready Gold CSV.  
Supports direct visualization in Power BI dashboards.

## Code:
```python
import os
import pandas as pd
import geopandas as gpd

# -------------------
# FILE PATHS
# -------------------
DATA_DIR = r"C:\Urban green and human comfort index\data"
GREENERY_FOLDER = os.path.join(DATA_DIR, "Greenery")
PRESENCE_FOLDER = os.path.join(DATA_DIR, "presence")
AREA_FILE = os.path.join(DATA_DIR, "area.geojson")
OUTPUT_CSV = os.path.join(DATA_DIR, "gold_output_full1.csv")

# -------------------
# LOAD AREA
# -------------------
area_gdf = gpd.read_file(AREA_FILE)
area_gdf["area_sq_m"] = area_gdf.geometry.area
area_gdf["area_sq_km"] = area_gdf["area_sq_m"] / 1_000_000
area_gdf["planning_area_clean"] = area_gdf["PLN_AREA_N"].str.strip().str.upper()

# -------------------
# LOAD GREENERY AND SPATIAL JOIN
# -------------------
greenery_list = []
for file in os.listdir(GREENERY_FOLDER):
    if file.endswith(".geojson"):
        path = os.path.join(GREENERY_FOLDER, file)
        gdf = gpd.read_file(path)
        gdf = gdf.to_crs(area_gdf.crs)  # Ensure same CRS for spatial join
        gdf["greenery_area_sq_m"] = gdf.geometry.area
        greenery_list.append(gdf[["geometry", "greenery_area_sq_m"]])

if greenery_list:
    all_greenery_gdf = pd.concat(greenery_list, ignore_index=True)
    all_greenery_gdf = gpd.GeoDataFrame(all_greenery_gdf, geometry="geometry", crs=area_gdf.crs)
    # Spatial join: which greenery polygon falls in which planning area
    joined = gpd.sjoin(all_greenery_gdf, area_gdf, how="left", predicate="intersects")
    greenery_df = joined.groupby("planning_area_clean", as_index=False)["greenery_area_sq_m"].sum()
else:
    greenery_df = pd.DataFrame(columns=["planning_area_clean", "greenery_area_sq_m"])

# -------------------
# LOAD HUMAN PRESENCE
# -------------------
presence_list = []
for file in os.listdir(PRESENCE_FOLDER):
    if file.endswith(".csv"):
        path = os.path.join(PRESENCE_FOLDER, file)
        df = pd.read_csv(path)
        # Rename columns to match
        if "PA" in df.columns:
            df.rename(columns={"PA": "planning_area"}, inplace=True)
        if "Pop" in df.columns:
            df.rename(columns={"Pop": "population"}, inplace=True)
        df["planning_area_clean"] = df["planning_area"].str.strip().str.upper()
        df["population"] = pd.to_numeric(df["population"], errors="coerce").fillna(0)
        presence_list.append(df[["planning_area_clean", "population"]])

if presence_list:
    presence_df = pd.concat(presence_list).groupby("planning_area_clean", as_index=False).sum()
else:
    presence_df = pd.DataFrame(columns=["planning_area_clean", "population"])

# -------------------
# MERGE ALL
# -------------------
gold_df = area_gdf.merge(greenery_df, on="planning_area_clean", how="left")
gold_df = gold_df.merge(presence_df, on="planning_area_clean", how="left")

# Fill missing values
gold_df["greenery_area_sq_m"] = gold_df["greenery_area_sq_m"].fillna(0)
gold_df["population"] = gold_df["population"].fillna(0)
gold_df["population_density"] = (gold_df["population"] / gold_df["area_sq_km"]).round(2)
gold_df["greenery_pct"] = ((gold_df["greenery_area_sq_m"] / gold_df["area_sq_m"]) * 100).round(2)

# -------------------
# SAVE OUTPUT
# -------------------
gold_df.to_csv(OUTPUT_CSV, index=False)
print(f"🎉 GOLD dataset saved: {OUTPUT_CSV}")
print(gold_df[["planning_area_clean", "greenery_area_sq_m", "greenery_pct", "population", "population_density"]].head())
```
