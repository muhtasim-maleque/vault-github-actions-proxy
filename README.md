# vault-github-actions-proxy
Repro for proxy issues with Github Actions and Vault

## 0) Start a local Vault server

## 1) Create a proxy environment
1. Install & run mitmproxy on host (observes proxy traffic):
```bash
brew install mitmproxy    # if needed
mitmproxy --listen-port 8888
```
Keep this terminal open to show requests.

## 2) Verify curl goes through proxy
```bash
export HTTP_PROXY=http://127.0.0.1:8888
export HTTPS_PROXY=http://127.0.0.1:8888
curl -v --proxy http://127.0.0.1:8888 http://127.0.0.1:8200/v1/sys/health
```

## 3a) Start a Docker-based self-hosted GitHub runner
```bash
docker pull ghcr.io/actions/actions-runner:latest
# start interactive container (will drop you into container shell)
docker run -it --name vault-runner-proxy \
  -e RUNNER_ALLOW_RUNASROOT=1 \
  ghcr.io/actions/actions-runner:latest
```

Inside the container run the config (use a fresh token from your Github Repo Settings → Actions → Runners → New self-hosted runner):
```bash
./config.sh --url https://github.com/<owner>/<repo> --token <TOKEN> --unattended --name vault-runner-proxy
./run.sh
```

## 3b) Add repository secrets (Vault AppRole + address)

Before running the workflows, add the Vault connection and AppRole credentials as **Actions secrets**:

**Repo UI:** `Settings` → `Secrets and variables` → `Actions` → `New repository secret`

- `VAULT_ADDR` = `<your VAULT address>`  
- `VAULT_ROLE_ID` = `<your Approle role_id>`  
- `VAULT_SECRET_ID` = `<your Approle secret_id>`

## 4a) Run Basic Approle workflow (just for sanity check, skip step 2 if doing this)
```text
Actions → basic-approle-test → Run workflow
```

## 4b) Run proxy workflow
```text
Actions > test-vault-proxy > Run workflow
```
