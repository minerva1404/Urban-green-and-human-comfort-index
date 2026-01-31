# 🌳 Urban Green & Human Comfort Index

## Project Overview
End-to-end pipeline to assess urban greenery and human comfort across Singapore’s planning areas. Aggregates multiple datasets—including urban greenery, human presence, weather, and planning areas—into a unified Gold dataset ready for visualization and analysis.

This project demonstrates **real-world data engineering skills**: Kafka-based ingestion, spatial processing with GeoPandas, metric aggregation, and analytics-ready outputs.

---

## Project Description
- Ingests raw data from multiple sources (GeoJSON, CSV, API-ready formats)
- Streams data via **Kafka Producer → Consumer** architecture
- Performs **spatial joins** to map greenery and human presence to planning areas
- Computes **key metrics per planning area**:
  - Total greenery area (m²)
  - Greenery coverage (%)
  - Population and population density
- Outputs a **Gold CSV** for downstream analytics and Power BI dashboards
- Supports scalable, repeatable ETL workflows for urban spatial data

---

## Tech Stack
- **Python 3.x** – Data processing and ETL
- **GeoPandas / Shapely** – Spatial processing
- **Pandas** – Tabular aggregation
- **Kafka** – Real-time/batch data ingestion
- **CSV** – Analytics-ready output
- **Power BI** – Dashboarding & visualization
- **Windows / Cross-platform**

---

## Key Features
- Integration of heterogeneous urban datasets
- Kafka Producer/Consumer pipeline for data orchestration
- Spatial joins to associate greenery polygons with planning areas
- Automatic computation of urban metrics (greenery coverage, population density)
- Gold dataset ready for **mapping and dashboard visualization**
- Easily extendable to include additional urban datasets

---

## Installation & Setup
```bash
# Clone repository
git clone <https://github.com/minerva1404/Urban-green-and-human-comfort-index>
cd urban-green-human-comfort-index
```

# Create Python virtual environment
python -m venv venv
### Windows
```
venv\Scripts\activate
```
### Linux / Mac
```
source venv/bin/activate
```

# Install dependencies
pip install -r requirements.txt

⚠️ Ensure Kafka broker is running and topics are created:
Topics: urban_greenery, human_presence, planning_areas, weather

⸻

Usage Examples

1️⃣ Producer

python producer.py

	•	Reads raw datasets
	•	Sends events to Kafka topics

2️⃣ Consumer

python consumer.py

	•	Subscribes to Kafka topics
	•	Aggregates and cleans data
	•	Produces Gold CSV output with metrics per planning area

3️⃣ Power BI Dashboard

•	Import gold_output_full.csv

•	Pre-built visuals:

•	Map Visual: Planning areas with greenery coverage

•	Bar Chart: Comparison of greenery area across areas

•	Card Visuals: Number of planning areas, average greenery coverage, population density

⸻

Output:

•	gold_output_full.csv – Analytics-ready dataset

•	Metrics include:

•	Planning area name/code

•	Total greenery area (m²)

•	Greenery coverage (%)

•	Population and population density

•	Geometry (WKT) for mapping

⸻

Notes

•	Designed for reproducible ETL workflows

•	Supports extension to additional urban datasets or smart city metrics

•	Gold dataset is Power BI-ready for visualization
