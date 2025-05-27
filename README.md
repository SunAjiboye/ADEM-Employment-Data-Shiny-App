# ADEM-Employment-Data-Shiny-App
ADEM Employment Data Shiny App with Angular Front-End
===================================================

Overview
--------
This project is a hybrid application for visualizing employment statistics from ADEM's Open Data (https://data.public.lu/fr/datasets/chiffres-cles-de-ladem/) covering 2009 to 2025. It includes:
- An automated ETL pipeline for monthly data updates.
- A PostgreSQL database to store cleaned data.
- A Plumber API to serve data to the front-end.
- An R Shiny app as a fallback or alternative interface.
- A secure Angular front-end with interactive filters and time-series visualizations.
- Dockerized deployment for reproducibility.
- Secure code management practices, including Angular-specific security and automated code scanning via GitHub Actions.

Project Structure
----------------
- `backend/app.R`: R Shiny app with UI and server logic (fallback interface).
- `backend/etl_script.R`: ETL pipeline to fetch, clean, and store data.
- `backend/api.R`: Plumber API for exposing employment stats to the Angular front-end.
- `frontend/src/`: Angular application source code.
- `Dockerfile`: Docker configuration for the backend (Shiny and Plumber).
- `frontend/Dockerfile`: Docker configuration for the Angular front-end.
- `docker-compose.yml`: Orchestrates PostgreSQL, Shiny/Plumber backend, and Angular front-end.
- `.github/workflows/etl.yml`: GitHub Actions workflow for monthly ETL runs.
- `.github/workflows/codeql.yml`: GitHub Actions workflow for code scanning on push and pull requests.
- `.env.example`: Template for environment variables (do not commit actual `.env`).
- `.gitignore`: Excludes sensitive files (e.g., `.env`, temporary data, Angular build artifacts).

Prerequisites
-------------
- **Backend**:
  - R 4.4.3
  - PostgreSQL 14
  - R packages: shiny, shinydashboard, shinyMobile, RPostgres, DBI, dplyr, plotly, httr, readr, plumber, dotenv
- **Front-End**:
  - Node.js 20.x
  - Angular CLI 17.x
  - npm packages: @angular/core, @angular/common, @angular/http, chart.js, rxjs
- Docker and Docker Compose
- GitHub account (for Actions) or Jenkins (for ETL scheduling)
- Secret management tool (e.g., Docker secrets, GitHub Secrets)

Setup Instructions
------------------
1. **Clone the Repository**
   ```
   git clone <repository_url>
   cd adem-shiny-app
   ```
   - Ensure `.gitignore` includes `.env`, `*.csv`, `*.rds`, `/data/`, and `/frontend/dist/` to prevent accidental commits of sensitive data or build artifacts.

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

3. **Install Backend Dependencies (R 4.4.3)**
   - Pin package versions for compatibility:
     ```
     Rscript -e "install.packages(c('shiny=1.8.1', 'shinydashboard=0.7.2', 'shinyMobile=1.0.1', 'RPostgres=1.4.7', 'DBI=1.2.3', 'dplyr=1.1.4', 'plotly=4.10.4', 'httr=1.4.7', 'readr=2.1.5', 'plumber=1.2.2', 'dotenv=0.1.0'))"
     ```
   - Verify package integrity using trusted CRAN mirrors (e.g., https://cran.r-project.org).

4. **Install Front-End Dependencies (Angular)**
   - Navigate to the `frontend` directory:
     ```
     cd frontend
     npm install -g @angular/cli@17
     npm install
     ```
   - Verify Angular package integrity using `npm audit` and update vulnerable dependencies.

5. **Configure Environment Variables**
   - Copy `.env.example` to `.env` and update with secure credentials:
     ```
     cp .env.example .env
     ```
     Example `.env`:
     ```
     POSTGRES_USER=your_user
     POSTGRES_PASSWORD=your_secure_password
     API_TOKEN=your_secure_api_token
     API_URL=http://localhost:8000
     ```
   - Never commit `.env`. Use secret management tools (e.g., GitHub Secrets, Docker secrets).
   - Generate secure passwords/tokens using a password manager or `openssl rand -base64 32`.

6. **Set Up Code Scanning with GitHub Actions**
   - A CodeQL workflow (`codeql.yml`) is included to scan code on every push and pull request.
   - To initialize CodeQL:
     ```
     gh repo set-default <repository_url>
     gh codeql init
     ```
   - The workflow scans R and Angular code for vulnerabilities and coding errors.[](https://docs.github.com/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning)

7. **Run the ETL Pipeline**
   - Update `etl_script.R` with the correct ADEM Open Data CSV URL.
   - Run manually:
     ```
     Rscript backend/etl_script.R
     ```
   - Schedule monthly runs via GitHub Actions (`.github/workflows/etl.yml`) with secrets:
     ```yaml
     env:
       POSTGRES_USER: ${{ secrets.POSTGRES_USER }}
       POSTGRES_PASSWORD: ${{ secrets.POSTGRES_PASSWORD }}
     ```
   - Alternatively, use Jenkins with secure credential storage.

8. **Run the Backend Locally**
   - Start the Plumber API:
     ```
     Rscript -e "plumber::plumb('backend/api.R')$run(port=8000)"
     ```
   - Start the Shiny app (optional):
     ```
     Rscript -e "shiny::runApp('backend/app.R', port=3838)"
     ```

9. **Run the Angular Front-End Locally**
   - Navigate to `frontend` and start the Angular app:
     ```
     cd frontend
     ng serve --open
     ```
   - Access at `http://localhost:4200`.

10. **Deploy with Docker**
    - Build and run:
      ```
      docker-compose up -d
      ```
    - Access the Angular front-end at `https://localhost:80` and Shiny app at `https://localhost:3838` (configure HTTPS with Nginx).
    - Push to DockerHub (optional):
      ```
      docker tag backend your_dockerhub_username/adem_backend:latest
      docker tag frontend your_dockerhub_username/adem_frontend:latest
      docker push your_dockerhub_username/adem_backend:latest
      docker push your_dockerhub_username/adem_frontend:latest
      ```
    - Use Docker secrets for sensitive environment variables:
      ```
      docker secret create postgres_password your_secure_password.txt
      ```

Secure Code Management
----------------------
- **Credential Security**:
  - Store sensitive data (e.g., `POSTGRES_USER`, `POSTGRES_PASSWORD`, `API_TOKEN`, `API_URL`) in `.env` or secret management tools.
  - In R, use `dotenv` package to load environment variables:
    ```R
    library(dotenv)
    load_dot_env()
    ```
  - In Angular, use environment files (`frontend/src/environments/`):
    ```typescript
    export const environment = {
      production: false,
      apiUrl: 'http://localhost:8000'
    };
    ```
  - Avoid hardcoding credentials. Use `Sys.getenv()` in R and `environment.apiUrl` in Angular.
- **Version Control**:
  - Ensure `.gitignore` includes:
    ```
    .env
    *.csv
    *.rds
    /data/
    /frontend/dist/
    /frontend/node_modules/
    ```
  - Use `git-secrets` to scan commits for sensitive data:
    ```
    git-secrets --scan
    ```
  - Commit `.env.example` with placeholder values.
- **Dependency Security**:
  - For R: Install packages from trusted CRAN mirrors. Check vulnerabilities with `pak::pkg_deps_tree()` or Dependabot.
  - For Angular: Run `npm audit` to identify vulnerabilities. Use `npm install --save-exact` to pin versions.
  - Pin versions for reproducibility:
    - R: See pinned versions in Setup Instructions.
    - Angular: Example `package.json`:
      ```json
      {
        "dependencies": {
          "@angular/core": "17.0.0",
          "@angular/common": "17.0.0",
          "chart.js": "4.4.0",
          "rxjs": "7.8.0"
        }
      }
      ```
- **Code Review**:
  - Use pull requests for all changes with at least one reviewer.
  - Enable branch protection on the main branch in GitHub to prevent direct pushes.
- **Access Control**:
  - Restrict repository access to authorized contributors.
  - Use GitHub teams or roles to manage permissions.
- **Angular Security**:
  - Prevent XSS by sanitizing inputs using Angular’s `DomSanitizer`:
    ```typescript
    import { DomSanitizer } from '@angular/platform-browser';
    constructor(private sanitizer: DomSanitizer) {}
    safeHtml = this.sanitizer.bypassSecurityTrustHtml(data);
    ```
  - Secure API calls with HTTP interceptors to add bearer tokens:
    ```typescript
    intercept(req: HttpRequest<any>, next: HttpHandler) {
      const token = localStorage.getItem('token');
      const authReq = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
      return next.handle(authReq);
    }
    ```
  - Use HTTPS for API communication.
  - Enable Angular’s built-in CSRF protection for forms.
  - Validate and sanitize all user inputs on the backend (Plumber API).
- **Code Scanning**:
  - A GitHub Actions workflow (`codeql.yml`) runs CodeQL scans on every push and pull request to detect vulnerabilities in R and Angular code.[](https://github.blog/changelog/2021-09-27-showing-code-scanning-alerts-on-pull-requests/)[](https://docs.github.com/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning)
  - Review scan results in the GitHub Security tab and resolve alerts in pull requests.[](https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/triaging-code-scanning-alerts-in-pull-requests)
  - Example `codeql.yml`:
    ```yaml
    name: CodeQL Analysis
    on:
      push:
        branches: [main]
      pull_request:
        branches: [main]
    jobs:
      codeql:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - uses: github/codeql-action/init@v3
            with:
              languages: javascript, r
          - uses: github/codeql-action/analyze@v3
    ```
- **Secure Deployment**:
  - Configure HTTPS using Nginx with Let’s Encrypt.
  - Run Docker containers as non-root users:
    ```dockerfile
    USER shiny  # For backend
    USER node   # For frontend
    ```
  - Monitor vulnerabilities with Trivy or Docker Scan.
  - Rotate API tokens and database credentials periodically.

Usage
-----
- **Angular Front-End**: Open `https://localhost:80` (or deployed URL). Use the interface to:
  - Select a sector.
  - Choose a date range (2009–2025).
  - Pick a metric (employment or unemployment rate).
  - View interactive time-series charts (powered by Chart.js).
- **Shiny App (Fallback)**: Open `https://localhost:3838`. Use the sidebar for similar functionality.
- **API**: Access at `https://localhost:8000/stats?sector_id=X&start_date=YYYY-MM-DD&end_date=YYYY-MM-DD` with a bearer token.
- **ETL**: Runs monthly to fetch, clean, and store ADEM Open Data in PostgreSQL.

Troubleshooting
---------------
- **Database Connection Issues**: Verify PostgreSQL is running and credentials match `.env`. Check firewall settings for port 5432.
- **ETL Failures**: Confirm the ADEM Open Data URL and network connectivity. Review logs in `etl_run_log` table.
- **Shiny App Errors**: Ensure R 4.4.3 and pinned package versions are installed. Verify database schema.
- **Angular Errors**: Run `npm audit fix` for dependency issues. Check console logs in the browser.
- **Security Issues**: Scan for exposed credentials using `git log -p | grep -E "POSTGRES|API_TOKEN"`. Review CodeQL alerts in GitHub Security tab.
- **Code Scanning Failures**: Check GitHub Actions logs for CodeQL errors. Ensure `codeql.yml` is in the repository root.

Contributing
------------
- Fork the repository.
- Create a feature branch (`git checkout -b feature/your-feature`).
- Commit changes (`git commit -m "Add your feature"`).
- Run `git-secrets` and CodeQL scans to ensure no sensitive data or vulnerabilities.
- Push and create a pull request with detailed descriptions.

License
-------
MIT License

Contact
-------
For issues or questions, open a GitHub issue or contact the project maintainer.
