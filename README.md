# BrainForces

[![Django CI](https://github.com/fivan999/BrainForces/actions/workflows/django.yml/badge.svg)](https://github.com/fivan999/BrainForces/actions/workflows/django.yml)
[![Python package](https://github.com/fivan999/BrainForces/actions/workflows/python-package.yml/badge.svg)](https://github.com/fivan999/BrainForces/actions/workflows/python-package.yml)

## What Is This?

BrainForces is a Django web application for running competitive online quizzes. It supports user registration, quiz participation, answer submission, standings, user ratings, question archives, and organization-owned quizzes and posts.

The project is structured as a server-rendered Django application with HTML templates, Django class-based views, Django ORM models, and optional Docker/PostgreSQL runtime support.

## Key Functionality

- User signup, login, logout, password reset, password change, profile editing, avatars, and account activation by email.
- Login attempt tracking with account deactivation after a configured number of failed attempts.
- Public and private organizations with user roles: invited, participant, admin, and creator.
- Organization member management, invitations, posts, private posts, and post comments.
- Quiz listing, search, registration, timed access, question pages, answer submission, user answer history, and standings.
- Organization admins can create quizzes with questions and answer variants through application forms.
- Quiz results can update solved counts, placements, and user ratings for rated public quizzes.
- Archive page for searching questions by name, text, or tags.
- Django admin and CKEditor upload routes are enabled.

## Architecture & Technology

BrainForces is a monolithic Django 3.2 application split into domain apps: `users`, `quiz`, `organization`, `archive`, `homepage`, `about`, and `core`. The app uses Django templates, class-based views, Django ORM models, migrations, custom managers, and a custom user model that authenticates by email.

The default local database is SQLite. PostgreSQL can be enabled with environment variables, and Docker Compose provides a web service plus a PostgreSQL 15.2 service. Rich text and uploads are handled with `django-ckeditor`; thumbnails use `sorl-thumbnail`; debug tooling uses `django-debug-toolbar` when `DEBUG=True`.

## Prerequisites

- Python 3.9 or 3.10
- `pip`
- Docker and Docker Compose, if running with containers
- PostgreSQL, if running locally with `USE_POSTGRES=True`

The Docker image is based on `python:3.10-alpine3.16`.

## Installation

Clone the repository:

```bash
git clone https://github.com/fivan999/BrainForces
cd BrainForces
```

Create and activate a virtual environment:

```bash
python -m venv venv
source venv/bin/activate
```

On Windows:

```powershell
python -m venv venv
venv\Scripts\activate
```

Install runtime dependencies:

```bash
pip install -r requirements/base.txt
```

For development and tests, also install:

```bash
pip install -r requirements/dev.txt
pip install -r requirements/test.txt
```

Create an environment file from the example:

```bash
cp brainforces/.env.example brainforces/.env
```

Review the values in `brainforces/.env` before starting the application.

## Environment Variables

The application loads environment variables with `python-dotenv`. The code calls `dotenv.load_dotenv()` from `brainforces/brainforces/settings.py`; when running commands from the repository root, either export variables in your shell or pass an env file explicitly. Docker commands in this README use `--env-file brainforces/.env`.

Boolean values are treated as true when set to one of: `true`, `y`, `1`, or `yes` after lowercasing.

| Name | Required | Description | Example |
|------|----------|-------------|---------|
| `SECRET_KEY` | No | Django secret key. Defaults to `default`, but a real value should be used outside local development. | `django-insecure-change-me` |
| `DEBUG` | No | Enables Django debug mode and debug toolbar. Defaults to `True`. | `True` |
| `ALLOWED_HOSTS` | Required when `DEBUG=False` | Space-separated allowed hosts. Ignored when `DEBUG=True`, where hosts are set to `*`. | `127.0.0.1 localhost` |
| `INTERNAL_IPS` | No | Space-separated internal IPs for `django-debug-toolbar`. Defaults to `127.0.0.1`. | `127.0.0.1` |
| `USE_POSTGRES` | No | Enables PostgreSQL when true. If false, or when running tests, SQLite is used. Defaults to `False`. | `True` |
| `DB_NAME` | Required when PostgreSQL is used | PostgreSQL database name. Also feeds `POSTGRES_DB` in Docker Compose. | `project` |
| `DB_USER` | Required when PostgreSQL is used | PostgreSQL username. Also feeds `POSTGRES_USER` in Docker Compose. | `user` |
| `DB_PASS` | Required when PostgreSQL is used | PostgreSQL password. Also feeds `POSTGRES_PASSWORD` in Docker Compose. | `1234` |
| `DB_HOST` | Required when PostgreSQL is used | PostgreSQL host. Docker Compose sets this to `database` for the web container. | `127.0.0.1` |
| `DB_PORT` | No | PostgreSQL port. Only added to Django database settings when provided. | `5432` |
| `USER_IS_ACTIVE` | No | Controls whether new users are active immediately after signup. Defaults to `False`. | `False` |
| `LOGIN_ATTEMPTS` | No | Number of failed login attempts allowed before account deactivation. Defaults to `3`. | `3` |
| `LOGIN_ATTEMPS` | No | Present in `.env.example`, but the code reads `LOGIN_ATTEMPTS`. Keep the correctly spelled variable. | `3` |
| `EMAIL` | No | Email value stored in settings and used by email-related configuration. | `brainforces@example.com` |
| `USE_REAL_EMAIL` | No | Enables SMTP email settings when set. If unset, Django uses the file-based email backend. | `True` |
| `EMAIL_HOST` | Required when `USE_REAL_EMAIL` is set | SMTP host. | `smtp.yandex.ru` |
| `EMAIL_PORT` | Required when `USE_REAL_EMAIL` is set | SMTP port. | `587` |
| `EMAIL_USE_TLS` | No | Enables TLS for SMTP when set. Takes precedence over `EMAIL_USE_SSL` in settings. | `True` |
| `EMAIL_USE_SSL` | No | Enables SSL for SMTP when `EMAIL_USE_TLS` is not set. | `False` |
| `EMAIL_HOST_USER` | Required when `USE_REAL_EMAIL` is set | SMTP username. Also used as `SERVER_EMAIL` and `DEFAULT_FROM_EMAIL`. | `brainforces@yandex.ru` |
| `EMAIL_HOST_PASSWORD` | Required when `USE_REAL_EMAIL` is set | SMTP password. | `YOUR_PASSWORD` |
| `PYTHONUNBUFFERED` | No | Set by Docker Compose for unbuffered Python output in the web container. | `1` |
| `POSTGRES_DB` | Docker internal | PostgreSQL container variable derived from `DB_NAME` in `docker-compose.yml`. | `${DB_NAME}` |
| `POSTGRES_USER` | Docker internal | PostgreSQL container variable derived from `DB_USER` in `docker-compose.yml`. | `${DB_USER}` |
| `POSTGRES_PASSWORD` | Docker internal | PostgreSQL container variable derived from `DB_PASS` in `docker-compose.yml`. | `${DB_PASS}` |
| `TZ` | Docker internal | Time zone set in the Docker image. | `Europe/Moscow` |
| `DJANGO_SETTINGS_MODULE` | Usually no | Set by `manage.py`, `asgi.py`, and `wsgi.py` to load Django settings. | `brainforces.settings` |

## Running Locally

Run migrations:

```bash
python brainforces/manage.py migrate
```

Start the development server:

```bash
python brainforces/manage.py runserver
```

Open:

```text
http://127.0.0.1:8000/
```

Useful routes include:

- `/` - homepage with public organization posts
- `/quiz/` - quiz list
- `/archive/` - archived question search
- `/organizations/` - organization list
- `/auth/signup/` - signup
- `/auth/login/` - login
- `/admin/` - Django admin
- `/about/` - about page
- `/__debug__/` - debug toolbar routes when `DEBUG=True`

To create an admin user:

```bash
python brainforces/manage.py createsuperuser
```

## Development

Install all development dependencies:

```bash
pip install -r requirements/base.txt
pip install -r requirements/dev.txt
pip install -r requirements/test.txt
```

Run the Django test suite:

```bash
cd brainforces
python manage.py test
```

Run the same style checks configured in GitHub Actions:

```bash
flake8 . --count --show-source --statistics
black . --check
isort . --check
```

The repository also includes `mypy` and `djlint` in `requirements/dev.txt`; formatting and lint configuration lives in `pyproject.toml` and `setup.cfg`.

## Build & Deployment

### Docker Compose

Build the application image:

```bash
docker-compose --env-file brainforces/.env build
```

Apply migrations:

```bash
docker-compose --env-file brainforces/.env run --rm web-app sh -c "python manage.py migrate"
```

Start the web app and PostgreSQL:

```bash
docker-compose --env-file brainforces/.env up
```

The application is exposed on:

```text
http://127.0.0.1:8000/
```

### CI

GitHub Actions runs two workflows on pushes and pull requests to `main`:

- `Django CI`: installs runtime, dev, and test dependencies, then runs `python manage.py test` from `brainforces/`.
- `Python package`: installs dependencies, then runs `flake8`, `black --check`, and `isort --check`.

## Project Structure

```text
.
|-- brainforces/
|   |-- manage.py
|   |-- brainforces/        # Django settings, root URLs, ASGI/WSGI
|   |-- users/              # Custom user model, auth views, profiles
|   |-- quiz/               # Quizzes, questions, answers, results, standings
|   |-- organization/       # Organizations, roles, posts, comments
|   |-- archive/            # Question archive and search
|   |-- homepage/           # Homepage post listing
|   |-- about/              # Static about page
|   |-- core/               # Shared abstract model utilities
|   |-- templates/          # Django templates
|   |-- fixtures/           # Seed fixture data
|   `-- media/              # Uploaded media committed in this repository
|-- requirements/
|   |-- base.txt            # Runtime dependencies
|   |-- dev.txt             # Linting, formatting, typing, template tools
|   `-- test.txt            # Test-only dependencies
|-- Dockerfile
|-- docker-compose.yml
|-- pyproject.toml
`-- setup.cfg
```

## Troubleshooting

### PostgreSQL is not used

PostgreSQL is only selected when `USE_POSTGRES` is true and the command is not a Django test run. If `USE_POSTGRES` is false or missing, Django uses local SQLite at `db.sqlite3`.

### Docker database host

When running with Docker Compose, the web container uses `DB_HOST=database` from `docker-compose.yml`. For a local PostgreSQL server outside Docker, use a host such as `127.0.0.1` and set `DB_PORT` if needed.

### Account activation email

New users are inactive by default because `USER_IS_ACTIVE` defaults to `False`. In that mode, signup sends an activation email. Configure SMTP with `USE_REAL_EMAIL=True` and the `EMAIL_*` variables, or set `USER_IS_ACTIVE=True` for local development.

### Email backend caveat

When `USE_REAL_EMAIL` is not set, settings configure Django's file-based email backend and write messages under `brainforces/sent_emails`. However, the current activation email service references `EMAIL_HOST_USER`, so activation flows are expected to need SMTP configuration unless user activation is disabled with `USER_IS_ACTIVE=True`.

### `.env.example` typo

The example file contains `LOGIN_ATTEMPS`, but the Django settings read `LOGIN_ATTEMPTS`. Use `LOGIN_ATTEMPTS` in real environments.

## Additional Notes

- The application language is configured as Russian (`LANGUAGE_CODE='ru'`) and the Django time zone is `Europe/Moscow`.
- Static files use `STATIC_URL=/static/`; uploaded media uses `MEDIA_URL=/media/` and `MEDIA_ROOT=brainforces/media`.
- CKEditor uploads are stored under `uploads/` inside the configured media directory.
- Tests always use SQLite because settings switch to SQLite when `test` is present in command arguments.
