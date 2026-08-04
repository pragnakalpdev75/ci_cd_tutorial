---
sidebar_position: 0
title: "Introduction"
slug: /
---

# CI/CD with GitHub Actions

Welcome to this hands-on guide to **CI/CD with GitHub Actions**.

The goal of this documentation is to help you understand not only **how** to build CI/CD pipelines, but also **why** each component exists and how everything fits together in a real-world deployment workflow.

Instead of jumping directly into writing YAML files, we'll build our knowledge step by step—from the fundamentals of CI/CD to deploying a production-ready application.

---

## What you'll build

By the end of this guide, you'll have a complete GitHub Actions pipeline that can:

- Automatically run whenever code is pushed to GitHub
- Lint and test a Django REST Framework application
- Build the application
- Package it using Docker
- Deploy it to an AWS EC2 instance

Along the way, you'll learn how GitHub Actions actually works under the hood, making it easier to understand existing pipelines and create your own.

---

## Prerequisites

This guide assumes you're familiar with:

- Basic Git and GitHub
- Python
- Django (or Django REST Framework)
- Basic command-line usage
- Docker fundamentals (helpful but not required)

No previous GitHub Actions experience is required.

---

## Course Structure

The chapters are designed to be completed in order.

1. Introduction to CI/CD
2. GitHub Actions Fundamentals
3. Creating Your First Workflow
4. `uses:` vs `run:`
5. Jobs and Dependencies
6. Environment Variables and Secrets
7. Artifacts
8. Building a Real Django Pipeline
9. Dockerizing the Application
10. Deploying to AWS EC2

Each chapter builds on the previous one, so it's recommended not to skip ahead.

---

## Repository Structure

Throughout this guide, you'll create GitHub workflow files inside:

```text
.github/
└── workflows/
    └── ci.yml
```

As the guide progresses, you'll gradually expand these workflows into a complete CI/CD pipeline.

---

Let's begin by understanding what **CI/CD** actually means and why modern software teams rely on it.