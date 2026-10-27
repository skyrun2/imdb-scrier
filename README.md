# IMDb ETL + API + CLI

A complete data engineering system that extracts IMDb datasets, processes them through an optimized ETL pipeline, stores results in PostgreSQL, and provides access through both a REST API and command-line interfaces. Each module has its own README file with more detailed information.

## Project Overview

This project is organized into three main modules that work together:

- **`ingest_module`** – Extracts IMDb datasets, transforms and cleans the data, and loads it into PostgreSQL using an optimized streaming approach.
- **`server_module`** – Provides a FastAPI-based REST API with search capabilities and full-text search indexes.
- **`cli_module`** – Provides a user-friendly command-line interface for querying actors and movies.

The system runs on Windows with a locally installed PostgreSQL database. Each module can be configured and executed independently.

### Architecture

![System Architecture](docs/architecture.png)

### Project Structure

```text
.
├── ingest_module/           # ETL pipeline
│   ├── extract/             # Data extraction from IMDb
│   ├── transform/           # Data cleaning and transformation
│   ├── load/                # Database loading
│   └── utils/               # Shared utilities
├── server_module/           # FastAPI REST API
│   ├── src/
│   │   ├── actors/          # Actor routes and logic
│   │   └── movies/          # Movie routes and logic
│   └── core/                # Config, database, logging
├── cli_module/              # Click CLI
│   └── imdb_cli/            # CLI commands and formatters
├── docs/                    # Documentation and diagrams
├── README.md                # Project documentation
└── .env                     # Configuration (create from template)
```

## Getting Started on Windows

### Prerequisites

Install the following before setting up the project:

- Windows 10 or Windows 11
- Python 3.11 or later
- PostgreSQL 18 or a compatible PostgreSQL version
- Git for Windows
- `uv` Python package manager

Ensure PostgreSQL is installed, its service is running, and you can connect to the database.

### 1. Install uv

Open Git Bash and run:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh # restart pc after
```

Alternatively, install `uv` using the official Windows installer instructions:

https://docs.astral.sh/uv/getting-started/installation/

Restart your terminal if necessary, then verify the installation:

```bash
uv --version
```

### 2. Create the PostgreSQL Database

Connect to PostgreSQL using pgAdmin or the `psql` command-line utility.

Create the project database and user if they do not already exist:

```sql
CREATE USER imdb_user WITH PASSWORD 'imdb_pass';

CREATE DATABASE imdb OWNER imdb_user;
```

If you already have the user or database, reuse your existing credentials instead of creating duplicates.

Verify that the database is accessible before proceeding.

### 3. Configure Environment Variables

Create a `.env` file in the project root, using the following configuration as a starting point:

```env
# Project
PROJECT_NAME=imdb-project

# Database
POSTGRES_DB=imdb
POSTGRES_USER=imdb_user
POSTGRES_PASSWORD=imdb_pass
DB_HOST=localhost
DB_PORT=5432

# API
API_URL=http://localhost:8000

# Connection strings
DATABASE_URL=postgresql://imdb_user:imdb_pass@localhost:5432/imdb
API_DATABASE_URL=postgresql+asyncpg://imdb_user:imdb_pass@localhost:5432/imdb
```

Update the credentials and port if your PostgreSQL installation uses different values.

The connection strings above use `localhost` because PostgreSQL runs directly on Windows. Make sure the environment variable names match what each module actually reads.

### 4. Set Up and Run the ETL Pipeline

Open Git Bash and navigate to the ingestion module:

```bash
cd ingest_module
```

Install the dependencies using `uv`:

```bash
uv sync
```

Run the ingestion command specified by the module's README or its Python entry point. For example, if the module exposes `main.py`:

```bash
uv run python main.py
```

Replace `main.py` with the actual entry point if your project uses a different filename or command.

The ingestion pipeline:

1. Downloads the IMDb datasets when necessary.
2. Detects remote file changes using ETags, where supported.
3. Processes the data in chunks to reduce memory consumption.
4. Loads actor and movie records into PostgreSQL.
5. Prepares the database for full-text search, according to the implemented schema and indexing logic.

Wait for ingestion to finish before starting searches against newly imported data.

### 5. Set Up and Run the API Server

Open a new Git Bash terminal from the project root:

```bash
cd server_module
uv sync
```

Start the FastAPI server using the module's configured entry point. If the application object is `app` in `src/main.py`, run:

```bash
uv run uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

Adjust the module path if the FastAPI application is defined elsewhere.

Once the server starts, visit:

- **API documentation:** http://127.0.0.1:8000/docs
- **Alternative API documentation:** http://127.0.0.1:8000/redoc
- **Health check:** http://127.0.0.1:8000/health

Keep this terminal running while using the API or CLI.

### 6. Set Up and Run the CLI

Open another Git Bash terminal:

```bash
cd cli_module
uv sync
```

Test the CLI:

```bash
uv run imdb --help
```

If the CLI is installed as an editable package through its project configuration, `uv sync` should make the `imdb` command available inside the project's virtual environment.

Example commands:

```bash
uv run imdb actor "Tom Hanks" --limit 5
uv run imdb movie "Inception" --limit 3
```

Ensure the API is running and that the CLI's API URL points to `http://localhost:8000`.

## Database Schema

### Actors Table

| Column | Type | Description |
|---|---|---|
| `nconst` | TEXT | IMDb person ID; primary key |
| `primary_name` | TEXT | Actor, director, or other person's name |
| `birth_year` | SMALLINT | Birth year |
| `death_year` | SMALLINT | Death year; nullable |
| `primary_profession` | TEXT | Primary professions |
| `search_vector` | TSVECTOR | Generated full-text search document |

### Movies Table

| Column | Type | Description |
|---|---|---|
| `tconst` | TEXT | IMDb title ID; primary key |
| `primary_title` | TEXT | Primary title |
| `original_title` | TEXT | Original-language title |
| `genres` | TEXT | Comma-separated genres |
| `search_vector` | TSVECTOR | Generated full-text search document |

Full-text search uses PostgreSQL `tsvector` columns and GIN indexes to support efficient text queries. The generated columns must be present in the database schema and maintained consistently with the SQLAlchemy models.

## Module Documentation

### Ingestion Module

See [`ingest_module/README.md`](ingest_module/README.md).

**Key features:**

- Streams IMDb data directly from the web.
- Uses ETag-based change detection to avoid unnecessary downloads.
- Processes large datasets in chunks for memory efficiency.
- Uses PostgreSQL `COPY` for faster bulk inserts.
- Processes millions of records efficiently.

**Supported datasets:**

- `name.basics.tsv.gz` → `actors`
- `title.basics.tsv.gz` → `movies`

### Server Module

See [`server_module/README.md`](server_module/README.md).

**Key features:**

- Search actors by name.
- Search movies by title or genres.
- PostgreSQL full-text search with GIN indexes.
- Asynchronous database queries.
- Automatic OpenAPI documentation.

**Main endpoints:**

- `GET /actors/search?name=<query>` – Search actors.
- `GET /movies/search?title=<query>` – Search movies.
- `GET /health` – Check API health.

### CLI Module

See [`cli_module/README.md`](cli_module/README.md).

**Key features:**

- Search actors and movies from the terminal.
- Configure the number of results.
- Display results in a readable format.
- Connect to a local or remote API.

**Usage:**

```bash
uv run imdb actor "Tom Hanks" --limit 5
uv run imdb movie "Inception" --limit 3
```

## Troubleshooting

### Database Connection Issues

- Confirm the PostgreSQL service is running.
- Verify the username, password, database name, host, and port in `.env`.
- Test the connection through pgAdmin or `psql`.
- Ensure the Python database driver required by each module is installed.

### API Not Responding

- Confirm the FastAPI server is running.
- Check the terminal for startup errors.
- Verify that the configured API URL and port match the running server.
- Visit http://127.0.0.1:8000/docs to verify that the application responds.

### Ingestion Failed

- Check your internet connection; IMDb datasets must be accessible online.
- Verify that the PostgreSQL user has permission to create or modify the required tables.
- Check available disk space and database storage.
- Review the ingestion module's logs for extraction, transformation, or loading errors.
- Confirm that the database schema matches the SQLAlchemy models.

### Search Not Working

- Verify that the `search_vector` columns exist in the `actors` and `movies` tables.
- Ensure the generated columns contain valid search vectors.
- Confirm that the GIN indexes have been created.
- Check that the API uses the correct database and connection string.

To inspect the actor table in `psql`:

```sql
\d actors
```

To inspect the movie table:

```sql
\d movies
```

Alternatively, use pgAdmin's Query Tool:

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name IN ('actors', 'movies')
ORDER BY table_name, ordinal_position;
```

## Performance Benchmarks

The following figures are approximate and depend on hardware, network speed, dataset size, and database configuration:

- **ETL process:** Approximately 50 seconds under the original benchmark conditions.
- **Search column and index creation:** Approximately 40 seconds under the original benchmark conditions.
- **API response time:** Under 100 ms for many queries when indexes are used effectively.
- **Initial setup:** Approximately 1–2 minutes under the original benchmark conditions, excluding dependency installation and any additional first-time setup.

These are reference figures, not guaranteed Windows performance measurements.

## Technology Stack

| Component | Technology |
|---|---|
| ETL Pipeline | Python, Pandas, SQLAlchemy |
| Database | PostgreSQL, `tsvector`, GIN indexes |
| API | FastAPI, AsyncPG |
| CLI | Python Click, Requests |
| Dependency Management | uv |
| Database Administration | pgAdmin, `psql` |
| Operating System | Windows 10/11 |
