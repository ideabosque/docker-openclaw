## Introduction

Docker Compose deployment for OpenClaw with Ollama integration. This project provides a containerized environment for running OpenClaw (autonomous AI assistant) alongside Ollama for local LLM inference.

Based on the structure of [silvaengine_docker](../silvaengine/silvaengine_docker) and incorporating patterns from [mythrantic/ollama-docker](https://github.com/mythrantic/ollama-docker).

## Services

| Service | Description | Port |
|---------|-------------|------|
| openclaw | OpenClaw Gateway (Node.js + Python) | 18789 |
| ollama | Ollama LLM Server | 11434 |
| ollama-webui | Open WebUI Interface | 8080 |

## Installation

1. Clone this repository and navigate to the directory:

    ```bash
    cd docker-openclaw
    ```

2. Create the required directories:

    ```bash
    mkdir www/logs
    mkdir www/projects
    ```

3. (Optional) Place SSH keys in `python/.ssh/` for GitHub access.

4. Create a `.env` file from the example:

    ```bash
    cp .env.example .env
    ```

    Configure your environment variables:

    ```
    PIP_INDEX_URL=https://pypi.org/simple/
    PYTHON=python3.12
    NODE_VERSION=22
    PROJECTS_FOLDER={path to the projects folder}
    ANTHROPIC_API_KEY=your_anthropic_api_key
    OPENAI_API_KEY=your_openai_api_key
    ```

    **Example:**

    - PIP_INDEX_URL: https://pypi.org/simple/
    - PROJECTS_FOLDER: "C:/Users/developer/GitHubRepos/openclaw_projects"
    - PYTHON: python3.12

5. Build the Docker images:

    ```bash
    docker compose build
    ```

6. Start the containers:

    ```bash
    docker compose up -d
    ```

## Configure requirements.txt

Configure the `python/requirements.txt` file to manage the necessary packages. Ensure this file includes all dependencies required for your OpenClaw projects to run smoothly.

## Usage

### OpenClaw Gateway

The OpenClaw container includes:
- Node.js 22 with OpenClaw installed globally via pnpm
- Python 3.12 with virtual environment at `/var/python3.12/openclaw/env`
- Ollama, Anthropic, OpenAI Python SDKs
- Supervisor for process management

### Ollama

Pull models via the API or Open WebUI:

```bash
# Pull a model
curl http://localhost:11434/api/pull -d '{"name": "llama3.2"}'

# List models
curl http://localhost:11434/api/tags
```

### Open WebUI

Access the web interface at `http://localhost:8080` to interact with Ollama models.

## GPU Support

The `ollama` service is configured for NVIDIA GPU support. Ensure you have:
- NVIDIA drivers installed
- NVIDIA Container Toolkit installed

For CPU-only mode, remove the `deploy` section from the ollama service in `docker-compose.yaml`.

## Directory Structure

```
docker-openclaw/
├── docker-compose.yaml
├── .env.example
├── .gitignore
├── README.md
├── python/
│   ├── Dockerfile
│   ├── openclaw.ini
│   ├── requirements.txt
│   └── .ssh/
└── www/
    ├── logs/
    └── projects/
```

## License

MIT
