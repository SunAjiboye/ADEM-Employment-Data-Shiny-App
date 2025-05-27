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
- Secure code management practices to protect sensitive data and ensure robust development workflows.

Project Structure
----------------
- `app.R`: Main Shiny app script with UI and server logic.
- `etl_script.R`: ETL pipeline to fetch, clean, and store data.
- `api.R`: Optional Plumber API for exposing employment stats.
- `Dockerfile`: Docker configuration for the Shiny app and ETL.
- `docker-compose.yml`: Orchestrates PostgreSQL, Shiny app, and ETL services.
- `.github/workflows/etl.yml`: GitHub Actions workflow for monthly ETL runs.
- `.env.example`: Template for environment variables (do not commit actual `.env`).
- `.gitignore`: Excludes sensitive files (e.g., `.env`, temporary data).

Prerequisites
-------------
- R 4.4.3
- PostgreSQL 14
- Docker and Docker Compose
- R packages: shiny, shinydashboard, shinyMobile, RPostgres, DBI, dplyr, plotly, httr, readr, plumber, dotenv
- GitHub account (for Actions) or Jenkins (for ETL scheduling)
- Secret management tool (e.g., Docker secrets, GitHub Secrets)

Setup Instructions
------------------
1. **Clone the Repository**
   ```
   git clone <repository_url>
   cd adem-shiny-app
   ```
   - Ensure `.gitignore` includes `.env`, `*.csv`, and temporary files to prevent accidental commits of sensitive data.

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
   - Pin package versions for compatibility with R 4.4.3:
     ```
     Rscript -e "install.packages(c('shiny=1.8.1', 'shinydashboard=0.7.2', 'shinyMobile=1.0.1', 'RPostgres=1.4.7', 'DBI=1.2.3', 'dplyr=1.1.4', 'plotly=4.10.4', 'httr=1.4.7', 'readr=2.1.5', 'plumber=1.2.2', 'dotenv=0.1.0'))"
     ```
   - Verify package integrity using trusted CRAN mirrors (e.g., https://cran.r-project.org).

4. **Configure Environment Variables**
   - Copy `.env.example` to `.env` and update with secure credentials:
     ```
     cp .env.example .env
     ```
     Example `.env`:
     ```
     POSTGRES_USER=your_user
     POSTGRES_PASSWORD=your_secure_password
     API_TOKEN=your_secure_api_token
     ```
   - Never commit `.env`. Use secret management tools (e.g., GitHub Secrets for Actions, Docker secrets for deployment).
   - Generate secure passwords/tokens using a password manager or `openssl rand -base64 32`.

5. **Run the ETL Pipeline**
   - Update `etl_script.R` with the correct ADEM Open Data CSV URL.
   - Run manually:
     ```
     Rscript etl_script.R
     ```
   - Schedule monthly runs via GitHub Actions (`.github/workflows/etl.yml`) with secrets:
     ```yaml
     env:
       POSTGRES_USER: ${{ secrets.POSTGRES_USER }}
       POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
     ```
   - Alternatively, use Jenkins with secure credential storage.

6. **Run the Shiny App Locally**
   ```
   Rscript -e "shiny::runApp('app.R', port=3838)"
   ```

7. **Deploy with Docker**
   - Build and run:
     ```
     docker-compose up -d
     ```
   - Access the Shiny app at `https://localhost:3838` (configure HTTPS with Nginx reverse proxy).
   - Push to DockerHub (optional):
     ```
     docker tag shiny-app your_dockerhub_username/adem_shiny:latest
     docker push your_dockerhub_username/adem_shiny:latest
     ```
   - Use Docker secrets for sensitive environment variables:
     ```
     docker secret create postgres_password your_secure_password.txt
     ```

Secure Code Management
----------------------
- **Credential Security**:
  - Store sensitive data (e.g., `POSTGRES_USER`, `POSTGRES_PASSWORD`, `API_TOKEN`) in `.env` or secret management tools.
  - Use `dotenv` package in R to load environment variables securely:
    ```R
    library(dotenv)
    load_dot_env()
    ```
  - Avoid hardcoding credentials in scripts. Use `Sys.getenv()` to access them.
- **Version Control**:
  - Ensure `.gitignore` includes:
    ```
    .env
    *.csv
    *.rds
    /data/
    ```
  - Use `git-secrets` or similar tools to scan commits for sensitive data.
  - Commit `.env.example` with placeholder values to guide setup.
- **Dependency Security**:
  - Install packages from trusted CRAN mirrors.
  - Regularly check for vulnerabilities using `pak::pkg_deps_tree()` or tools like Dependabot.
  - Pin package versions to avoid breaking changes or unverified updates.
- **Code Review**:
  - Use pull requests for all changes. Require at least one reviewer.
  - Enable branch protection on the main branch in GitHub to prevent direct pushes.
- **Access Control**:
  - Restrict repository access to authorized contributors.
  - Use GitHub teams or roles to manage permissions.
- **Secure Deployment**:
  - Configure HTTPS using a reverse proxy (e.g., Nginx with Let’s Encrypt).
  - Restrict Docker container permissions (e.g., run as non-root user):
    ```dockerfile
    USER shiny
    ```
  - Monitor for vulnerabilities using tools like Trivy or Docker Scan.
  - Rotate API tokens and database credentials periodically.

Usage
-----
- **Shiny App**: Open `https://localhost:3838` (or deployed URL). Use the sidebar to:
  - Select a sector.
  - Choose a date range (2009–2025).
  - Pick a metric (employment or unemployment rate).
  - Click "Update" to view a time-series plot.
- **API (Optional)**: Access at `https://localhost:8000/stats?sector_id=X&start_date=YYYY-MM-DD&end_date=YYYY-MM-DD` with a bearer token.
- **ETL**: Runs monthly to fetch new data from ADEM’s Open Data, clean it, and store it in PostgreSQL.

Troubleshooting
---------------
- **Database Connection Issues**: Verify PostgreSQL is running and credentials match `.env`. Check firewall settings for port 5432.
- **ETL Failures**: Confirm the ADEM Open Data URL and network connectivity. Review logs in `etl_run_log` table.
- **Shiny App Errors**: Ensure R 4.4.3 and pinned package versions are installed. Verify database schema.
- **Security Issues**: Scan for exposed credentials using `git log -p | grep -E "POSTGRES|API_TOKEN"`.

Contributing
------------
- Fork the repository.
- Create a feature branch (`git checkout -b feature/your-feature`).
- Commit changes (`git commit -m "Add your feature"`).
- Run `git-secrets` or similar to ensure no sensitive data is committed.
- Push and create a pull request with detailed descriptions.

License
-------
MIT License

Contact
-------
For issues or questions, open a GitHub issue or contact the project maintainer.
