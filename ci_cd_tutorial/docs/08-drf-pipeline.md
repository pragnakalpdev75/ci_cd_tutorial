# Chapter 8: A Real DRF Pipeline

With the fundamentals in place, this chapter builds a real CI workflow for a Django REST Framework (DRF) project. A solid CI workflow for DRF should contain three things:

1. **Tests** — every project should have tests to make sure each piece of functionality works properly.
2. **Linting** — checking the structure of the code for formatting issues and unnecessary imports. This guide uses [Ruff](https://docs.astral.sh/ruff/) for that.
3. **Smoke Testing** — after tests and linting pass, verify the server actually runs correctly inside Docker by starting the container and hitting a `health/` endpoint.

The full pipeline looks like this:

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

## The base of the pipeline

Before adding linting or tests, it's worth verifying each stage individually, because debugging is much easier when you confirm each layer works before adding the next one:

```yaml
name: DRF CI Pipeline

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Verify Python
        run: python --version

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Verify Django
        run: python -m django --version
```

### Why the "verify" steps?

Instead of immediately jumping to `pytest`, this workflow deliberately checks `python -m django --version` first. If that command succeeds, you already know three things:

- ✅ Python is installed
- ✅ Requirements installed successfully
- ✅ Django is importable

If something breaks later, you can immediately rule these three things out.

## Job 1: Lint

**Purpose:** check the code for style and formatting issues without running the application.

```yaml
lint:
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@v4

    - uses: actions/setup-python@v5
      with:
        python-version: "3.11"
        cache: pip

    - run: pip install ruff

    - run: ruff check .
```

What happens, step by step:

1. Create a fresh Ubuntu VM.
2. Check out the repository so the runner has your code.
3. Install Python 3.11.
4. Restore the pip cache (if available) to speed up installation.
5. Install Ruff, the linter.
6. Run `ruff check .` to look for code quality issues.
7. Destroy the VM.

If Ruff finds any issues, the job fails.

## Job 2: Test (with coverage)

**Purpose:** verify the application works by running the full test suite against a real database.

```yaml
test:
  runs-on: ubuntu-latest

  services:
    postgres:
      image: postgres:16
      env:
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: postgres
        POSTGRES_DB: test_db
      ports:
        - 5432:5432

  env:
    SECRET_KEY: ${{ secrets.SECRET_KEY }}
    DEBUG: "False"
    # ... any other required env vars

  steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Setup Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.11"
        cache: pip

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run tests
      run: pytest --cov=apps --cov-report=html

    - name: Upload coverage report
      uses: actions/upload-artifact@v4
      with:
        name: coverage-report
        path: htmlcov/
```

What happens, step by step:

1. Create a fresh Ubuntu VM.
2. Start a PostgreSQL service container alongside the runner.
3. Check out the repository.
4. Install Python 3.11.
5. Restore the pip cache.
6. Install project dependencies from `requirements.txt`.
7. Run `pytest` to execute the test suite and generate a coverage report.
8. Upload the coverage report as an artifact (see Chapter 7) so it can be downloaded later.
9. Destroy the VM and the PostgreSQL container.

If any test fails, the job fails.

## Why separate jobs?

Each job has a single responsibility:

- `lint` → *"Is the code clean?"*
- `test` → *"Does the code work?"*

Because they're separate, GitHub can:

- Run them **in parallel**, saving time (see Chapter 5).
- Show exactly which stage failed, without digging through one giant log.
- Allow later jobs — like a Docker build or a deploy step — to depend on **both** succeeding via `needs: [lint, test]`.

This separation is a common CI/CD convention because it keeps workflows modular and easier to maintain as the project grows. The next chapter picks up from here and adds Docker into the pipeline.
