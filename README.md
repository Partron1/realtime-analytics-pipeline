## realtime-analytics-pipeline

### Streaming Pipeline Using Dataflow: 
Real-time Air Quality Monitoring in Accra

**Source:** IoT sensors in cities stream air quality data (timestamp, location, PM2.5, PM10, CO, NO2, O3, temperature and humidity levels every few seconds)

- In this project I used Pub/sub to BigQuery 
  
The Pub/sub template is a streaming pipeline that can read JSON-formatted messages from a Pub/Sub topic and write them to a BigQuery table 

**Pipeline:**  
- Pub/sub (Raw messages) 
- Dataflow (Custom User Defined Function (UDF) stage: clean/transform/enrich)
- BigQuery (Analytics-ready table) 

**Dashboard:** 

Tableau showing city-level pollution heatmaps, alerts when thresholds are exceeded. 
Impact: Demonstrates environmental monitoring for smart cities 

- Pub/Sub topic in JSON format used;
```JSON  
{
     
"timestamp":1672531200000,

"location":"Accra",

"PM2_5":41.3,

"PM10":12.56,

"NO2":13.65,

"CO":2.57,

"O3":131.05,

"temperature":26.6,

"humidity":71.6

}

```
IoT sensors (or the software that manages it, like Raspberry Pi, Arduino, or edge gateway) sends HTTP or gRPC request to Pub/Sub APIs.

For this project I did not have a real-time streaming data from any IoT sensor so I used a local file as a streaming source **(batch-to-stream trick)**

Python script to read the file line by line and publish each row into Pub/Sub (with a delay e.g., 1 sec per row). These mimics streaming, even though the source is a static file.

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

**Enable Roles:(Go to IAM)**
Note: The "Include Google-provided role grants" *ensures Dataflow runs smoothly without me worrying about every micro-permission* 
1. User Account
- Dataflow Admin
- Service Account User
  
2. Grant access
Compute Engine default service account email: 527899781926-compute@developer.gserviceaccount.com
*This is an automatically specially created account by Google when you create a google cloud project.*
- Dataflow worker role
- Storage Object Admin role
- Pub/sub editor role
- BigQuery data editor role
- Viewer role
  
**Security:** you can also replace the default Compute Engine account with a customer service account for Dataflow Pipelines. It’s one best practice to show security awareness.

Save

**Create a cloud storage bucket**
Create a cloud storage bucket for Dataflow for temporary files, staging files, and sometimes pipeline output.
*Entered a unique bucket name. No sensitive information because the bucket name space in global and publicly visible.*
- Confirm: Public access prevention on this bucket

*Copy the following needed for the later section;*
*- Cloud Storage Bucket name*
*- Google Cloud Project ID*

**Create a BigQuery Dataset and Table**
- Create dataset
- Create an empty table for dataset: *air_quality*, *schema:edit as text*, *partition by field-timestamp*
  
*Create appropriate schema to match the structure of the incoming Pub/Sub data*

**Shema:** This references the topic (incoming data) schema
```    
timestamp:timestamp,

location: string,

PM2.5: float,

PM10:float,

NO2:float,

CO:float,

O3:float,

temperature:float,

humidity:float

```
**Running The Pipeline**
Test Dataflow before Running: 
- Use DirectRunner(Local Testing): Apache Beam provides DirectRunner, which runs my pipeline locally on my machine (no dataflow, no costs).
- Use Small Sample Data: Instead of real Pub/Sub streams, I created a fake dataset (like a JSON file with 10-20 records)
  
The pipeline gets incoming data from the input topic

**Go to Jobs**
- Create job from template
- Job name: air-data
- Template: Pub/Sub to BigQuery template
- BigQuery output table: tekstain-25:pollution.air_quality 
- Input Pub/Sub topic: Enter topic manually 
- Save: 
- Temp location: gs://teckflow_bucket/temp/
- Network, subnetwork
- Run Job

  
```python
# Batch-to-stream simulator with Pub/Sub
import time
import json
from google.cloud import pubsub_v1

# Pub/Sub configuration
project_id = "tekstain-25"
topic_id = "air-quality"

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path(project_id, topic_id)

# Input file (each line is one "event")
input_file = "air_quality_data.jsonl"  # JSON lines file

# Publish line by line with delay
with open(input_file, "r") as f:
    for line in f:
        line = line.strip()
        if not line:
            continue

        # Convert to bytes (Pub/Sub requires bytes)
        data = line.encode("utf-8")
        future = publisher.publish(topic_path, data)
        print(f"Published message ID: {future.result()}")

        # Delay between messages to mimic streaming
        time.sleep(1)  # 1 second per event
```


```python
# Multi-device IoT stream simulator (Python)
import time
import json
import random
from datetime import datetime
from google.cloud import pubsub_v1

# Pub/Sub configuration
project_id = "tekstain-25"
topic_id = "air-quality"

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path(tekstain-25, air-quality)

# Simulated IoT devices
device_ids = [f"device-{i}" for i in range(1, 6)]  # device-1 ... device-5

# Input file (one row = one payload)
input_file = "air_quality_data.jsonl"  # or .csv

with open(input_file, "r") as f:
    for line in f:
        line = line.strip()
        if not line:
            continue

        # Choose a random device
        device_id = random.choice(device_ids)

        # Build IoT-style message
        message = {
            "device_id": device_id,
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "payload": json.loads(line) if line.startswith("{") else {"raw": line}
        }

        # Publish to Pub/Sub
        data = json.dumps(message).encode("utf-8")
        future = publisher.publish(topic_path, data)
        print(f"Published from {device_id}, message ID: {future.result()}")

        # Delay between events
        time.sleep(1)  # 1 second per simulated event
```

View Results

Got to BigQuery page
