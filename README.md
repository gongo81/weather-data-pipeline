# weather-data-pipeline

# Data Engineering Project – Weather Forecast Pipeline for Germany

This project demonstrates an automated data engineering pipeline built on Microsoft Fabric. It collects and transforms 7-day weather forecasts for all German state capitals using the Open-Meteo API. The final dataset is enriched and visualized in Power BI, enabling geographical and temporal insights.

## Project Overview

- Automated ingestion of daily weather forecast api data
- Medallion architecture with Bronze, Silver, and Gold layers
- Parameterized notebooks with dynamic start and end dates
- Daily automation using Microsoft Fabric Data Factory
- Power BI integration for data exploration and visualization

## Folder Structure

```
├── notebooks/
│   ├── Bronze_Notebook.ipynb          
│   ├── Silver_Notebook.ipynb          
│   └── Gold_Notebook.ipynb          
│
├── data/
│   ├── sample_bronze.json          
│   ├── sample_silver.csv             
│   └── sample_gold.csv             
│
├── powerbi/
│   └── powerbi_weather_map.png      
│
├── data_factory_visualization.png  
└── README.md                         
```


## Architecture and Data Flow

The project uses a medallion architecture as follows:

### Bronze Layer – Raw Ingestion

- Source: Open-Meteo API (https://open-meteo.com/)
- Retrieves 7-day forecasts for 16 German state capitals
- Stores unprocessed JSON files in a lakehouse folder
- Parameters passed: `start_date` (default: today), `end_date` (default: today + 6)

### Silver Layer – Cleaned and Structured

- Explodes daily arrays in the raw data for daily updated weather data
- Normalizes structure into columns:
  - city, latitude, longitude, date, temp_max, temp_min, temp_mean, rain_prob, weather_code
- Saves results to the `Silver_Layer` table

### Gold Layer – Enriched and Analyzable

- Adds human-readable weather descriptions from weather codes
- Adds time features: weekday, month, year
- Deletes overlapping forecast dates before each new load
- Final output table is `Gold_Layer`, used for Power BI

## Automation Using Microsoft Fabric

The project is orchestrated using Microsoft Fabric's Data Factory, which triggers the notebooks daily. It uses dynamic parameters for start and end date calculation:

- `start_date`: `@formatDateTime(utcNow(),'yyyy-MM-dd')`
- `end_date`: `@formatDateTime(addDays(utcNow(), 6), 'yyyy-MM-dd')`

This approach ensures that each run loads the most current 7-day forecast, overwriting the last six days and appending the newly forecasted day.

### Pipeline and Parameter Configuration

The screenshot below illustrates the parameterized pipeline setup and notebook chaining:

<img width="1802" height="797" alt="data_factory_visualization" src="https://github.com/user-attachments/assets/3b14dfa8-afaa-4aa5-b823-98ea075734a0" />

## Power BI Integration

The Gold Layer is connected to Power BI where the data is visualized using:

- **Interactive Slicer:** Select forecast dates to filter the data dynamically.
- **Table View:** Displays city, date, weekday, temperature mean, rain probability, and weather description.
- **Map Visualization:** Uses Azure Maps to show all German capital cities with pie charts representing:
  - Average temperature (`temp_mean`)
  - Rain probability (`rain_prob`)
  - Min/Max temperature
  - Most frequent weather description
- **Color Encoding:** Each weekday is represented by a different color for intuitive filtering.

An example map visualization is available under `powerbi/powerbi_weather_map.png`.

## Sample Data

You can explore example outputs from each layer under the `data/` folder:

- `sample_bronze.json`: Raw API JSON
- `sample_silver.csv`: Structured forecast records
- `sample_gold.csv`: Enriched records with temporal and descriptive information

## Setup Notes

This project runs entirely inside Microsoft Fabric.
