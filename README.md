# Data Processing using Dataflow and BigQuery on Google Cloud
> **This project demonstrates an ETL batch pipeline using GCP services, Cloud Storage for data storage, Dataflow for data processing, and BigQuery as a sink for the results. The pipeline ingests data from a publicly available dataset, processes it using Dataflow, and loads it into BigQuery for further analysis.**

<p align="center">
<img width="800" src="https://github.com/user-attachments/assets/4ffd64d9-75f1-4542-a8fa-ccf16db4e560" />
</p>

___

# Pipeline Setup Guide

Ensure that the **Dataflow API** is enabled

Setting an environment variables
```bash
export REGION=us-east4
gcloud config set compute/region $REGION
```
```bash
export PROJECT=Data_Processing_Project
gcloud config set project $PROJECT
```

## 1: Create a Cloud Storage Bucket
Create a new regional bucket within your project:
```bash
gcloud storage buckets create gs://$PROJECT --location=$REGION
```

## 2: Copying data from local to the Bucket
Copy files and scripts of the pipeline
```bash
gsutil cp -r ./dataflow_python gs://$PROJECT/dataflow_python
```
```bash
gsutil cp -r ./data_files gs://$PROJECT/data_files
```

## 3: Create a BigQuery dataset
In the Cloud Shell, create a dataset in BigQuery Dataset called lake. This is where all of your tables will be loaded in BigQuery:
```bash
bq mk lake
```

## 4: Build a Dataflow pipeline
We will build a Dataflow pipeline with a TextIO source and a BigQueryIO destination to ingest data into BigQuery, it will:
* Ingest the files from Cloud Storage.
* Filter out the header row in the files.
* Convert the lines read to dictionary objects.
* Output the rows to BigQuery.

<p align="center">
<img width="200" alt="xu2ZNOCZ0IgdlDNdPrmJN_DIpdFiPVPCSstHGOYHeDM=" src="https://github.com/user-attachments/assets/b38ff679-ce2b-487a-8b0d-a68a8a6714a7" />
</p>

## 5: Run Dataflow pipeline
The Dataflow job here requires Python3.8. To ensure you're on the proper version, you will run the Dataflow processes in a Python 3.8 Docker container.

Run the following in Cloud Shell to start up a Python Container:
```bash
docker run -it -e PROJECT=$PROJECT -v $(pwd)/dataflow-python:/dataflow python:3.8 /bin/bash
```
Once the container finishes pulling, and starts executing in the Cloud Shell, run the following to install apache-beam in that running container:
```bash
pip install apache-beam[gcp]==2.59.0
```
Change directories into where you linked the source code:
```bash
cd dataflow/
```

## 6: Run the ingestion Dataflow pipeline
The code is an Apache Beam pipeline that reads a CSV file, parses each row into a dictionary format, and writes the data into a BigQuery table without any transformation.
```bash
python dataflow_python/data_ingestion.py \
  --project=$PROJECT \
  --region=$REGION \
  --runner=DataflowRunner \
  --machine_type=e2-standard-2 \
  --staging_location=gs://$PROJECT/test \
  --temp_location gs://$PROJECT/test \
  --input gs://$PROJECT/data_files/head_usa_names.csv \
  --save_main_session
```

## 7. Data transformation
The code is an Apache Beam pipeline that reads a CSV file, transforms the year column into BigQuery's date format, and writes the data into a BigQuery table using a predefined JSON schema.
```bash
python dataflow_python/data_transformation.py \
  --project=$PROJECT \
  --region=$REGION \
  --runner=DataflowRunner \
  --machine_type=e2-standard-2 \
  --staging_location=gs://$PROJECT/test \
  --temp_location gs://$PROJECT/test \
  --input gs://$PROJECT/data_files/head_usa_names.csv \
  --save_main_session
```

## 8. Run the Data Enrichment Dataflow pipeline
The code defines an Apache Beam pipeline that reads a CSV file containing U.S. state-related data, enriches it with state full names retrieved from BigQuery, and writes the transformed data to a BigQuery table.
```bash
python dataflow_python/data_enrichment.py \
  --project=$PROJECT \
  --region=$REGION \
  --runner=DataflowRunner \
  --machine_type=e2-standard-2 \
  --staging_location=gs://$PROJECT/test \
  --temp_location gs://$PROJECT/test \
  --input gs://$PROJECT/data_files/head_usa_names.csv \
  --save_main_session
```

## 9. Run the Pipeline to perform the Data Join and create the resulting table in BigQuery
This script defines an Apache Beam Dataflow pipeline that reads data from two BigQuery tables (orders and account), joins them using side inputs, and writes the denormalized data back to a BigQuery table.
```bash
python dataflow_python/data_lake_to_mart.py \
  --worker_disk_type="compute.googleapis.com/projects//zones//diskTypes/pd-ssd" \
  --max_num_workers=4 \
  --project=$PROJECT \
  --region=$REGION
  --runner=DataflowRunner \
  --machine_type=e2-standard-2 \
  --staging_location=gs://$PROJECT/test \
  --temp_location gs://$PROJECT/test \
  --save_main_session \
```

___

## Deployed Dataflow pipelines

<p align="center">
<img width="800" src="https://github.com/user-attachments/assets/d101e39e-199f-48e5-9239-b2f44255a84f" />
</p>


## BigQuery output results

<p align="center">
<img width="800" src="https://github.com/user-attachments/assets/c1e03466-9bac-4b35-b28b-0c07dbc747b4" />
</p>



