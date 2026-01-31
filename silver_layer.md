# Silver
Cleans and validates Bronze data, enforcing proper types.  
Parses timestamps, numeric fields, and spatial coordinates.  
Merges datasets with planning areas for spatial context.  
Outputs Silver CSVs ready for aggregation.  
Prepares enriched, analytics-ready data for Gold layer.

##Code:
```python
import pandas as pd
from pathlib import Path

# =========================
# PATHS
# =========================
BRONZE_DIR = Path(r"C:/Urban green and human comfort index/output")
SILVER_DIR = Path(r"C:/Urban green and human comfort index/silver")
SILVER_DIR.mkdir(parents=True, exist_ok=True)

# Load Bronze CSVs
greenery = pd.read_csv(BRONZE_DIR / "urban_greenery.csv")
human = pd.read_csv(BRONZE_DIR / "human_presence.csv")
planning = pd.read_csv(BRONZE_DIR / "planning_areas.csv")
weather = pd.read_csv(BRONZE_DIR / "weather.csv")

# =========================
# CLEANING
# =========================
# Greenery: ensure lat/lon are floats
greenery["latitude"] = pd.to_numeric(greenery["latitude"], errors="coerce")
greenery["longitude"] = pd.to_numeric(greenery["longitude"], errors="coerce")

# Human presence: make numeric
human["population"] = pd.to_numeric(human["population"], errors="coerce")
human["area_sq_km"] = pd.to_numeric(human["area_sq_km"], errors="coerce")
human["population_density"] = pd.to_numeric(human["population_density"], errors="coerce")

# Planning: ensure lat/lon floats
planning["latitude"] = pd.to_numeric(planning["latitude"], errors="coerce")
planning["longitude"] = pd.to_numeric(planning["longitude"], errors="coerce")

# Weather: convert timestamp to datetime
weather["timestamp"] = pd.to_datetime(weather["timestamp"], errors="coerce")

# =========================
# ENRICHMENT
# =========================
# Example: assign greenery to planning area via planning_area column if exists
silver_greenery = greenery.merge(planning[["planning_area", "subzone"]], on="planning_area", how="left")

# Example: combine human presence with planning area
silver_human = human.merge(planning[["planning_area", "subzone"]], on="planning_area", how="left")

# =========================
# SAVE SILVER
# =========================
silver_greenery.to_csv(SILVER_DIR / "silver_greenery.csv", index=False)
silver_human.to_csv(SILVER_DIR / "silver_human_presence.csv", index=False)
planning.to_csv(SILVER_DIR / "silver_planning_areas.csv", index=False)
weather.to_csv(SILVER_DIR / "silver_weather.csv", index=False)

print("✨ Silver layer saved successfully in", SILVER_DIR)
```
