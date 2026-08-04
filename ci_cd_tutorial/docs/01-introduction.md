# Chapter 1: Introduction to CI/CD

Before writing any workflows, it's worth understanding *why* CI/CD exists and what each term actually means.

## The three terms

- **Continuous Integration (CI)** — Automatically build and test your code whenever someone pushes changes.
- **Continuous Delivery (CD)** — Automatically prepare your application for deployment after CI passes.
- **Continuous Deployment** — Automatically deploy to production without manual approval.

The difference between the last two is subtle but important: **Delivery** means the code is *ready* to ship (a human still clicks "deploy"), while **Deployment** means it ships *automatically* the moment it passes checks.

## A typical pipeline

```text
Developer
   |
   git push
   |
   ▼
GitHub Repository
   |
GitHub Actions
   |
   ├── Checkout code
   ├── Install dependencies
   ├── Run tests
   ├── Build application
   └── Deploy
   |
   ▼
Server (AWS EC2)
```

Every push triggers this chain automatically — no one has to remember to run tests or manually copy files to a server.

## Where a workflow lives

GitHub Actions workflows are just YAML files inside a specific folder in your repository:

```text
.github/
  workflows/
    ci.yml
```

Anything placed in `.github/workflows/` is automatically picked up by GitHub.

## What this guide covers

This documentation walks through GitHub Actions from the ground up, in the following order:

1. **GitHub Actions Fundamentals** — the core building blocks (workflow, event, job, runner, step, action).
2. **Creating Your First Workflow** — a hands-on "Hello World" pipeline.
3. **`uses:` vs `run:`** — the two ways a step can do work.
4. **Jobs** — how jobs run in isolation, in parallel, and how to chain them with `needs:`.
5. **Environment Variables and Secrets** — passing configuration and credentials safely.
6. **Artifacts** — sharing files between jobs and preserving build output.
7. **A Real DRF Pipeline** — linting, testing, and smoke-testing a Django REST Framework project.
8. **Containerizing the Application** — packaging the app with Docker and Docker Compose.
9. **Deployment to EC2** — shipping the finished image to a live server over SSH.

By the end, you'll have a complete pipeline that lints, tests, builds, containerizes, and deploys a project automatically on every push to `main`.
