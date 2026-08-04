# Chapter 3: Creating Your First Workflow

Now that you know the vocabulary, let's actually create one.

## Step 1: Create the workflow directory

```text
.github/
└── workflows/
    └── hello.yml
```

## Step 2: Add a basic test workflow

```yaml
name: My First Workflow

on:
  push:
  workflow_dispatch:

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Show current directory
        run: pwd

      - name: List repository files
        run: ls -la

      - name: Print greeting
        run: echo "Hello from GitHub Actions!"

      - name: Show Python version
        run: python --version
```

## Step 3: Commit and push

```bash
git add .
git commit -m "Add GitHub Actions test workflow"
git push origin main
```

## Step 4: View the workflow

Go to your repository on GitHub — you'll see an **Actions** tab.

```text
Repository
  |
  ├── Code
  ├── Issues
  ├── Pull requests
  ├── Actions   ← Click here
  └── Settings
```

Open the run and check the output of each step:

- What did `pwd` print?
- What did `ls -la` show?
- Did you see your repository files?

If the answer to all three is "yes," the checkout step worked correctly and the runner had a real copy of your repository to work with. This is a good sanity check to run any time a new workflow behaves unexpectedly — confirm the basics (directory, files, tool versions) before debugging anything more complex.
