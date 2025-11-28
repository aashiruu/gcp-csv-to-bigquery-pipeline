
# Serverless CSV to BigQuery Pipeline on GCP

A completely serverless, event-driven data pipeline on Google Cloud Platform that automatically processes CSV files uploaded to Cloud Storage, transforms them, and loads them into BigQuery for analysis. This is a foundational pattern for building cost-effective, scalable ETL (Extract, Transform, Load) workflows without managing any servers.

## Architecture Overview

The pipeline is triggered automatically by a simple file upload, making it both powerful and easy to use.

1.  **Trigger:** A user (or system) uploads a CSV file to a designated Google Cloud Storage (GCS) bucket.
2.  **Event Capture:** The GCS bucket emits an event notification.
3.  **Processing & Transformation:** The event triggers a **Cloud Function** (2nd Gen).
4.  **Data Load:** The Cloud Function reads the CSV, performs validation/transformation, and streams the data into **BigQuery**.
5.  **Analysis:** The data is immediately available in BigQuery for querying and visualization.

   
## Features

*   **Fully Serverless:** No infrastructure to manage. Scales automatically with usage.
*   **Event-Driven:** Processing begins within seconds of a file being uploaded.
*   **Flexible Schema Handling:** The function can be modified to handle different CSV formats and perform data cleansing.
*   **Cost-Effective:** Pay only for the compute time and storage/query costs used.

## Quick Start

### Prerequisites

- A Google Cloud Project with billing enabled.
- The `gcloud` CLI installed and authenticated.
- Basic knowledge of Python and GCP services.

### 1. Enable Required APIs

Enable the necessary Google Cloud APIs:

```bash
gcloud services enable \
  cloudfunctions.googleapis.com \
  eventarc.googleapis.com \
  cloudbuild.googleapis.com \
  logging.googleapis.com \
  bigquery.googleapis.com \
  storage.googleapis.com
```

2. Run the BigQuery Setup Commands: After the APIs are enabled,the guide tells you to run these next to create your dataset and table.

```bash
bq mk --dataset \
  --location=US \
  $GOOGLE_CLOUD_PROJECT:csv_ingestion
```

```bash
bq mk --table \
  csv_ingestion.user_data \
  name:STRING,age:INTEGER,city:STRING
```

Note: $GOOGLE_CLOUD_PROJECT is an environment variable. You can replace it with your actual Project ID if it's not set.

3. Run the GCS Bucket Creation Command: Then,you create the bucket that will trigger the function.

```bash
gsutil mb -l US gs://$YOUR_BUCKET_NAME
```

Note: You must replace $YOUR_BUCKET_NAME with a unique name.
4. Create the Python Files and Deploy the Function

```bash
gcloud functions deploy csv-to-bigquery-loader \
    --gen2 \
    --runtime=python311 \
    --region=us-central1 \
    --source=. \
    --entry-point=csv_to_bigquery \
    --trigger-event-filters="type=google.cloud.storage.object.v1.finalized" \
    --trigger-event-filters="bucket=$YOUR_BUCKET_NAME" \
    --service-account=$(gcloud config get-value account)
```
5. Test the Pipeline

1. Create a test CSV file (test_data.csv):
   ```csv
   name,age,city
   Alice,30,Seattle
   Bob,25,New York
   Charlie,35,Austin
   ```
2. Upload the file to your trigger bucket:
   ```bash
   gsutil cp test_data.csv gs://$YOUR_BUCKET_NAME/
   ```
3. Verify the results:
   · Check the Cloud Function Logs in the GCP Console.
   · Query BigQuery to see your data:
     ```bash
     bq query --use_legacy_sql=false \
     "SELECT * FROM \`$GOOGLE_CLOUD_PROJECT.csv_ingestion.user_data\`"
     ```
