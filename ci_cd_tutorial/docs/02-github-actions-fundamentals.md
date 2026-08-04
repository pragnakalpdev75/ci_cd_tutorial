# Chapter 2: GitHub Actions Fundamentals

## What is GitHub Actions?

GitHub Actions is GitHub's built-in automation platform. It listens for an **event** (like a push to your repository) and performs one or more **jobs** automatically.

```text
You write code
   |
   ▼
git push origin main
   |
   ▼
GitHub receives the push
   |
   ▼
Workflow starts
   |
   ▼
Run tests
   |
   ▼
Build application
   |
   ▼
Deploy (optional)
```

## The core building blocks

Everything in GitHub Actions is built from six concepts:

```text
Workflow
  ├── Event (Trigger)
  ├── Job
  ├── Step
  └── Action
```

(Runner is the fifth concept — it's *where* a job actually executes.)

### 1. Workflow

A **workflow** is the entire automation process. It's simply a YAML file inside `.github/workflows/`:

```text
# Example
.github/
    workflows/
        ci.yml
```

### 2. Event (Trigger)

A workflow needs something to start it. That's called an **event**.

```yaml
on:
  push:
```

Meaning:

```text
Developer pushes code
        |
        ▼
   Workflow starts
```

Other common triggers:

```yaml
# On pull request
on:
  pull_request:

# Manual trigger
on:
  workflow_dispatch:

# Runs every day at midnight
on:
  schedule:
    - cron: '0 0 * * *'
```

You can also combine them:

```yaml
on:
  push:
  pull_request:
  workflow_dispatch:
```

`workflow_dispatch` is worth calling out specifically — it adds a **"Run workflow"** button directly in the GitHub UI, letting you trigger the pipeline manually.

### 3. Job

A workflow contains one or more **jobs**. Think of a job as one complete task.

```yaml
jobs:
  test:
```

```text
Workflow
   |
   ├── Test Job
   ├── Build Job
   └── Deploy Job
```

Each job gets its own fresh machine.

### 4. Runner

Where does a job actually run? On a **runner**.

```yaml
runs-on: ubuntu-latest
```

```text
GitHub
  |
  ▼
Create Ubuntu VM
  |
  ▼
Run commands
  |
  ▼
Destroy VM
```

Every workflow starts from scratch. Nothing is saved between runs unless you explicitly cache dependencies or upload artifacts (covered in Chapter 7).

### 5. Steps

A job is divided into **steps**, executed in order.

```yaml
steps:
  - name: Print Hello
    run: echo "Hello"
```

### 6. Action

An **Action** is reusable code written by someone else. Instead of writing Git commands manually to clone a repo, you can write:

```yaml
# You can use this to clone the code:
- uses: actions/checkout@v4

# Another example — installs Python
- uses: actions/setup-python@v5
```

This is the same idea as `import requests` in Python — you didn't write the library, someone else did, and you're reusing it.

## Putting it all together

Here's a complete, minimal workflow using every concept above:

```yaml
name: My First Workflow

on:
  push:

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Print message
        run: echo "Hello GitHub Actions!"
```

Tracing what happens when you push:

```text
git push
   |
   ▼
GitHub receives push
   |
   ▼
Reads .github/workflows/*.yml
   |
   ▼
Starts Workflow
   |
   ▼
Creates Ubuntu Runner
   |
   ▼
Runs Job "hello"
   |
   ▼
Step 1: Checkout repository
   |
   ▼
Step 2: echo "Hello GitHub Actions!"
   |
   ▼
Workflow finished
```
