# FastAPI Project - Development

## Getting Started

### Development Environment Setup

To get your local development environment configured, follow these setup steps in order from the root of the project. After this, you'll be ready to run the application.

1.  **Configure Environment**:
    Create a `.env` file for your local setup. You can copy the example file.
    ```bash
    cp .env.example .env
    ```
    Then, generate new secret keys for `SECRET_KEY`, `FIRST_SUPERUSER_PASSWORD`, and `POSTGRES_PASSWORD` and update your `.env` file. You can generate a key with:
    ```bash
    python -c "import secrets; print(secrets.token_urlsafe(32))"
    ```

    > **Note**: The `FIRST_SUPERUSER_PASSWORD` must be 40 characters or less, so you may need to remove a few characters if using the above Python code snippet to generate a value for it.

2.  **Install Backend Dependencies**:
    Navigate to the `backend` directory to install the Python dependencies.
    ```bash
    cd backend
    uv sync
    ```

3.  **Install Git Pre-commit Hooks**:
    While still in the `backend` directory, install the pre-commit hooks to ensure code quality.
    ```bash
    uv run pre-commit install
    cd ..
    ```

4.  **Install Frontend Dependencies**:
    Navigate to the `frontend` directory to install the Node.js dependencies. This requires `nvm` or `fnm`.
    ```bash
    cd frontend
    nvm install
    nvm use
    npm install
    cd ..
    ```

5.  **Generate Frontend API Client**:
    Finally, generate the type-safe client for the frontend to communicate with the backend.
    ```bash
    # Make sure the script is executable
    chmod +x ./scripts/generate-client.sh
    # Activate backend environment and run script
    source backend/.venv/bin/activate
    ./scripts/generate-client.sh
    ```

### Running the Application for Development

With the setup complete, you can now run the application. There are two main workflows for local development, depending on whether you prioritize a production-like environment or faster iteration with local tooling.

#### Workflow 1: Fully Dockerized Development

This is the simplest way to get all services running in an environment that closely mimics production.

**To start all services:**
```bash
docker compose watch
```

**To stop all services:**
```bash
docker compose down
```
> **Note**: By default, this command preserves the database data. If you want a completely fresh start and to delete all data, use the `--volumes` (or `-v`) flag: `docker compose down -v`.

#### Workflow 2: Hybrid Development (Recommended for Active Coding)

For a faster development workflow with full debugger support and live reloading, run the application code directly on your machine while using Docker for backing services like the database.

1.  **Start the Database and other services**:
    In a terminal, start the services your application depends on.
    ```bash
    docker compose up -d db mailcatcher adminer
    ```

2.  **Run the Backend Locally**:
    In a separate terminal, from the project root, activate the virtual environment and start the backend.
    ```bash
    source backend/.venv/bin/activate
    fastapi dev backend/app/main.py
    ```

3.  **Run the Frontend Locally**:
    In a third terminal, start the frontend development server.
    ```bash
    cd frontend
    npm run dev
    ```

When you are finished, stop the Docker services:
```bash
docker compose down
```

Now you should be up-to-speed with the basics of development. For more in-depth information, see below.

## Docker Compose

* Start the local stack with Docker Compose:

```bash
docker compose watch
```

* Now you can open your browser and interact with these URLs:

Frontend, built with Docker, with routes handled based on the path: http://localhost:5173

Backend, JSON based web API based on OpenAPI: http://localhost:8000

Automatic interactive documentation with Swagger UI (from the OpenAPI backend): http://localhost:8000/docs

Adminer, database web administration: http://localhost:8080

Traefik UI, to see how the routes are being handled by the proxy: http://localhost:8090

**Note**: The first time you start your stack, it might take a minute for it to be ready. While the backend waits for the database to be ready and configures everything. You can check the logs to monitor it.

To check the logs, run (in another terminal):

```bash
docker compose logs
```

To check the logs of a specific service, add the name of the service, e.g.:

```bash
docker compose logs backend
```

## Local Development

The Docker Compose files are configured so that each of the services is available in a different port in `localhost`.

For the backend and frontend, they use the same port that would be used by their local development server, so, the backend is at `http://localhost:8000` and the frontend at `http://localhost:5173`.

This way, you could turn off a Docker Compose service and start its local development service, and everything would keep working, because it all uses the same ports.

For example, you can stop that `frontend` service in the Docker Compose, in another terminal, run:

```bash
docker compose stop frontend
```

And then start the local frontend development server:

```bash
cd frontend
npm run dev
```

Or you could stop the `backend` Docker Compose service:

```bash
docker compose stop backend
```

And then you can run the local development server for the backend:

```bash
cd backend
fastapi dev app/main.py
```

## Docker Compose in `localhost.tiangolo.com`

When you start the Docker Compose stack, it uses `localhost` by default, with different ports for each service (backend, frontend, adminer, etc).

When you deploy it to production (or staging), it will deploy each service in a different subdomain, like `api.example.com` for the backend and `dashboard.example.com` for the frontend.

In the guide about [deployment](deployment.md) you can read about Traefik, the configured proxy. That's the component in charge of transmitting traffic to each service based on the subdomain.

If you want to test that it's all working locally, you can edit the local `.env` file, and change:

```dotenv
DOMAIN=localhost.tiangolo.com
```

That will be used by the Docker Compose files to configure the base domain for the services.

Traefik will use this to transmit traffic at `api.localhost.tiangolo.com` to the backend, and traffic at `dashboard.localhost.tiangolo.com` to the frontend.

The domain `localhost.tiangolo.com` is a special domain that is configured (with all its subdomains) to point to `127.0.0.1`. This way you can use that for your local development.

After you update it, run again:

```bash
docker compose watch
```

When deploying, for example in production, the main Traefik is configured outside of the Docker Compose files. For local development, there's an included Traefik in `docker-compose.override.yml`, just to let you test that the domains work as expected, for example with `api.localhost.tiangolo.com` and `dashboard.localhost.tiangolo.com`.

## Docker Compose files and env vars

There is a main `docker-compose.yml` file with all the configurations that apply to the whole stack, it is used automatically by `docker compose`.

And there's also a `docker-compose.override.yml` with overrides for development, for example to mount the source code as a volume. It is used automatically by `docker compose` to apply overrides on top of `docker-compose.yml`.

These Docker Compose files use the `.env` file containing configurations to be injected as environment variables in the containers.

They also use some additional configurations taken from environment variables set in the scripts before calling the `docker compose` command.

After changing variables, make sure you restart the stack:

```bash
docker compose watch
```

## The .env file

The `.env` file is the one that contains all your configurations, generated keys and passwords, etc.

Depending on your workflow, you could want to exclude it from Git, for example if your project is public. In that case, you would have to make sure to set up a way for your CI tools to obtain it while building or deploying your project.

One way to do it could be to add each environment variable to your CI/CD system, and updating the `docker-compose.yml` file to read that specific env var instead of reading the `.env` file.

## Pre-commits and code linting

we are using a tool called [pre-commit](https://pre-commit.com/) for code linting and formatting.

When you install it, it runs right before making a commit in git. This way it ensures that the code is consistent and formatted even before it is committed.

You can find a file `.pre-commit-config.yaml` with configurations at the root of the project.

#### Install pre-commit to run automatically

`pre-commit` is already part of the dependencies of the project, but you could also install it globally if you prefer to, following [the official pre-commit docs](https://pre-commit.com/).

After having the `pre-commit` tool installed and available, you need to "install" it in the local repository, so that it runs automatically before each commit.

First, install the backend dependencies, from the `backend` directory:

```bash
cd backend
uv sync
```

Then, still in the `backend` directory, install the pre-commit hooks:

```bash
uv run pre-commit install
```

And now you can go back to the project root directory:

```bash
cd ..
```

Now whenever you try to commit, e.g. with:

```bash
git commit
```

...pre-commit will run and check and format the code you are about to commit, and will ask you to add that code (stage it) with git again before committing.

Then you can `git add` the modified/fixed files again and now you can commit.

#### Running pre-commit hooks manually

you can also run `pre-commit` manually on all the files, you can do it using `uv` with:

```bash
❯ uv run pre-commit run --all-files
check for added large files..............................................Passed
check toml...............................................................Passed
check yaml...............................................................Passed
ruff.....................................................................Passed
ruff-format..............................................................Passed
eslint...................................................................Passed
prettier.................................................................Passed
```

## URLs

The production or staging URLs would use these same paths, but with your own domain.

### Development URLs

Development URLs, for local development.

Frontend: http://localhost:5173

Backend: http://localhost:8000

Automatic Interactive Docs (Swagger UI): http://localhost:8000/docs

Automatic Alternative Docs (ReDoc): http://localhost:8000/redoc

Adminer: http://localhost:8080

Traefik UI: http://localhost:8090

MailCatcher: http://localhost:1080

### Development URLs with `localhost.tiangolo.com` Configured

Development URLs, for local development.

Frontend: http://dashboard.localhost.tiangolo.com

Backend: http://api.localhost.tiangolo.com

Automatic Interactive Docs (Swagger UI): http://api.localhost.tiangolo.com/docs

Automatic Alternative Docs (ReDoc): http://api.localhost.tiangolo.com/redoc

Adminer: http://localhost.tiangolo.com:8080

Traefik UI: http://localhost.tiangolo.com:8090

MailCatcher: http://localhost.tiangolo.com:1080