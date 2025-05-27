# ADEM-Employment-Data-Shiny-App
ADEM Employment Data Shiny App
=============================

Overview
--------
This project is an R Shiny application for visualizing employment statistics from ADEM's Open Data (https://data.public.lu/fr/datasets/chiffres-cles-de-ladem/) covering 2009 to 2025. It includes:
- An automated ETL pipeline for monthly data updates.
- A PostgreSQL database to store cleaned data.
- An optional REST API for data access.
- A mobile-first Shiny app with interactive filters and time-series visualizations.
- Dockerized deployment for reproducibility.

Project Structure
----------------
- `app.R`: Main Shiny app script with UI and server logic.
- `etl_script.R`: ETL pipeline to fetch, clean, and store data.
- `api.R`: Optional Plumber API for exposing employment stats.
- `Dockerfile`: Docker configuration for the Shiny app and ETL.
- `docker-compose.yml`: Orchestrates PostgreSQL, Shiny app, and ETL services.
- `.github/workflows/etl.yml`: GitHub Actions workflow for monthly ETL runs.

Prerequisites
-------------
- R 4.3.0 or later
- PostgreSQL 14
- Docker and Docker Compose
- R packages: shiny, shinydashboard, shinyMobile, RPostgres, DBI, dplyr, plotly, httr, readr, plumber
- GitHub account (for Actions) or Jenkins (for ETL scheduling)

Setup Instructions
------------------
1. **Clone the Repository**
   ```
   git clone <repository_url>
   cd adem-shiny-app
   ```

2. **Set Up PostgreSQL**
   - Install PostgreSQL 14.
   - Create a database named `adem_data`.
   - Run the following SQL to set up tables:
     ```sql
     CREATE TABLE sector (
         sector_id SERIAL PRIMARY KEY,
         name VARCHAR(100) NOT NULL UNIQUE
     );
     CREATE TABLE job_statistics (
         id SERIAL PRIMARY KEY,
         sector_id INTEGER REFERENCES sector(sector_id),
         employment_rate FLOAT,
         unemployment_rate FLOAT,
         date DATE NOT NULL
     );
     CREATE TABLE etl_run_log (
         log_id SERIAL PRIMARY KEY,
         run_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
         records_added INTEGER
     );
     ```

3. **Install R Dependencies**
   ```
   Rscript -e "install.packages(c('shiny', 'shinydashboard', 'shinyMobile', 'RPostgres', 'DBI', 'dplyr', 'plotly', 'httr', 'readr', 'plumber'))"
   ```

4. **Configure Environment Variables**
   - Create a `.env` file with:
     ```
     POSTGRES_USER=your_user
     POSTGRES_PASSWORD=your_password
     API_TOKEN=your_api_token  # For optional API
     ```

5. **Run the ETL Pipeline**
   - Update `etl_script.R` with the correct ADEM Open Data CSV URL.
   - Run manually:
     ```
     Rscript etl_script.R
     ```
   - Schedule monthly runs via GitHub Actions (see `.github/workflows/etl.yml`) or Jenkins.

6. **Run the Shiny App Locally**
   ```
   Rscript -e "shiny::runApp('app.R', port=3838)"
   ```

7. **Deploy with Docker**
   - Build and run:
     ```
     docker-compose up -d
     ```
   - Access the Shiny app at `http://localhost:3838`.
   - Push to DockerHub (optional):
     ```
     docker tag shiny-app your_dockerhub_username/adem_shiny:latest
     docker push your_dockerhub_username/adem_shiny:latest
     ```

Usage
-----
- **Shiny App**: Open `http://localhost:3838` (or deployed URL). Use the sidebar to:
  - Select a sector.
  - Choose a date range (2009–2025).
  - Pick a metric (employment or unemployment rate).
  - Click "Update" to view a time-series plot.
- **API (Optional)**: If enabled, access at `http://localhost:8000/stats?sector_id=X&start_date=YYYY-MM-DD&end_date=YYYY-MM-DD` with a bearer token.
- **ETL**: Runs monthly to fetch new data from ADEM’s Open Data, clean it, and store it in PostgreSQL.

Troubleshooting
---------------
- **Database Connection Issues**: Ensure PostgreSQL is running and credentials match `.env`.
- **ETL Failures**: Check the ADEM Open Data URL and network connectivity. Review logs in `etl_run_log` table.
- **Shiny App Errors**: Verify R package versions and database schema.

Contributing
------------
- Fork the repository.
- Create a feature branch (`git checkout -b feature/your-feature`).
- Commit changes (`git commit -m "Add your feature"`).
- Push and create a pull request.

License
-------
MIT License

Contact
-------
For issues or questions, open a GitHub issue or contact the project maintainer.
