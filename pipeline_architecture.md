# Urban Green & Human Comfort Index – Pipeline Architecture Overview

The Urban Green & Human Comfort Index project implements a robust, end-to-end data engineering pipeline designed to transform raw urban, environmental, and human presence 
data into actionable, analytics-ready insights for city planning and sustainability analysis. The pipeline emphasizes reproducibility, scalability, and modularity,
enabling efficient ingestion, processing, and aggregation of diverse geospatial and temporal datasets.

⸻

## 1. Data Ingestion Layer

The pipeline begins by ingesting multiple heterogeneous data sources:\
	•	Urban Greenery Data: GeoJSON files representing parks, tree cover, and vegetation polygons.\
	•	Planning Area Boundaries: GeoJSON files delineating urban planning zones.\
	•	Human Presence Data: CSV files capturing pedestrian or population counts with timestamps and location identifiers.\
	•	Weather Data: JSON files with temporal weather observations per location.

Each data source is programmatically ingested via Python scripts using GeoPandas for geospatial data and Pandas for tabular CSV/JSON files.

⸻

## 2. Kafka-Based Producers & Topics

Data is then streamed in real-time using Apache Kafka, enabling decoupled and scalable ingestion:\
	•	Producers: Individual Python scripts serve as Kafka producers for each data source.\
	•	Metadata Included: Each message includes:\
	•	Coordinates (latitude, longitude)\
	•	Timestamps of observation or record creation\
	•	Source Identifiers to track dataset origin\
	•	Topics: Dedicated topics per source, e.g., greenery_topic, human_presence_topic, weather_topic, ensuring logical segregation and easier downstream consumption.

This design allows near real-time updates and efficient scaling as new data streams are added.

⸻

## 3. Bronze Layer – Kafka Consumer

The Bronze layer acts as the first consolidation point:\
	•	Functionality:\
	•	Kafka consumers subscribe to multiple topics and read event streams continuously.\
	•	Raw events are serialized into CSV files stored in a Bronze folder, maintaining the original schema.\
	•	Checkpointing is implemented to track read offsets and ensure exactly-once processing, preventing data loss or duplication.\
	•	Tools: Python with kafka-python or confluent-kafka for reliable consumption.

This layer ensures immutable, auditable raw data storage that can be replayed for debugging or pipeline reprocessing.

⸻

## 4. Silver Layer – Data Cleaning & Transformation

The Silver layer transforms Bronze data into a structured, analytics-ready format:\
	•	Cleaning & Type Casting:\
	•	Ensures numeric columns are correctly typed.\
	•	Converts timestamps to datetime objects with consistent timezone handling.\
	•	Spatial Joins:\
	•	Urban greenery and human presence data are spatially linked to planning areas using GeoPandas.\
	•	This produces enriched datasets associating each observation with a planning area.\
	•	Additional Processing:\
	•	Filtering invalid coordinates, removing duplicates, handling missing entries.\
	•	Ensures each dataset is ready for aggregation.

This layer demonstrates data wrangling, geospatial joins, and reproducibility, forming a solid foundation for advanced metrics computation.

⸻

## 5. Gold Layer – Aggregation & Analytics-Ready Output

The Gold layer merges cleaned Silver datasets to produce actionable insights:\
	•	Aggregations:\
	•	Total greenery area per planning area.\
	•	Percentage of green coverage relative to planning area size.\
	•	Population and population density computations.\
	•	Handling Missing Values: Imputation or exclusion based on context ensures accuracy.\
	•	Output: A unified, analytics-ready CSV file that powers dashboards and further visualizations.\
	•	Tools: Python (Pandas, GeoPandas) for processing; Power BI for dashboard visualization.

This stage highlights skills in data aggregation, feature engineering, and preparing datasets for BI consumption.

⸻

## 7. Key Data Engineering Skills Demonstrated
•	Real-time data ingestion & streaming via Kafka.\
•	Bronze/Silver/Gold pipeline design for reproducible and audit-ready processing.\
•	Geospatial data handling with GeoPandas (spatial joins, area computations).\
•	Data cleaning, type enforcement, and temporal alignment.\
•	Aggregation & metrics computation for analytics-ready outputs.\
•	Version control & modular development using Git.\
•	Integration with BI tools for dashboard visualization.


![dashboard](https://github.com/user-attachments/assets/1fa29583-9c0e-4ace-8fd5-a8e399a88872)

