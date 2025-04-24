
# 🌩️ Real-Time Weather Intelligence — Azure | Fabric | Event Hub | Databricks | Power BI

A cutting-edge data engineering project that captures **real-time weather data** from external APIs and streams it into **Microsoft Fabric** through **Azure Event Hubs**, **Azure Functions**, **Databricks**, and **Kusto DB**, visualized live in **Power BI**, and automated using **Data Activator**.

---

## 💡 Project Overview

This end-to-end streaming solution:
- Ingests real-time weather data using Azure Functions
- Sends data to Azure Event Hub
- Processes streaming data on Azure Databricks with Spark
- Feeds data into Microsoft Fabric (Event Stream + Kusto DB)
- Visualizes and alerts with Power BI and Data Activator

---

## 📐 Architecture

```mermaid
graph TD;
    A[Live Weather API] --> B[Azure Function App];
    B --> C[Azure Event Hub];
    C --> D[Azure Databricks Streaming Job];
    C --> E[Microsoft Fabric Event Stream];
    D --> F[Delta Lake Storage];
    E --> G[Fabric Real-Time Intelligence];
    G --> H[Kusto DB (Eventhouse)];
    H --> I[Power BI Dashboard];
    H --> J[Data Activator];
    J --> K[Microsoft Teams / Email Alerts];
    L[Azure Key Vault] --> B;
    M[Cost Management] --> C;
```

---

## 🔎 What is Microsoft Fabric?

**Microsoft Fabric** is a unified data platform that combines multiple services like:
- **Event Stream**: Entry point for streaming data into Fabric.
- **Real-Time Intelligence**: Component for processing and routing streaming data in near real-time.
- **Kusto DB (Eventhouse)**: Time-series database built for fast queries over telemetry data.
- **Power BI + Data Activator**: Dashboarding + real-time alerting engine.

Together, they provide scalable, low-latency analytics and automation.

---

## 🤖 Why These Tools?

| Tool                | Why it’s used                                                                 |
|---------------------|--------------------------------------------------------------------------------|
| **Azure Functions** | Serverless trigger-based API ingestion                                        |
| **Azure Event Hubs**| Managed event streaming layer                                                 |
| **Databricks**      | Scalable, real-time Spark processing                                          |
| **Fabric (Kusto DB)**| Real-time telemetry storage and analysis                                      |
| **Power BI**        | Interactive analytics                                                         |
| **Data Activator**  | Rule-based automation on top of live data                                     |
| **Key Vault**       | Secure API keys and connection strings                                        |

---

## 🧪 Azure Function Sample Code (Python)

```python
import requests, os
from azure.eventhub import EventHubProducerClient, EventData

def main(mytimer: func.TimerRequest) -> None:
    api_url = "https://api.weatherapi.com/v1/current.json"
    params = {"key": os.environ["WEATHER_API_KEY"], "q": "London"}
    res = requests.get(api_url, params=params).json()

    producer = EventHubProducerClient.from_connection_string(
        conn_str=os.environ["EVENT_HUB_CONN_STR"],
        eventhub_name="weather-stream"
    )
    batch = producer.create_batch()
    batch.add(EventData(str(res)))
    producer.send_batch(batch)
```

---

## 🔁 Databricks PySpark Streaming Snippet

```python
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, IntegerType

schema = StructType([
    StructField("location", StructType([
        StructField("name", StringType()),
        StructField("country", StringType())
    ])),
    StructField("current", StructType([
        StructField("temp_c", DoubleType()),
        StructField("humidity", IntegerType()),
        StructField("wind_kph", DoubleType())
    ]))
])

df = (spark.readStream
    .format("eventhubs")
    .option("eventhubs.connectionString", "<EVENT_HUB_CONN_STR>")
    .load()
    .selectExpr("cast(body as string) as json")
    .select(from_json(col("json"), schema).alias("data"))
    .select("data.*"))

df.writeStream.format("delta").outputMode("append").option("checkpointLocation", "/tmp/chk").start("/mnt/delta/weather_data")
```

---

## 🚨 Data Activator Use Case

**Triggering real-time alerts** from weather conditions:

- Condition: `temp_c > 38 OR wind_kph > 60`
- Action: "Send Teams message to emergency channel"

---

## 📊 Power BI Dashboard Insights

- Current temperatures and trends by region
- Max wind speeds & humidity anomalies
- Hourly weather patterns with auto-refresh
- Integration with Eventhouse as DirectQuery source

---

## 🛠️ Setup Instructions

1. Create Azure Function App and deploy ingestion code
2. Set secrets in Azure Key Vault (API key, Event Hub connection string)
3. Set up Azure Event Hub instance
4. Import notebook into Databricks and run with Spark cluster
5. Configure Microsoft Fabric Event Stream and Real-Time Intelligence
6. Connect Power BI to Eventhouse (Kusto DB) table
7. Create alert rules using Data Activator

---

## 🔐 Security Best Practices

- Always store secrets in **Azure Key Vault**
- Enable **Managed Identity** for secure access
- Monitor Event Hub throughput with **Cost Management**

---

## 📄 License

MIT License. Free to use, share, and build on.
