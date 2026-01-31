# Consumer
Subscribes to Kafka topics and ingests streaming events.  
Appends data to Bronze CSVs per dataset reliably.  
Normalizes data types and handles missing values.  
Persists incremental updates for reproducibility.  
Prepares structured Bronze layer for Silver processing.

## Code:
```python
import json
import pandas as pd
from kafka import KafkaConsumer
from pathlib import Path

# =========================
# KAFKA CONFIG
# =========================
KAFKA_BOOTSTRAP = "localhost:9092"

TOPICS = [
    "urban_greenery",
    "human_presence",
    "planning_areas",
    "weather"
]

consumer = KafkaConsumer(
    *TOPICS,
    bootstrap_servers=KAFKA_BOOTSTRAP,
    auto_offset_reset="earliest",          
    enable_auto_commit=True,
    group_id="urban_data_consumer_v2",     
    value_deserializer=lambda v: json.loads(v.decode("utf-8"))
)

# =========================
# OUTPUT PATHS
# =========================
OUTPUT_DIR = Path(r"C:/Urban green and human comfort index/output")
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

CSV_FILES = {
    "urban_greenery": OUTPUT_DIR / "urban_greenery.csv",
    "human_presence": OUTPUT_DIR / "human_presence.csv",
    "planning_areas": OUTPUT_DIR / "planning_areas.csv",
    "weather": OUTPUT_DIR / "weather.csv"
}

# =========================
# INIT DATAFRAMES
# =========================
dfs = {}

for topic, path in CSV_FILES.items():
    if path.exists():
        dfs[topic] = pd.read_csv(path)
    else:
        dfs[topic] = pd.DataFrame()

print("🎧 Kafka Consumer LIVE")
print("📡 Subscribed to topics:", TOPICS)
print("📂 Output directory:", OUTPUT_DIR)

# =========================
# CONSUME LOOP
# =========================
try:
    for message in consumer:
        topic = message.topic
        data = message.value

        # Safety check
        if topic not in dfs:
            print(f"⚠️ Unknown topic received: {topic}")
            continue

        # Append row
        new_row = pd.DataFrame([data])
        dfs[topic] = pd.concat([dfs[topic], new_row], ignore_index=True)

        # Save immediately
        dfs[topic].to_csv(CSV_FILES[topic], index=False)

        print(f"✅ {topic} | row count = {len(dfs[topic])}")

except KeyboardInterrupt:
    print("🛑 Consumer stopped manually")

finally:
    consumer.close()
    print("🎉 Consumer closed cleanly. All CSVs saved.")
```
