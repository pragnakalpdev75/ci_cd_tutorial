# Chapter 9: Containerizing the Application

With linting and testing in place, the next step is to package the application so it can run identically anywhere — locally, in CI, or on a production server. That's what Docker gives you.

We'll create three files:

1. `Dockerfile`
2. `.dockerignore`
3. `docker-compose.yml`

## Why containerize?

Your local machine already has Python, pip, and various system libraries installed — which is exactly the problem. A Docker image packages all of that together into one portable unit, so anyone who has Docker can run that exact environment, regardless of what's installed on their own machine.

## Step 1: Create a `.dockerignore`

Just like `.gitignore`, this keeps unnecessary or sensitive files out of the image:

```text
.git
.github
.venv
venv
__pycache__
*.pyc
.pytest_cache
.ruff_cache
.coverage
htmlcov
.env
.env.*
media
staticfiles
```

## Step 2: Create a `Dockerfile`

Create a file named exactly `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### What each line does

- **`FROM python:3.11-slim`** — downloads an official, lightweight Python image to build on top of.
- **`WORKDIR /app`** — everything happens inside `/app` instead of the container's root directory.
- **`COPY requirements.txt .`** — copies *only* the requirements file first. This is a deliberate optimization: Docker caches each layer, so if your dependencies haven't changed, this layer is reused and the (slow) install step is skipped on the next build.
- **`RUN pip install ...`** — installs your dependencies.
- **`COPY . .`** — copies the rest of your project in.
- **`EXPOSE 8000`** — documents that the application listens on port 8000.
- **`CMD ...`** — runs Django when the container starts.

## Step 3: Build and run the image locally

```bash
docker build -t ci-cd-testing .
```

## Step 4: Create a Docker Compose file

The Compose file describes every container your app needs to run — the app itself, PostgreSQL, Redis, etc. — and how they connect.

```yaml
services:
  web:
    image: <your-username>/<image>:latest

    ports:
      - "8000:8000"

    env_file:
      - .env

    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:17

    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASS}

    volumes:
      - postgres17_data:/var/lib/postgresql/data

  redis:
    image: redis:7

volumes:
  postgres17_data:
```

## Step 5: Test the Compose file locally

```bash
docker compose up --build
```

Confirm the app is reachable at `http://localhost:8000` before wiring any of this into CI.

## Step 6: Push the Docker image to Docker Hub

Once you're confident the image works, add a job that builds it and pushes it to a registry (Docker Hub, in this case). This way, the production server never needs a copy of your source code — it only needs the Docker Compose file and permission to pull the image.

1. Add credentials to the repository's GitHub secrets:
   - `DOCKERHUB_USERNAME`
   - `DOCKERHUB_TOKEN`

2. Add the job:

```yaml
docker:
  runs-on: ubuntu-latest

  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Create .env
      run: |
        echo "${{ secrets.DOTENV }}" > .env

    - name: Log in to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}

    - name: Build Docker image
      uses: docker/build-push-action@v6
      with:
        context: .
        file: ./Dockerfile
        push: true

        tags: |
          ${{ secrets.DOCKERHUB_USERNAME }}/django-web:${{ github.sha }}

        cache-from: type=gha
        cache-to: type=gha,mode=max
```

Tagging the image with `${{ github.sha }}` (the commit hash) means every build produces a uniquely identifiable image — useful for rolling back to a specific version later.

## Step 7: Smoke test the container

A smoke test spins the container up for real and checks that it actually responds, rather than just trusting that the build succeeded.

```yaml
smoke-test:
  needs: docker

  runs-on: ubuntu-latest

  steps:
    - name: Checkout Repository
      uses: actions/checkout@v4

    - name: Create .env
      run: |
        echo "${{ secrets.DOTENV }}" > .env   # creates env from the github repo secrets

    - name: Compose up
      run: docker compose up -d --build

    - name: Wait 30 seconds
      run: sleep 30

    - name: Verify Live Health Endpoint
      run: |
        # Fetch response
        RESPONSE=$(curl -s localhost:8000/health/)

        # Extract the status key
        STATUS=$(echo "$RESPONSE" | jq -r '.status')

        # Validate structure and value
        if [ "$STATUS" != "ok" ]; then
          echo "Smoke test failed! Expected {'status': 'ok'}, got: $RESPONSE"
          exit 1
        fi
        echo "Smoke test passed successfully."

    - name: Show Compose logs
      if: failure()
      run: docker compose logs

    - name: Show container status
      if: failure()
      run: docker compose ps

    - name: Compose down
      run: docker compose down
```

Two details worth calling out:

- **`sleep 30`** gives the container time to fully start before the health check hits it — without this, the request might arrive before the server is ready and produce a false failure.
- **`if: failure()`** on the logging steps means they only run when something above them failed, so successful runs stay quiet and failed runs get full diagnostic output automatically.

## The pipeline so far

```text
Push Code
   |
   ▼
Checkout
   |
   ▼
Setup Python
   |
   ▼
Install Dependencies
   |
   ▼
Run Ruff (Lint)
   |
   ▼
Run Django Tests
   |
   ▼
Build and run docker
   |
   ▼
Test with smoke tests
```

At this point the pipeline lints, tests, builds an image, pushes it to a registry, and confirms the container actually boots and responds correctly. The only piece missing is getting that image onto a real server — which is the subject of the final chapter.
