# Chapter 6: Environment Variables and Secrets

## The problem: your `.env` file

Your local `.env` file probably looks something like this:

```text
SECRET_KEY=django-insecure-...
DEBUG=True
DATABASE_URL=postgres://...
EMAIL_HOST_PASSWORD=...
```

## Why not just write these values in the code?

Suppose you do this instead:

```python
SECRET_KEY = "my-secret-key"
```

It works locally. Then you push your code to GitHub. Now everyone with access to the repository can see your secret — and if it's a public repository, the **whole world** can see it.

## Environment variables: the standard fix

Instead of hardcoding values, the operating system stores them separately from your code.

On Linux:

```bash
export DEBUG=True
export NAME=YourProjectName
```

Your program then reads them at runtime:

```python
import os

print(os.getenv("NAME"))
```

## How GitHub Actions provides environment variables

GitHub can inject variables into your workflow using `env:`.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    env:
      DEBUG: "True"

    steps:
      - run: echo $DEBUG
        # Output will be "True"
```

## Scope matters

GitHub Actions supports `env:` at three levels, and the scope determines where a variable is visible.

### Workflow level — available everywhere

```yaml
env:
  APP_NAME: DRF API

jobs:
  ...
```

### Job level — available only inside one job

```yaml
jobs:
  test:
    env:
      DEBUG: True
```

### Step level — available only in one step

```yaml
steps:
  - env:
      NAME: Mann
```

Visualized as a tree, with variables cascading down:

```text
Workflow
  ├── APP_NAME               (visible to every job)
  ├── Job 1
  │     ├── DEBUG             (visible only inside Job 1)
  │     ├── Step 1
  │     │     └── TEMP        (visible only inside Step 1)
  │     └── Step 2
  └── Job 2
```

A variable declared at the workflow level is visible to every job and step below it. A variable declared at the job level is visible only within that job. A variable declared at the step level is visible only within that single step.

## GitHub Secrets

Plain `env:` values are fine for non-sensitive configuration (like `DEBUG` or an app name), but they are **not** appropriate for credentials — anything written directly into a YAML file is visible to anyone who can read the repository.

GitHub provides a secure storage area for this called **Secrets**. You add them once, in the repository settings:

```text
Repository
   |
   ▼
Settings
   |
   ▼
Secrets and variables
   |
   ▼
Actions
```

Typical secrets to create:

```text
SECRET_KEY
DATABASE_PASSWORD
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

Using a secret inside a workflow:

```yaml
env:
  SECRET_KEY: ${{ secrets.SECRET_KEY }}
```

GitHub automatically masks secret values in log output, so even if a step accidentally prints one, it won't appear in plain text in the Actions UI.

## Environment secrets

Beyond repository-wide secrets, GitHub also supports **environment secrets** — values tied to a specific deployment environment, such as `Development`, `Staging`, or `Production`. This lets you use a different `DATABASE_PASSWORD`, for example, depending on which environment a job is deploying to, without changing your workflow file at all.
