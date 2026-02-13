# FaaS Forgejo Runner

Forgejo Actions runner for deploying FaaS function containers. Auto-registers on first startup — just set the environment variables and deploy.

## Quick Start

```bash
# Copy environment template and fill in your values
cp .env.example .env

# Start the runner
docker compose up -d

# Check it's running
docker logs forgejo-runner
```

## Setup Steps

1. **Enable Actions in Forgejo** — Site Administration → Actions → Enable
   (or add `FORGEJO__actions__ENABLED=true` to your Forgejo service env vars)

2. **Get a registration token** — Site Administration → Actions → Runners → Create new runner

3. **Configure environment** — Copy `.env.example` to `.env` and set:

| Variable | Required | Description |
|----------|----------|-------------|
| `FORGEJO_INSTANCE` | ✅ | Your Forgejo URL (e.g. `https://git.example.com`) |
| `FORGEJO_RUNNER_TOKEN` | ✅ | Registration token from step 2 |
| `FORGEJO_RUNNER_NAME` | | Runner name (default: `faas-runner`) |
| `FORGEJO_RUNNER_LABELS` | | Runner labels (default: `ubuntu-latest:host`) |

4. **Start** — `docker compose up -d`

5. **Verify** — Check Site Administration → Actions → Runners — the runner should appear as online

## How It Works

On first startup, the runner checks if it's already registered (`.runner` file in the data volume). If not, it registers with Forgejo using the environment variables. On subsequent restarts it skips registration and goes straight to the daemon.

The runner mounts the host Docker socket (`/var/run/docker.sock`), so workflow steps that run `docker build` and `docker run` operate directly on the host Docker — the same Docker where your gateway and function containers live.

## Notes

- The runner label `ubuntu-latest` is used in workflow files (`runs-on: ubuntu-latest`)
- Function containers are deployed to the `faas-network` Docker network
- Registration persists in the `runner-data` volume — survives restarts and redeployments
- The healthcheck monitors the runner daemon process