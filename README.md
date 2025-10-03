# vault-github-actions-proxy
Repro for proxy issues with Github Actions and Vault

## 1) Create a proxy environment
1. Install & run mitmproxy on host (observes proxy traffic):
```bash
brew install mitmproxy    # if needed
mitmproxy --listen-port 8888
```

## 2) Verify curl goes through proxy
```bash
export HTTP_PROXY=http://127.0.0.1:8888
export HTTPS_PROXY=http://127.0.0.1:8888
curl -v --proxy http://127.0.0.1:8888 http://127.0.0.1:8200/v1/sys/health
```

## 3) Start a Docker runner
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
