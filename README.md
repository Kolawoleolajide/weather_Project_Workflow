🌦️ Automated Weather Data Pipeline

----

📌 Overview

This project is a fully automated cloud-based data pipeline that collects real-time weather data from an API, stores it in a PostgreSQL database, and runs on a scheduled basis using GitHub Actions.

It demonstrates an end-to-end data workflow:

API data ingestion

Data storage in cloud database

Automated scheduling

Logging and monitoring

Integration with Power BI



----

🏗️ Architecture

API → Python Script → Supabase (PostgreSQL) → GitHub Actions → Power BI


----

⚙️ Tech Stack

Python (requests, psycopg2)

PostgreSQL (Supabase)

GitHub Actions (automation)

Power BI (data connection & reporting)



----

🧩 Database Setup

Weather Data Table

CREATE TABLE weather_data (
    timestamp TIMESTAMP,
    city TEXT,
    temperature FLOAT,
    humidity FLOAT,
    windspeed FLOAT,
    condition TEXT
);


----

Pipeline Audit Table

CREATE TABLE pipeline_audit (
    id SERIAL PRIMARY KEY,
    run_time TIMESTAMP,
    status TEXT,
    message TEXT
);


🐍 Python Pipeline

Features

Fetches data for multiple cities

Retry mechanism (3 attempts)

Timeout handling

Logging system

Audit tracking (success/failure)

Secure environment variables



---

Script (Core Logic)

import requests
import psycopg2
from datetime import datetime
import os
import logging
import time

API_KEY = os.getenv("API_KEY")
DB_URL = os.getenv("DB_URL")

cities = ["Lagos", "Abuja", "Kano", "Port Harcourt"]

logging.basicConfig(level=logging.INFO)

def fetch_weather(city, retries=3):
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={API_KEY}&units=metric"
    
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()
            return response.json()
        except Exception:
            time.sleep(2)
    
    raise Exception(f"Failed to fetch {city}")

def run_pipeline():
    conn = psycopg2.connect(DB_URL)
    cur = conn.cursor()

    try:
        for city in cities:
            data = fetch_weather(city)

            cur.execute("""
                INSERT INTO weather_data 
                VALUES (%s, %s, %s, %s, %s, %s)
            """, (
                datetime.now(),
                city,
                data["main"]["temp"],
                data["main"]["humidity"],
                data["wind"]["speed"],
                data["weather"][0]["description"]
            ))

        cur.execute("""
            INSERT INTO pipeline_audit (run_time, status, message)
            VALUES (%s, %s, %s)
        """, (datetime.now(), "SUCCESS", "Pipeline ran successfully"))

        conn.commit()

    except Exception as e:
        cur.execute("""
            INSERT INTO pipeline_audit (run_time, status, message)
            VALUES (%s, %s, %s)
        """, (datetime.now(), "FAILED", str(e)))
        conn.commit()
        raise

    finally:
        conn.close()

if __name__ == "__main__":
    run_pipeline()
----
🔐 Environment Variables

Set in GitHub Secrets:

Name	Description

DB_URL	PostgreSQL connection string
API_KEY	Weather API key

----

⚡ Automation (GitHub Actions)

Workflow File

name: Weather Pipeline

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
  DB_URL: ${{ secrets.DB_URL }}
  API_KEY: ${{ secrets.API_KEY }}

on:
  schedule:
    - cron: "0 * * * *"
  workflow_dispatch:

jobs:
  run-script:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: pip install requests psycopg2-binary
      - run: python weather_pipeline.py
----

📊 Power BI Connection

Steps

1. Open Power BI Desktop


2. Click Get Data


3. Select PostgreSQL database


4. Enter:



Server: aws-0-eu-west-1.pooler.supabase.com
Database: postgres

5. Authentication:

Username: postgres.xxxxx

Password: your database password

6. Enable SSL if prompted
7. Select table:
weather_data
8. Click Load

----

📸 Screenshots 

🔹 GitHub Actions Running

[.github/Github Action.png](https://github.com/Kolawoleolajide/weather_Project_Workflow/blob/bc55769f1f16f105efcce8934fe5b204cebb177e/.github/Github%20Action.png)

🔹 Supabase Table

[/screenshots/database-table.png](https://github.com/Kolawoleolajide/weather_Project_Workflow/blob/ab3b1c85406de176d3ac084f5ae8820906b0813a/.github/Supabase.png)

🔹 Power BI Connection

[/screenshots/powerbi-connection.png](https://github.com/Kolawoleolajide/weather_Project_Workflow/blob/ab3b1c85406de176d3ac084f5ae8820906b0813a/.github/weather.png)

----

✅ Results

Fully automated hourly data pipeline

Cloud-based (no local dependency)

Reliable with retry & logging

Audit tracking for monitoring

Ready for analytics and dashboards

----

🚀 Future Improvements

Data deduplication

Alert system (email/Slack on failure)

Add more cities

Store raw API responses

Build real-time dashboard

----

📌 Author

Kolawole Olajide
Data Analyst | Building Real-World Data Projects

