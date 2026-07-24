## Introduction

Docker Compose deployment for OpenClaw. This project provides a containerized environment for running OpenClaw (autonomous AI assistant).

Based on the structure of [silvaengine_docker](../silvaengine/silvaengine_docker).

## Services

| Service | Description | Port |
|---------|-------------|------|
| openclaw | OpenClaw Gateway (Node.js + Python) | 18789 |

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
- Anthropic, OpenAI, Ollama Python SDKs
- Supervisor for process management

### Ollama Cloud

Local inference is not bundled. To use [Ollama Cloud](https://ollama.com), set the following in `.env`:

```
OLLAMA_HOST=https://ollama.com
OLLAMA_API_KEY=your_ollama_cloud_api_key
```

Both values are passed through to the `openclaw` container's environment.

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
