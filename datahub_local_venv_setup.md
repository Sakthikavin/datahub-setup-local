# **DataHub Local Installation and Snowflake Integration Guide (Using Python venv)**

This guide walks you through installing **DataHub locally** on macOS using a Python virtual environment (venv) and configuring Snowflake ingestion.

---

## **Step 1: Install Homebrew (if not installed)**

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

* Homebrew is required to easily install Python and Docker.

---

## **Step 2: Install Python 3.11**

```bash
brew install python@3.11
python3 --version
```
* Verify installation using `python3 --version`.

---

## **Step 3: Create a Virtual Environment (venv)**

```bash
python3 -m venv ~/.venvs/datahub
source ~/.venvs/datahub/bin/activate
```

* This creates an isolated Python environment for DataHub.
* You’ll see `(datahub)` at the start of your terminal prompt once activated.

To deactivate the venv anytime, run:
```bash
deactivate
```

---

## **Step 4: Install DataHub CLI with Snowflake Support**

```bash
pip install --upgrade pip
pip install 'acryl-datahub[snowflake]'
```

* Installs the DataHub CLI and Snowflake dependencies inside the virtual environment.
* Verify installation:

```bash
datahub --help
```

---

## **Step 5: Install Docker Desktop**

* Download and install Docker from [Docker Desktop](https://www.docker.com/products/docker-desktop/).
* Make sure Docker is running before starting DataHub.

---

## **Step 6: Start DataHub Locally**

```bash
datahub docker quickstart
```

* Spins up DataHub in Docker.
* Access the DataHub UI at: [http://localhost:9002](http://localhost:9002)

---

## **Step 7: Configure Snowflake Permissions**

After installing DataHub, configure a **role and grants in Snowflake**:

```sql
-- Create a new role for DataHub
CREATE OR REPLACE ROLE DATAHUB_SNOWFLAKE_ROLE;


-- Grant the role to your DataHub user. This user is used in datasource/snowflake.yml for logging in to snowflake.
GRANT ROLE DATAHUB_SNOWFLAKE_ROLE TO USER Snowflake;

-- Grant access to a warehouse to run queries
GRANT OPERATE, USAGE ON WAREHOUSE COMPUTE_WH TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- Grant access to view database and schema
GRANT USAGE ON DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT USAGE ON ALL SCHEMAS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- Grant select on streams
GRANT SELECT ON ALL STREAMS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON FUTURE STREAMS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- If NOT using Snowflake Profiling or Classification: grant references
GRANT REFERENCES ON ALL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT REFERENCES ON FUTURE TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT REFERENCES ON ALL EXTERNAL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT REFERENCES ON FUTURE EXTERNAL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT REFERENCES ON ALL VIEWS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT REFERENCES ON FUTURE VIEWS IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- Grant monitor privileges for dynamic tables (Enterprise Edition)
GRANT MONITOR ON ALL DYNAMIC TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT MONITOR ON FUTURE DYNAMIC TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- If using Snowflake Profiling or Classification: grant select privileges
GRANT SELECT ON ALL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON FUTURE TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON ALL EXTERNAL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON FUTURE EXTERNAL TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON ALL DYNAMIC TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;
GRANT SELECT ON FUTURE DYNAMIC TABLES IN DATABASE DATAHUB_DATABASE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- lineage
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE DATAHUB_SNOWFLAKE_ROLE;

-- validate the grants given to the role.. run twice before running the ingestion command
SHOW GRANTS TO ROLE DATAHUB_SNOWFLAKE_ROLE;
SHOW GRANTS TO ROLE DATAHUB_SNOWFLAKE_ROLE;
```

---

## **Step 8: Create Snowflake Ingestion YAML**

Create a file named `datasource/snowflake.yml`:

```yaml
source:
  type: snowflake
  config:
    account_id: <replace snowflake account identifier>
    username: <replace snowflake username>
    password: <replace snowflake password>
    role: DATAHUB_SNOWFLAKE_ROLE
    warehouse: COMPUTE_WH

    include_tables: true
    include_views: true
    include_table_lineage: true
    include_view_lineage: true

    profiling:
      enabled: true
      profile_table_level_only: true

    stateful_ingestion:
      enabled: true

pipeline_name: 'urn:li:dataHubIngestionSource:c603f176-e4f9-4b58-9c5d-6ba13f1e07d5'

sink:
  type: datahub-rest
  config:
    server: "http://localhost:8080"  # Update if your DataHub instance runs elsewhere
```

---

## **Step 9: Run the Ingestion**

Make sure DataHub is running:

```bash
datahub docker quickstart
```

Then run the ingestion:

```bash
datahub ingest -c datasource/snowflake.yml
```

* This ingests metadata from Snowflake into your local DataHub instance.

---

## **Notes**

* Always ensure Docker is running before starting DataHub.
* Use `datahub docker quickstart` to monitor logs and service status.
* Customize Snowflake credentials and database names in the YAML file.
* If re-opening the terminal, reactivate your venv before running DataHub:
  ```bash
  source ~/.venvs/datahub/bin/activate
  ```

