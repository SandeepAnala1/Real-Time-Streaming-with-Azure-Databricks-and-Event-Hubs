# Real-Time-Streaming-with-Azure-Databricks-and-Event-Hubs

---

## Project Overview
This project is about building a real-time analytics solution using:

- Azure Event Hubs (for streaming data)
- Azure Databricks (for processing the data)
- Power BI (for visualization)

## 🔷 **Solution Architecture Overview**
![image](https://github.com/SandeepAnala1/Real-Time-Streaming-with-Azure-Databricks-and-Event-Hubs/blob/main/Azure%20Solution%20Architecture.png)

### ✅ **Technologies Used**
- **Azure Event Hubs** – For ingesting real-time streaming data
- **Azure Databricks (with Unity Catalog)** – For data processing and storage using Spark Structured Streaming
- **Power BI (Desktop)** – For visualizing near real-time data (latency allowance: up to 20 minutes)

### ✅ **Data Flow**
1. **Data Generation**
   - Simulated JSON weather data will be generated using Event Hubs’ *Test Data feature*.
   
2. **Data Ingestion**
   - Data from Event Hubs is ingested using **Spark Structured Streaming** on Databricks.

3. **Data Lakehouse Storage**
   - Data is stored in a Unity Catalog-backed Lakehouse using the **Medallion Architecture**:
     - **Bronze Layer**: Raw data landing zone
     - **Silver Layer**: Cleaned and transformed data
     - **Gold Layer**: Aggregated and analytical-ready data

4. **Data Visualization**
   - Power BI accesses the **Gold Layer** via **Direct Query** for near real-time reporting.

---

## 📘 **Important Definitions**

### 🔹 **Azure Event Hubs**
A **big data streaming platform and event ingestion service** capable of handling millions of events per second. It’s ideal for telemetry, real-time analytics, and distributed data streaming.

---

### 🔹 **Key Components of Event Hubs**

| Component             | Description |
|----------------------|-------------|
| **Event Producers**  | Applications, devices, or services that send data to Event Hubs. |
| **Event Hub**        | The actual endpoint that receives streaming data from producers. |
| **Throughput Units** | Defines the capacity (per unit):<br> • Ingress: 1 MB/sec or 1,000 events/sec<br> • Egress: 2 MB/sec or 4,096 events/sec |
| **Partitions**       | Sub-divisions of an Event Hub to enable parallel processing and scalability. |
| **Namespace**        | A container that groups multiple Event Hubs. Acts as a billing and scoping unit. |
| **Consumer Groups**  | Allow multiple applications to independently consume and process the same data stream. |
| **Receivers**        | Applications that read data from the Event Hub using a consumer group. |

---

### 🔹 **Unity Catalog**
A unified governance solution for all data and AI assets in Databricks that allows for **fine-grained access control** and **centralized metadata management**.

---

### 🔹 **Medallion Architecture**
A **three-tiered** data refinement approach used in Lakehouse models:
- **Bronze Layer**: Ingested raw data.
- **Silver Layer**: Cleaned, validated, and transformed data.
- **Gold Layer**: Aggregated and business-ready data for analytics.

---

## ⚠️ **Important Tips**
- **Streaming jobs run indefinitely** unless manually stopped.
- **Databricks clusters do not auto-terminate** if streaming jobs are running.
- Always **stop streaming queries** and **shut down clusters** when finished to avoid extra charges.

Absolutely! Let’s break down the **Gold Layer processing** in **simple terms** and make **clear notes** for your understanding.

---

## 🪙 Gold Layer 

The **Gold Layer** is the final stage in the **Medallion Architecture** (Bronze → Silver → Gold). It provides **aggregated and refined data** that is ready for business use—like dashboards, reports, and analytics.

### 🔁 What’s Happening in the Gold Layer?

You're:
1. **Reading cleaned data** from the Silver table (already structured and parsed).
2. **Grouping it by time windows** (e.g., 5-minute intervals).
3. **Calculating averages** for temperature, humidity, wind speed, and precipitation.
4. **Handling late-arriving data** with a concept called **Watermarking**.
5. **Writing only complete windows** to the Gold table for stable reporting.

---

## 📝 Gold Layer
### ✅ 1. **Why group by time window?**
- To analyze trends (like average temperature every 5 minutes).
- Groups data into fixed periods (called **tumbling windows**).

---

### ✅ 2. **What does Watermark do?**
- **Waits for late data** before closing a window.
- Ensures **more accurate aggregations**.
- You specify it using `.withWatermark("timestamp", "5 minutes")`.

---

### ✅ 3. **How does it work?**
- Spark keeps track of the **latest event time seen**.
- It subtracts the watermark delay (e.g., 5 mins) from that latest event time.
- If that result is **past the end of a window**, Spark **closes the window** and outputs the result.

---

### ✅ 4. **What happens to very late data?**
- If data comes **after the watermark threshold**, it's **dropped** (not included in aggregation).
- This is to avoid keeping memory usage high indefinitely.

---

### ✅ 5. **How is it written?**
- You use `writeStream` with **output mode = append** (only finalized windows get written).
- Data goes into a Gold table, like `weather_summary`.

---

### ✅ 6. **Output Example**
Each row in the gold table has:
- **Start and end time** of the window.
- **Average values** of selected metrics.
- Clean and summarized data, perfect for **Power BI dashboards**.

---

### ✅ 7. **Why only finalized windows show up?**
- Spark only writes windows that are **closed** (after watermark threshold passed).
- So in Power BI or SQL, **you won’t see the very latest data** until its window is closed.

---

## 🖼️ Visual Summary

| Stage      | Action                                              | Output Type     |
|------------|-----------------------------------------------------|-----------------|
| **Silver** | Cleaned raw events                                   | One row per event |
| **Gold**   | Grouped by 5-min window & aggregated                 | One row per time window |


