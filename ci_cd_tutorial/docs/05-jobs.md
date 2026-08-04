# Chapter 5: Jobs, Isolation, and Dependencies

## A job is one complete machine

Until now, workflows in this guide have had a single job:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - ...
      - ...
      - ...
```

Think of a job as:

> **One complete machine dedicated to performing one task.**

Many beginners assume jobs are like running commands in the same terminal session. **They aren't.** Each job gets a completely new, isolated runner.

## Proof: jobs don't share files

Create `.github/workflows/05-jobs.yml`:

```yaml
name: Understanding Jobs

on:
  workflow_dispatch:

jobs:
  first-job:
    runs-on: ubuntu-latest

    steps:
      - name: Show hostname
        run: hostname

      - name: Create a file
        run: |
          echo "Hello from Job 1" > hello.txt

      - name: Show files
        run: ls -la

  second-job:
    runs-on: ubuntu-latest

    steps:
      - name: Show hostname
        run: hostname

      - name: Show files
        run: ls -la

      - name: Try to read file
        run: cat hello.txt
```

Run it, and you'll see `second-job` **fails** to find `hello.txt`. This happens because both jobs run on separate virtual machines — they don't share a filesystem, and `first-job`'s hostname will be different from `second-job`'s.

## How do jobs share data, then?

If jobs are isolated, how can a deployment job use something a build job produced? GitHub provides **artifacts** — think of them like a courier service between machines.

```text
Job 1
  ├── Build application
  ├── Create build.zip
  └── Upload artifact
        |
        ▼
   GitHub stores it
        |
        ▼
Job 2 downloads artifact
        |
        ▼
   Deploy build.zip
```

Artifacts are covered in full in Chapter 7 — for now, just remember: **if two jobs need to share a file, an artifact is the mechanism.**

## Jobs run in parallel by default

```yaml
jobs:
  first-job:
    ...
  second-job:
    ...
```

Because there's no dependency declared between them, GitHub runs both jobs **simultaneously**. You can see this in the Actions UI — both jobs typically start around the same time.

This has a real performance benefit. If:

- Linting takes 30 seconds
- Tests take 120 seconds

...and they run in parallel, total workflow time is roughly **120 seconds** instead of **150 seconds**, because the two jobs overlap.

```text
Workflow Starts
        |
   ┌────┴────┐
   ▼         ▼
Lint Job   Test Job
   |         |
   ▼         ▼
Success    Success
   └────┬────┘
        ▼
Workflow Complete
```

An important consequence of this: **if `lint` fails, GitHub still runs `test`.** Sometimes that's exactly what you want — you get both results in one run instead of stopping at the first failure.

## Making one job depend on another: `needs:`

Sometimes independence is *not* what you want. Consider deployment:

```text
Push
  |
  ▼
Lint
  |
  ▼
Tests
  |
  ▼
Deploy to Production
```

Should GitHub deploy if the tests fail? **Absolutely not.** Deployment should only happen if:

- ✅ Linting passes
- ✅ Tests pass

This is exactly what `needs:` is for:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

Now the execution order becomes strictly sequential for the dependent job:

```text
Build Job
   |
   ▼
Finished Successfully
   |
   ▼
Deploy Job Starts
```

If `build` fails, `deploy` is automatically skipped — GitHub won't waste time (or risk shipping broken code) by running a job whose dependency failed.

`needs:` also accepts a list, so a job can wait on more than one predecessor:

```yaml
deploy:
  needs: [lint, test]
```

This is the pattern used later in this guide: **lint** and **test** run in parallel to save time, and **deploy** only fires once both have succeeded.
