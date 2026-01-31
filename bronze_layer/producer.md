# Producer
Streams raw urban datasets (greenery, human presence, planning areas, weather) into Kafka topics.  
Extracts key fields, computes centroids and population density.  
Serializes messages as JSON for downstream processing.  
Handles GeoJSON and CSV inputs with error checks.  
Ensures real-time ingestion for ETL pipeline continuity.

#Code:
```python
import os
import json
import pandas as pd
import geopandas as gpd
from kafka import KafkaProducer
from time import sleep
from pathlib import Path

# =========================
# KAFKA CONFIG
# =========================
KAFKA_BOOTSTRAP = "localhost:9092"
producer = KafkaProducer(
    bootstrap_servers=KAFKA_BOOTSTRAP,
    value_serializer=lambda v: json.dumps(v).encode("utf-8")
)

# =========================
# FOLDERS & FILE PATHS
# =========================
DATA_DIR = r"C:/Urban green and human comfort index/data"
GREENERY_FOLDER = os.path.join(DATA_DIR, "Greenery")
PRESENCE_FOLDER = os.path.join(DATA_DIR, "presence")
AREA_FILE = os.path.join(DATA_DIR, "area.geojson")
WEATHER_FOLDER = DATA_DIR  # JSON files are directly in data folder

# =========================
# GREENERY PRODUCER
# =========================
KAFKA_TOPIC_GREENERY = "urban_greenery"
for file in os.listdir(GREENERY_FOLDER):
    if file.endswith(".geojson"):
        filepath = os.path.join(GREENERY_FOLDER, file)
        try:
            gdf = gpd.read_file(filepath)
            # Determine column names for park
            park_col = None
            if "NAME" in gdf.columns:
                park_col = "NAME"
            elif "park_name" in gdf.columns:
                park_col = "park_name"
            else:
                print(f"⚠️ Skipping {file}, missing park name column")
                continue

            for _, row in gdf.iterrows():
                centroid = row.geometry.centroid
                event = {
                    "source": "urban_greenery",
                    "park_name": row.get(park_col, "Unknown"),
                    "latitude": centroid.y,
                    "longitude": centroid.x,
                    "geometry": row.geometry.wkt
                }
                producer.send(KAFKA_TOPIC_GREENERY, event)
                sleep(0.01)
            print(f"✅ Produced greenery from {file}")
        except Exception as e:
            print(f"❌ Failed {file}: {e}")

# =========================
# HUMAN PRESENCE PRODUCER
# =========================
KAFKA_TOPIC_PRESENCE = "human_presence"
for file in os.listdir(PRESENCE_FOLDER):
    if file.endswith(".csv"):
        filepath = os.path.join(PRESENCE_FOLDER, file)
        try:
            df = pd.read_csv(filepath)
            required_cols = ["PA", "AG", "Pop"]
            if not set(required_cols).issubset(df.columns):
                print(f"⚠️ Skipping {file}, missing required columns")
                continue

            for _, row in df.iterrows():
                try:
                    area_sq_km = float(row.get("AG", 1))
                    population = int(row.get("Pop", 0))
                except ValueError:
                    # skip rows with non-numeric values
                    continue

                event = {
                    "source": "human_presence",
                    "planning_area": row.get("PA", "Unknown"),
                    "population": population,
                    "area_sq_km": area_sq_km,
                    "population_density": round(population / area_sq_km, 2) if area_sq_km > 0 else 0
                }
                producer.send(KAFKA_TOPIC_PRESENCE, event)
                sleep(0.01)
            print(f"✅ Produced human presence from {file}")
        except Exception as e:
            print(f"❌ Failed {file}: {e}")

# =========================
# PLANNING AREAS PRODUCER
# =========================
KAFKA_TOPIC_AREA = "planning_areas"
try:
    gdf = gpd.read_file(AREA_FILE)
    required_cols = ["PLN_AREA_N", "SUBZONE_N"]
    if not set(required_cols).issubset(gdf.columns):
        print(f"❌ Area file missing required columns")
    else:
        for _, row in gdf.iterrows():
            centroid = row.geometry.centroid
            event = {
                "source": "planning_area",
                "planning_area": row["PLN_AREA_N"],
                "subzone": row["SUBZONE_N"],
                "latitude": centroid.y,
                "longitude": centroid.x,
                "geometry": row.geometry.wkt
            }
            producer.send(KAFKA_TOPIC_AREA, event)
            sleep(0.01)
        print("✅ Produced planning areas successfully")
except Exception as e:
    print(f"❌ Failed area producer: {e}")

# =========================
# WEATHER PRODUCER
# =========================
KAFKA_TOPIC_WEATHER = "weather"
weather_files = ["air-temperature.json", "rainfall.json", "relative-humidity.json", "wind-speed.json", "wind-direction.json"]

def load_json_safely(file_path):
    try:
        with open(file_path, "r", encoding="utf-8") as f:
            return json.load(f)
    except Exception as e:
        print(f"[SKIPPED] {file_path} -> {e}")
        return None

def parse_weather_json(json_data, source_file):
    rows = []
    if not json_data:
        return rows
    data = json_data.get("data", {})
    stations = data.get("stations", [])
    readings = data.get("readings", [])
    reading_type = data.get("readingType", "UNKNOWN")
    reading_unit = data.get("readingUnit", "UNKNOWN")

    station_lookup = {
        s["id"]: {
            "station_name": s.get("name"),
            "latitude": s.get("location", {}).get("latitude"),
            "longitude": s.get("location", {}).get("longitude"),
        }
        for s in stations
    }

    for reading in readings:
        timestamp = reading.get("timestamp")
        values = reading.get("data", [])
        for v in values:
            station_id = v.get("stationId")
            value = v.get("value")
            meta = station_lookup.get(station_id, {})
            event = {
                "source": "weather",
                "source_file": source_file,
                "timestamp": timestamp,
                "station_id": station_id,
                "station_name": meta.get("station_name"),
                "latitude": meta.get("latitude"),
                "longitude": meta.get("longitude"),
                "reading_type": reading_type,
                "reading_unit": reading_unit,
                "value": value
            }
            producer.send(KAFKA_TOPIC_WEATHER, event)
            sleep(0.01)
    print(f"✅ Produced weather data from {source_file}")

for wf in weather_files:
    file_path = os.path.join(WEATHER_FOLDER, wf)
    json_data = load_json_safely(file_path)
    parse_weather_json(json_data, wf)

# =========================
# FLUSH & CLOSE
# =========================
producer.flush()
producer.close()
print(" All producers finished successfully!")
```
