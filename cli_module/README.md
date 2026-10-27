# IMDb CLI

A command-line interface built with Click for querying actors and movies from the IMDb dataset through the IMDb REST API. Simple, fast, and beautifully formatted.

## Features

- **Fast searches** – Queries the IMDb API through optimized endpoints.
- **Flexible limits** – Display a custom number of results.
- **Error handling** – Clear error messages for connection issues, timeouts, and API errors.
- **Beautiful formatting** – Human-readable output with contextual information.

## Project Structure

```text
cli_module/
├── README.md
├── imdb_cli/
│   ├── __init__.py
│   ├── formatters.py       # Output formatting utilities
│   ├── main.py             # CLI entry point and commands
│   └── models.py           # Pydantic models for API responses
├── test/
│   ├── __init__.py
│   ├── test_cli_actor.py   # Actor command unit tests
│   ├── test_cli_movie.py   # Movie command unit tests
│   └── test_formatter.py   # Formatter utility tests
├── pyproject.toml          # Dependencies and metadata
└── uv.lock                 # Dependency lock file
```

## Installation on Windows

### Prerequisites

- Windows 10 or Windows 11.
- Python 3.11 or later.
- Git for Windows with Git Bash.
- `uv` Python package manager.
- The FastAPI server running and accessible at the configured API URL.

### Step 1: Install uv

Open Windows Terminal, Command Prompt, or PowerShell and run:

```powershell
winget install --id=astral-sh.uv -e
```

Close and reopen your terminal if necessary, then verify the installation in Git Bash:

```bash
uv --version
```

Alternatively, follow the official installation instructions:

https://docs.astral.sh/uv/getting-started/installation/

### Step 2: Navigate to the CLI Module

Open Git Bash and navigate to the `cli_module` directory:

```bash
cd path/to/your/project/cli_module
```

Replace the example path with the actual location of your project.

### Step 3: Install Dependencies

Run:

```bash
uv sync
```

This creates the `.venv` virtual environment if needed and installs the project's dependencies from `pyproject.toml` and `uv.lock`.

If the CLI is configured as an installable project with an appropriate script entry point, `uv sync` will also make the `imdb` command available inside the environment.

### Step 4: Verify the Installation

```bash
uv run imdb --help
```

You should see the available CLI commands and their options.

You can also activate the virtual environment in Git Bash:

```bash
source .venv/Scripts/activate
```

After activation, you can run commands directly:

```bash
imdb --help
```

## Configuration

The CLI needs to know the URL of the FastAPI server.

Configure the API URL using the environment variable or configuration mechanism implemented by the CLI. For example, if your application reads `API_URL` from the root `.env` file:

```env
API_URL=http://localhost:8000
```

Ensure the `.env` file is located where the application expects it and that the variable name matches the implementation.

Start the server from `server_module` before running searches. See the main project README and the Server Module README for database setup and API startup instructions.

## Usage

The CLI provides two main commands for searching the IMDb dataset.

### Search for Actors

```bash
uv run imdb actor "Tom Hanks"
```

**Options:**

- `--limit` – Number of results to display. Defaults to `1`.

**Examples:**

Show the first matching actor:

```bash
uv run imdb actor "Bruce Lee"
```

Example output:

```text
Found 1 actor.
Showing 1 actor.
1. Bruce Lee was born in 1940 and had the following professions: actor, writer, and miscellaneous.
```

Show up to five matching actors:

```bash
uv run imdb actor "Robert" --limit 5
```

Example output:

```text
Found 4610 actors.
Showing 5 actors.
1. Robert P. was born in 1982 and has the following profession: actor.
2. Robert An was born in 2003 and has the following profession: actor.
3. Robert Ray was born in 1978 and had the following profession: archive footage.
4. Robert Joy was born in 1951 and has the following professions: actor, writer, and composer.
5. Robert Lee was born in 1957 and had the following professions: miscellaneous, actor, and archive footage.
```

The output includes:

- Actor name.
- Birth year, when available.
- Life status reflected through the wording `has` for living persons and `had` for deceased persons.
- Primary professions.

*Example output is illustrative; actual results depend on the database contents and search implementation.*

### Search for Movies

```bash
uv run imdb movie "Shrek"
```

**Options:**

- `--limit` – Number of results to display. Defaults to `1`.

**Examples:**

Show the first matching movie:

```bash
uv run imdb movie "Iron Man"
```

Example output:

```text
Found 553 movies.
Showing 1 movie.
1. Iron Man, originally titled 'Iron Man', is an Action, a Crime, and a Drama.
```

Show up to ten matching movies:

```bash
uv run imdb movie "Blacksmith Scene" --limit 10
```

Example output:

```text
Found 3 movies.
Showing 3 movies.
1. Blacksmith Scene, originally titled 'Blacksmith Scene', is a Short.
2. Blacksmith Scene, originally titled 'Les forgerons', is a Documentary and a Short.
3. Blacksmith Rates 9 Forging Scenes from Movies and TV, originally titled 'Blacksmith Rates 9 Forging Scenes from Movies and TV', is a Documentary and a Short.
```

The output includes:

- Primary movie title.
- Original title.
- Genres.

*The examples demonstrate the output format; the number and order of actual results may vary.*

### Get Help

Display help for the CLI or a specific command:

```bash
uv run imdb --help
uv run imdb actor --help
uv run imdb movie --help
```

If you have activated the virtual environment, you can omit `uv run`.

## Requirements

Dependencies are managed through `pyproject.toml` and installed using `uv sync`.

Key dependencies include:

- `click>=8.3.1` – Command-line interface creation.
- `pydantic>=2.12.5` – Data validation.
- `python-dotenv>=1.0.0` – Environment variable management.
- `requests>=2.32.5` – HTTP requests.
- `pytest>=9.0.2` – Testing framework.

The actual versions installed are determined by the project's dependency configuration and lockfile.

## Testing

Unit tests are provided in the `test/` directory using pytest.

### Run Tests

From the `cli_module` directory, run:

```bash
uv run pytest
```

Run individual test files:

```bash
uv run pytest test/test_cli_actor.py
uv run pytest test/test_cli_movie.py
uv run pytest test/test_formatter.py
```

### Actor Command Tests

The actor command tests cover:

- `test_invalid_limit_validation` – Validates invalid limit values such as `0`, `-1`, and `-500`.
- `test_no_actors_found` – Handles empty API responses.
- `test_single_actor_success` – Tests a successful single-actor query.
- `test_multiple_actors_with_limit` – Tests multiple results with a limit.
- `test_http_404_error` – Handles HTTP 404 errors.
- `test_http_500_error` – Handles HTTP 500 errors.
- `test_connection_error` – Handles connection failures.
- `test_timeout_error` – Handles request timeouts.
- `test_generic_request_exception` – Handles other request exceptions.
- `test_generic_exception` – Handles unexpected exceptions.

### Movie Command Tests

The movie command tests cover:

- `test_invalid_movie_limit` – Validates invalid movie limits.
- `test_no_movies_found` – Handles empty API responses.
- `test_single_movie_success` – Tests a successful single-movie query.
- `test_multiple_movies_with_limit` – Tests multiple results with a limit.
- `test_http_404_error` – Handles HTTP 404 errors.
- `test_http_500_error` – Handles HTTP 500 errors.
- `test_connection_error` – Handles connection failures.
- `test_timeout_error` – Handles request timeouts.
- `test_generic_request_exception` – Handles other request exceptions.
- `test_generic_exception` – Handles unexpected exceptions.

### Formatter Tests

**Profession formatting:**

- `test_format_professions_single_actor` – Formats a single profession.
- `test_format_professions_multiple` – Formats multiple professions.
- `test_format_professions_none_or_empty` – Handles missing or empty professions.

**Genre formatting:**

- `test_format_genres_single` – Formats a single genre.
- `test_format_genres_multiple` – Formats multiple genres.
- `test_format_genres_none_or_empty` – Handles missing or empty genres.

**Life status formatting:**

- `test_is_dead` – Returns `had` for deceased persons.
- `test_is_alive` – Returns `has` for living persons.
- `test_is_dead_none` – Handles missing death-year values.

**Pluralization:**

- `test_plural_s_singular` – Returns an empty suffix for a count of one.
- `test_plural_s_plural_or_zero` – Returns `s` for counts such as zero, two, and one hundred.

## Troubleshooting

### Connection Error

Ensure the FastAPI server is running and that the CLI is configured with the correct API URL.

For a local server, the default URL may be:

```text
http://localhost:8000
```

You can verify that the server responds by opening:

http://127.0.0.1:8000/docs

### Module Not Found

If the `imdb` command cannot be found, run:

```bash
uv sync
uv run imdb --help
```

Check that `pyproject.toml` defines the correct console script entry point and that the project is configured for installation.

If necessary, reinstall the local package:

```bash
uv sync --reinstall
```

### Timeout or HTTP Errors

- Confirm the API server is running.
- Check the server terminal for database or application errors.
- Verify the configured API URL and port.
- Confirm that the API endpoints match the paths used by the CLI.

## Technology Stack

| Component | Technology |
|---|---|
| CLI Framework | Click |
| API Communication | Requests |
| Data Validation | Pydantic |
| Configuration | python-dotenv |
| Testing | Pytest |
| Dependency Management | uv |
| Backend API | FastAPI |
| Operating System | Windows 10/11 |
