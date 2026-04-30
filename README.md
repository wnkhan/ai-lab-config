# AI Lab Docker

Local Docker Compose stack for running Open WebUI with a local Ollama backend and a local SearXNG search service.

This setup is intended for personal development and local lab use. It is not hardened as a public internet service.

## Services

- Open WebUI: browser UI for chatting with local models.
- Ollama: local model runtime and model storage.
- SearXNG: local metasearch service that can be connected to Open WebUI for web search.

By default, the stack uses Docker named volumes for application data:

- `ollama-data`
- `open-webui-data`

Those volumes are managed by Docker and are not stored in this repository.

## Prerequisites

Install Docker and Docker Compose before using this repository.

### Windows

1. Install Docker Desktop for Windows.
2. Enable the WSL 2 backend when prompted.
3. If you want GPU acceleration, install a current NVIDIA driver with WSL support.
4. Run commands from PowerShell, Windows Terminal, or a WSL shell.

### macOS

1. Install Docker Desktop for Mac.
2. Start Docker Desktop before running Compose commands.
3. Apple Silicon and Intel Macs can run this stack, but the NVIDIA GPU reservation in `compose.yaml` is only useful on NVIDIA Linux hosts. For CPU-only or non-NVIDIA systems, remove or override the `deploy.resources.reservations.devices` block under the `ollama` service.

### Linux

1. Install Docker Engine.
2. Install the Docker Compose plugin.
3. Add your user to the `docker` group if you do not want to run Docker with `sudo`.
4. For NVIDIA GPU acceleration, install the NVIDIA driver and NVIDIA Container Toolkit.
5. For CPU-only systems, remove or override the `deploy.resources.reservations.devices` block under the `ollama` service.

## Setup

Create a local environment file:

```sh
cp .env.example .env
```

Edit `.env` and replace both secret placeholders with random values. On Linux or macOS:

```sh
openssl rand -hex 32
```

On Windows PowerShell:

```powershell
[guid]::NewGuid().ToString("N") + [guid]::NewGuid().ToString("N")
```

Do not commit `.env`. It is intentionally ignored by Git.

## Start The Stack

From this repository:

```sh
docker compose up -d
```

Open WebUI:

```text
http://localhost:3001
```

SearXNG:

```text
http://localhost:8080
```

Ollama is exposed on the host at:

```text
http://localhost:11435
```

Inside the Compose network, Open WebUI talks to Ollama at `http://ollama:11434`.

## Pull A Model

After the stack starts, pull a model into the Ollama container:

```sh
docker exec -it ai-lab-ollama ollama pull llama3.2
```

Then select the model in Open WebUI.

You can list installed models with:

```sh
docker exec -it ai-lab-ollama ollama list
```

## Open WebUI First Run

The first account created in Open WebUI becomes the initial admin account. Use a local account and keep the stack bound to trusted networks only.

## SearXNG

SearXNG configuration lives in:

```text
config/searxng/settings.yml
```

The SearXNG secret is supplied through the `SEARXNG_SECRET` variable in your local `.env` file. Keep real secrets out of `settings.yml` so the repository can remain public.

## Stop Or Update

Stop containers:

```sh
docker compose down
```

Pull newer images and recreate containers:

```sh
docker compose pull
docker compose up -d
```

Remove containers and the Docker network while keeping model and app data:

```sh
docker compose down
```

Remove containers, network, and named volumes:

```sh
docker compose down -v
```

`docker compose down -v` deletes downloaded models and Open WebUI data for this stack.

## Security Notes

- Do not commit `.env`.
- Use fresh random values for `WEBUI_SECRET_KEY` and `SEARXNG_SECRET`.
- This stack is designed for local use, not direct public exposure.
- Review port bindings before running on a shared machine or server.
- The `latest` image tags can change over time. Pin exact image versions if you need reproducible deployments.
