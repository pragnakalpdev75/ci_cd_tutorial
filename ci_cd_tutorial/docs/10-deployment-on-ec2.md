# Chapter 10: Deployment to an EC2 Instance

With a tested, containerized image pushed to Docker Hub, the last step is deploying it to a live server. This chapter walks through connecting GitHub Actions to an AWS EC2 instance over SSH and triggering a deploy automatically once every earlier stage has passed.

## Step 1: Create a dedicated deploy key

On **your local machine**, generate a new SSH key pair specifically for CI — don't reuse your personal key:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/github_actions_ec2 -C "github-actions-deploy"
```

You'll be prompted:

```text
Enter passphrase (empty for no passphrase):
```

For CI, **press Enter twice** (no passphrase). GitHub Actions can't type a passphrase interactively, so a protected key would cause the deploy step to hang or fail.

## Step 2: Add the public key to EC2

Display the public key:

```bash
cat ~/.ssh/github_actions_ec2.pub
```

SSH into your EC2 instance using your normal, existing key:

```bash
ssh -i ~/.ssh/your_current_key ubuntu@YOUR_EC2_IP
```

Once connected, append the new public key to the server's authorized keys:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh

echo "PASTE_THE_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
```

## Step 3: Verify it works

Before involving GitHub at all, confirm the new key logs in correctly on its own:

```bash
ssh -i ~/.ssh/github_actions_ec2 ubuntu@YOUR_EC2_IP
```

If this fails, nothing in CI will work either — fix it here first.

## Step 4: Add GitHub secrets

Add the following to the repository's secrets (Settings → Secrets and variables → Actions, as covered in Chapter 6):

| Secret | Value |
|---|---|
| `EC2_HOST` | Your EC2 public IP or DNS |
| `EC2_USER` | `ubuntu` (or your EC2 username) |
| `EC2_SSH_KEY` | Contents of `~/.ssh/github_actions_ec2` (the **private** key) |

Note that it's the private key that goes into GitHub Secrets — never the public one. This is the credential that lets GitHub Actions authenticate as you, so it must stay protected exactly like any other secret.

## Step 5: The deployment job

```yaml
deploy:
  needs: smoke-test

  runs-on: ubuntu-latest

  steps:
    - name: Deploy
      uses: appleboy/ssh-action@v1
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ${{ secrets.EC2_USER }}
        key: ${{ secrets.EC2_SSH_KEY }}

        script: |
          cd ~/production
          docker compose pull
          docker compose up -d
          docker image prune -f
```

What this does on the server, step by step:

1. `cd ~/production` — move into the directory that holds the server's `docker-compose.yml`.
2. `docker compose pull` — fetch the newest image that was just pushed to Docker Hub (tagged with the commit SHA, or `latest`, depending on how the build job tagged it).
3. `docker compose up -d` — restart the containers using the new image, detached.
4. `docker image prune -f` — clean up old, now-unused images so the server's disk doesn't fill up over time.

## Why `needs: smoke-test`?

This is the payoff of everything built in the earlier chapters. Because `deploy` declares `needs: smoke-test`, and `smoke-test` in turn declared `needs: docker`, GitHub won't run this job unless the entire chain has already succeeded:

```text
lint ─┐
      ├─→ (both must pass)
test ─┘
      │
      ▼
    docker (build & push image)
      │
      ▼
  smoke-test (container actually boots and responds)
      │
      ▼
     deploy (ships to production)
```

If linting fails, or a test fails, or the container doesn't boot correctly, **deployment never happens** — broken code simply can't reach production through this pipeline.

## The complete flow, end to end

```text
Developer pushes code
        |
        ▼
GitHub Actions triggers
        |
        ├── Lint (Ruff)
        ├── Test (pytest + coverage, against Postgres)
        │        |
        │        ▼
        ├── Build & push Docker image
        │        |
        │        ▼
        ├── Smoke test the container
        │        |
        │        ▼
        └── Deploy to EC2 over SSH
                 |
                 ▼
        docker compose pull && up -d
                 |
                 ▼
         Live on production
```

That's a complete CI/CD pipeline: every push is automatically linted, tested, containerized, verified, and — if everything passes — deployed to a live server with no manual steps in between.
