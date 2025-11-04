# Steam Analytics Pipeline

A data engineering project that extracts hourly player counts for popular Steam games, loads them into a PostgreSQL data warehouse, transforms the data using dbt, and visualizes the results with Apache Superset.

This project demonstrates a full, end-to-end ELT (Extract, Load, Transform) pipeline.

![Dashboard Screenshot Placeholder](https://i.imgur.com/gY9m0P0.png)
*(TODO: Replace this placeholder with a real screenshot of your Superset dashboard once it's built)*

---

## 🛠️ Tech Stack

* **Python 3:** For data extraction (from Steam API) and loading (to Postgres).
    * `requests` (for API calls)
    * `pandas` (for data handling)
    * `SQLAlchemy` / `psycopg2` (for database connection)
    * `python-dotenv` (for managing secrets)
* **PostgreSQL:** As the data warehouse, running in Docker.
* **Docker & Docker Compose:** For containerizing and managing all services (Postgres, Superset).
* **`dbt` (Data Build Tool):** For all data transformation and modeling (transforming raw data into a clean Star Schema).
* **Apache Superset:** For data visualization and building the final dashboard.
* **`cron`:** For scheduling the hourly data extraction script.
* **Git & GitHub:** For version control.

---

## 🚀 Getting Started

### Prerequisites

* Git
* Docker & Docker Compose
* Python 3 & `pip`
* A **Steam Web API Key**. You can get one here: [steamcommunity.com/dev/apikey](https://steamcommunity.com/dev/apikey)

### 1. Set Up The Project

```bash
# Clone the repository
git clone [https://github.com/TymonWojtanowski/steam-analytics-pipeline.git](https://github.com/TymonWojtanowski/steam-analytics-pipeline.git)
cd steam-analytics-pipeline

# Create a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt
```
2. Configure Your Environment
This project requires API keys and passwords, which are safely managed using a .env file.

Create your secret file:

```bash

# This copies the example file. Never commit the .env file!
cp .env.example .env
```
Edit the .env file with your favorite editor (e.g., nano .env) and fill in your credentials:

```ini, TOML

STEAM_API_KEY="YOUR_SECRET_STEAM_KEY"
DB_PASSWORD="YOUR_CHOSEN_STRONG_PASSWORD"
```
3. Launch Services
This one command will start your PostgreSQL database and your Apache Superset instance.

```bash

docker-compose up -d
```
PostgreSQL will be available at localhost:5432.

Apache Superset will be available at http://localhost:8088 (login: admin, pass: admin).

4. Run the Pipeline
Create the Database Schema: Connect to your database (e.g., using DataGrip) and run the schema.sql script to create the raw tables.

Run the One-Time Bootstrap Script: This script populates your raw_steam_games table with the games you want to track.

```bash

python bootstrap_games.py
Run the Transformation Layer: Go into the dbt project folder and run your models. This will build your clean Star Schema (dim_ and fct_ tables).

```bash

cd dbt_steam_analytics
dbt run
cd ..
```
Set Up the Scheduler: Run the extraction script once manually to test it, then set it up in cron to run automatically.

```bash

# Manual test run
python extract_player_counts.py

# Add to crontab (edit paths as needed)
(crontab -l 2>/dev/null; echo "0 * * * * /path/to/your/project/.venv/bin/python /path/to/your/project/extract_player_counts.py >> /path/to/your/project/cron.log 2>&1") | crontab -
```
5. View Your Dashboard
Go to http://localhost:8088 and log in to Superset.

Connect Superset to your PostgreSQL database.

Add your clean dbt tables (dim_games, fct_player_counts) as Datasets.

Build your charts and dashboard!

📁 Project Structure
```bash
.
├── dbt_steam_analytics/  # Contains the dbt project for all SQL transformations
│   ├── models/
│   │   ├── staging/      # Models for cleaning raw data
│   │   └── marts/        # Models for the final Star Schema (dim_ and fct_)
│   └── dbt_project.yml   # dbt project configuration
├── .env                  # (Ignored by Git) Holds all secret keys and passwords
├── .env.example          # An example file for environment variables
├── .gitignore            # Tells Git what files to ignore
├── bootstrap_games.py    # One-time script to populate the games list
├── docker-compose.yml    # Defines all services (Postgres, Superset)
├── extract_player_counts.py # Main extraction script, run by cron
├── README.md             # This file
├── requirements.txt      # Python dependencies
└── schema.sql            # SQL script to create the initial raw tables
```
