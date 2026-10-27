# Server Module

A FastAPI-based REST API server for the IMDb project that provides search capabilities for movies and actors with full-text search support.

## Features

- **RESTful API endpoints** – Search and retrieve actors and movies.
- **Full-text search** – Optimized searching with PostgreSQL GIN indexes on `tsvector` columns.
- **Async database queries** – Non-blocking database operations with AsyncPG.
- **Automatic documentation** – Interactive OpenAPI (Swagger) documentation at `/docs`.
- **Request validation** – Pydantic models for input validation.
- **Error handling** – Comprehensive error responses and logging.
- **Health checks** – `/health` endpoint for monitoring.

## Project Structure

```text
server_module/
├── main.py                 # FastAPI application entry point
├── core/
│   ├── __init__.py
│   ├── config.py           # Configuration management
│   ├── database.py         # Database connection and setup
│   └── logger.py           # Logging configuration
├── src/
│   ├── __init__.py
│   ├── actors/
│   │   ├── __init__.py
│   │   ├── models.py       # SQLAlchemy models
│   │   ├── schemas.py      # Pydantic validation schemas
│   │   ├── repository.py   # Database queries
│   │   ├── service.py      # Business logic
│   │   └── routes.py       # API endpoints
│   └── movies/
│       ├── __init__.py
│       ├── models.py       # SQLAlchemy models
│       ├── schemas.py      # Pydantic validation schemas
│       ├── repository.py   # Database queries
│       ├── service.py      # Business logic
│       └── routes.py       # API endpoints
├── tests/
│   ├── __init__.py
│   ├── test_actors.py     # Actor repository tests
│   └── test_movies.py     # Movie repository tests
├── pyproject.toml          # Dependencies and project metadata
└── README.md               # This file
```

## Architecture

### Layered Structure

The server follows a layered architecture that separates HTTP handling, business logic, database access, and data validation.

1. **Routes Layer** (`routes.py`) – Defines HTTP endpoints and handles incoming requests.
2. **Service Layer** (`service.py`) – Implements business logic.
3. **Repository Layer** (`repository.py`) – Handles database queries.
4. **Models Layer** (`models.py`) – Defines SQLAlchemy ORM models.
5. **Schemas Layer** (`schemas.py`) – Validates request data and structures responses.

### Database Connection

- Async SQLAlchemy with AsyncPG for non-blocking database operations.
- Connection pooling managed by SQLAlchemy.
- A database session per request, managed through FastAPI dependency injection.

### Full-Text Search

Initially, the API used `LIKE` queries to search movie titles. These queries had relatively high response times, around 1.5 seconds under the original benchmark conditions.

To improve performance, PostgreSQL full-text search was introduced using a `tsvector` column generated from movie titles, alongside a GIN index. The same approach can be applied to actor names.

This significantly reduced search response times in the original tests, bringing typical queries below 100 ms.

The search implementation uses PostgreSQL's full-text search operators and query functions. The generated columns and indexes must exist in the database schema before the corresponding queries can run.

#### Before optimization

![Search performance before optimization](../docs/before_optimization.png)

#### After optimization

![Search performance after optimization](../docs/after_optimization.png)

## Installation

### Prerequisites

- Windows 10 or Windows 11.
- Python 3.11 or higher.
- PostgreSQL 18 or another compatible PostgreSQL version, installed and running locally.
- Git for Windows.
- `uv` Python package manager.
- A configured `.env` file with valid database credentials.

### Step 1: Install uv

Install `uv` using the official Windows installation instructions:

https://docs.astral.sh/uv/getting-started/installation/

Alternatively, if using Git Bash with a compatible `curl` installation, follow the official installer instructions provided by Astral.

Verify the installation:

```bash
uv --version
```

### Step 2: Configure the Environment

Create a `.env` file in the project root or in the location expected by the application configuration.

Example:

```env
PROJECT_NAME=imdb-project

POSTGRES_DB=imdb
POSTGRES_USER=imdb_user
POSTGRES_PASSWORD=imdb_pass
DB_HOST=localhost
DB_PORT=5432

API_URL=http://localhost:8000

DATABASE_URL=postgresql://imdb_user:imdb_pass@localhost:5432/imdb
API_DATABASE_URL=postgresql+asyncpg://imdb_user:imdb_pass@localhost:5432/imdb
```

Update the values to match your PostgreSQL configuration. The environment variable names must match those used in `core/config.py`.

Ensure that the database and user exist and that the user has permission to access the database.

### Step 3: Install Dependencies

Open Git Bash and navigate to the server module directory:

```bash
cd path/to/your/project/server_module
```

Install the project's dependencies:

```bash
uv sync
```

This creates the virtual environment and installs the dependencies defined in `pyproject.toml` and its lockfile, when present.

### Step 4: Prepare the Database

Ensure PostgreSQL is running on Windows.

If you have not already loaded the IMDb datasets, run the ingestion pipeline from the `ingest_module` according to its README.

Before starting the API, verify that:

- The `actors` and `movies` tables exist.
- The required columns match the SQLAlchemy models.
- The `search_vector` generated columns exist.
- The GIN indexes have been created.

You can inspect the schema through pgAdmin or `psql`.

For example:

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name IN ('actors', 'movies')
ORDER BY table_name, ordinal_position;
```

### Step 5: Run the Server

From the `server_module` directory, start the FastAPI application:

```bash
uv run uvicorn main:app --reload --host 127.0.0.1 --port 8000
```

This command assumes that `main.py` is located directly in `server_module/` and exposes a FastAPI instance named `app`.

If your application uses a different module path, adjust the Uvicorn import string accordingly.

The API will be available at:

http://127.0.0.1:8000/

Keep the terminal running while you use the API.

## Interactive Documentation

Once the server is running, visit:

- **Swagger UI:** http://127.0.0.1:8000/docs
- **ReDoc:** http://127.0.0.1:8000/redoc

These interfaces let you inspect endpoints, submit test requests, and review the API's request and response schemas.

## API Endpoints

### Health Check

`GET /health`

Checks whether the API is running.

### Actor Search

`GET /actors/search?name=<query>`

Searches for actors by name.

**Query parameters:**

- `name` (string, required) – The actor name or search phrase.

**Example:**

```text
http://127.0.0.1:8000/actors/search?name=Tom%20Hanks
```

**Response:** A list of matching actors with the details defined by the response schema.

### Movie Search

`GET /movies/search?title=<query>`

Searches for movies by title.

**Query parameters:**

- `title` (string, required) – The movie title or search phrase.

**Example:**

```text
http://127.0.0.1:8000/movies/search?title=Inception
```

**Response:** A list of matching movies, including genres and other fields defined by the response schema.

## Dependencies

The project's dependencies are managed through `pyproject.toml` and installed using `uv sync`.

Key dependencies include:

- `fastapi>=0.109.0` – Web framework.
- `uvicorn>=0.27.0` – ASGI server.
- `sqlalchemy>=2.0.0` – Database ORM.
- `asyncpg>=0.29.0` – Asynchronous PostgreSQL driver.
- `pydantic>=2.0.0` – Data validation.
- `python-dotenv>=1.0.0` – Environment variable loading.
- `psycopg2-binary>=2.9.0` – PostgreSQL adapter, if required by synchronous database operations or other project components.
- `pytest>=9.0.2` – Testing framework.

The actual installed versions are determined by the project's dependency configuration and lockfile.

## Testing

Unit tests are provided in the `tests/` directory.

### Run Tests

From the `server_module` directory, run:

```bash
uv run pytest
```

To run individual test files:

```bash
uv run pytest tests/test_actors.py
uv run pytest tests/test_movies.py
```

### Test Files

**`tests/test_actors.py`**

- `TestActorRepository.test_get_by_name_returns_actors` – Tests successful actor searches.
- `TestActorRepository.test_get_by_name_not_found_raises_404` – Tests handling of missing actors.

**`tests/test_movies.py`**

- `TestMovieRepository.test_get_by_title_returns_movies` – Tests successful movie searches.
- `TestMovieRepository.test_get_by_title_not_found_raises_404` – Tests handling of missing movies.

Test names above describe the expected test cases; verify the actual test methods in your files.

## Performance Notes

- **Search response time:** Under 100 ms for typical queries in the original benchmark conditions.
- **Connection pooling:** SQLAlchemy manages database connections through a configurable pool.
- **Async processing:** Non-blocking database I/O supports concurrent API requests.
- **Full-text search:** GIN indexes can improve search performance on appropriately designed `tsvector` columns.

Performance depends on database size, query structure, indexes, available resources, and workload.

## Troubleshooting

### Database Connection Errors

- Confirm the PostgreSQL service is running.
- Verify the host, port, username, password, and database name.
- Ensure the API is using the correct database connection string.
- Check that the required Python dependencies are installed.

### Missing `search_vector` Column

If you encounter an error such as:

```text
UndefinedColumnError: column actors.search_vector does not exist
```

the database schema does not match the model or query.

Inspect the `actors` and `movies` tables, then apply the appropriate schema migration or database setup procedure to add the missing generated column. Do not assume that changing a SQLAlchemy model automatically changes an existing PostgreSQL table.

### API Startup Errors

- Confirm that `main.py` exposes an object named `app`.
- Check the terminal for import errors or configuration failures.
- Verify that port `8000` is available.
- Confirm that the `.env` file is loaded from the expected location.

### Search Returns No Results

- Verify that the relevant dataset has been ingested.
- Check the full-text search expressions in the repository layer.
- Confirm that the generated search vectors contain the expected text.
- Ensure the GIN indexes exist and the search query uses a compatible PostgreSQL operator.

## Technology Stack

| Component | Technology |
|---|---|
| Web Framework | FastAPI |
| ASGI Server | Uvicorn |
| Database | PostgreSQL |
| ORM | SQLAlchemy |
| Async Driver | AsyncPG |
| Data Validation | Pydantic |
| Full-Text Search | PostgreSQL `tsvector` and GIN indexes |
| Testing | Pytest |
| Dependency Management | uv |
| Operating System | Windows 10/11 |
