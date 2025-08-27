# realtime-analytics-pipeline

## Streaming Pipeline Using Dataflow: 
Real-time Air Quality Monitoring in Accra

**Source:** IoT sensors in cities stream air quality data (PM2.5, PM10, CO, NO2, O3 levels every few seconds)

- In this project I used Publish/Subscribe (Pub/sub) to BigQuery.
  
The Pub/sub template is a streaming pipeline that can read JSON-formatted messages from a Pub/Sub topic and write them to a BigQuery table 

**Pipeline:**  
- Pub/sub (raw messages) 
- Dataflow [Custom User Defined Function (UDF) stage: clean/transform/enrich] 
- BigQuery (analytics-ready table) 

**Dashboard:** Tableau showing city-level pollution heatmaps, alerts when thresholds are exceeded. 
Impact: Demonstrates environmental monitoring for smart cities 

- Pub/Sub topic in JSON format used;
{
  "station_id": "ST123",
  "city": “Accra”,
  "pm25": 32.5,
  "pm10": 65.0,
  "co2": "412",
  "timestamp": “2025-08-26T12:30:00Z,
}

IoT sensors (or the software that manages it, like Raspberry Pi, Arduino, or edge gateway) sends HTTP or gRPC request to Pub/Sub APIs. 
For this project I did not have a real-time streaming data from any IoT sensor so I used a local file as a streaming source (batch-to-stream trick): I wrote a python script to read the file line by line and publish each row into Pub/Sub (with a delay e.g., 1 sec per row). These mimics streaming, even though the source is a static file.

**Enabling APIs (Project Selector)**
Google Cloud Storage uses APIs to communicate and to create a communication. All necessary APIs were enabled.
- Dataflow
- Compute Engine
- Cloud Logging
- Cloud Storage
- Google Cloud Storage JSON
- BigQuery
- Pub/Sub
- Resource Manager

**Create a cloud storage bucket**
Create a cloud storage bucket for Dataflow for temporary files, staging files, and sometimes pipeline output.
Enter a unique bucket name. No sensitive information because the bucket name space in global and publicly visible.
- Choose where to save your data

Copy the following needed for the later section;
- Cloud Storage Bucket name
- Google Cloud Project ID

**Enable Roles:(Go to IAM)**
User Account
- Dataflow Admin
- Service Account User
Compute Service Engine Account-This is an automatically specially created account by Google when you create a google cloud project. 
- Dataflow worker role
- Storage Object Admin role
- Pub/sub editor role
- BigQuery data editor role
- Viewer role
  
**Security:** you can also replace the default Compute Engine account with a customer service account for Dataflow Pipelines. It’s one best practice to show security awareness.
Save


**Create a BigQuery Dataset and Table**
- Create dataset
- Create an empty table for dataset
  
*Create appropriate schema to match the structure of the incoming Pub/Sub data*
**Shema:** This references the topic (incoming data) schema

station_id: string, 
city: string, 
pm25: float, 
pm10: float,
co2: integer
timestamp: timestamp,
  
**Running The Pipeline**
Test Dataflow before Running: 
- Use DirectRunner(Local Testing): Apache Beam provides DirectRunner, which runs my pipeline locally on my machine (no dataflow, no costs).
- Use Small Sample Data: Instead of real Pub/Sub streams, I created a fake dataset (like a JSON file with 10-20 records)
The pipeline gets incoming data from the input topic

**Go to Jobs**
- Create job from template
- Enter job name
- Select Pub/Sub to BigQuery template
- BigQuery output table
- Enter topic manually 
- Save: 
- Temp location: gs://Bucket_Name/temp/
- Network, subnetwork
- Run Job

View Results
Got to BigQuery page
