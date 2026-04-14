FROM almalinux/10-base

# Set build arguments
ARG PIP_INDEX_URL
ARG PYTHON=python3.12
ARG NODE_VERSION=22

ENV PIP_INDEX_URL=${PIP_INDEX_URL}
ENV PYTHON=${PYTHON}
ENV NODE_VERSION=${NODE_VERSION}

# Base OS packages
RUN dnf -y upgrade && \
    dnf -y install epel-release && \
    dnf -y install \
        supervisor \
        git \
        gcc \
        gcc-c++ \
        zip \
        unzip \
        openssh-server \
        wget \
        tar \
        make \
        openssl-devel \
        bzip2-devel \
        libffi-devel \
        libxml2-devel \
        libxslt-devel \
        freetype-devel \
        socat \
        python3 \
        python3-pip \
        python3-devel && \
    dnf clean all && \
    rm -rf /var/cache/dnf

# Install Node.js
RUN curl -fsSL https://rpm.nodesource.com/setup_${NODE_VERSION}.x | bash - && \
    dnf -y install nodejs && \
    dnf clean all

# Install Homebrew
RUN useradd -m -s /bin/bash linuxbrew && \
    git clone https://github.com/Homebrew/brew /home/linuxbrew/.linuxbrew/Homebrew && \
    mkdir -p /home/linuxbrew/.linuxbrew/bin && \
    ln -s /home/linuxbrew/.linuxbrew/Homebrew/bin/brew /home/linuxbrew/.linuxbrew/bin/brew && \
    chown -R linuxbrew: /home/linuxbrew/.linuxbrew
ENV PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"

# Install pnpm, openclaw and codex globally
ENV PNPM_HOME="/root/.local/share/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN npm install -g pnpm@latest && \
    mkdir -p $PNPM_HOME && \
    pnpm add -g openclaw@latest && \
    pnpm add -g @openai/codex@latest && \
    pnpm add -g @anthropic-ai/claude-code@latest && \
    pnpm add -g opencode-ai@latest

# Install all OpenClaw bundled plugin runtime deps into openclaw's own node_modules
RUN OPENCLAW_DIR=$(pnpm root -g)/openclaw && \
    cd "$OPENCLAW_DIR" && \
    npm install --save --legacy-peer-deps \
        "@aws-sdk/client-bedrock@3.1024.0" \
        "@aws-sdk/client-s3@3.1024.0" \
        "@aws-sdk/s3-request-presigner@3.1024.0" \
        "@aws/bedrock-token-generator@^1.1.0" \
        "@buape/carbon@0.14.0" \
        "@clawdbot/lobster@2026.1.24" \
        "@discordjs/opus@^0.10.0" \
        "@discordjs/voice@^0.19.2" \
        "@google/genai@^1.48.0" \
        "@grammyjs/runner@^2.0.3" \
        "@grammyjs/transformer-throttler@^1.2.1" \
        "@lancedb/lancedb@^0.27.2" \
        "@larksuiteoapi/node-sdk@^1.60.0" \
        "@matrix-org/matrix-sdk-crypto-nodejs@^0.4.0" \
        "@matrix-org/matrix-sdk-crypto-wasm@18.0.0" \
        "@microsoft/teams.api@2.0.6" \
        "@microsoft/teams.apps@2.0.6" \
        "@opentelemetry/api@^1.9.1" \
        "@opentelemetry/api-logs@^0.214.0" \
        "@opentelemetry/exporter-logs-otlp-proto@^0.214.0" \
        "@opentelemetry/exporter-metrics-otlp-proto@^0.214.0" \
        "@opentelemetry/exporter-trace-otlp-proto@^0.214.0" \
        "@opentelemetry/resources@^2.6.1" \
        "@opentelemetry/sdk-logs@^0.214.0" \
        "@opentelemetry/sdk-metrics@^2.6.1" \
        "@opentelemetry/sdk-node@^0.214.0" \
        "@opentelemetry/sdk-trace-base@^2.6.1" \
        "@opentelemetry/semantic-conventions@^1.40.0" \
        "@pierre/diffs@1.1.10" \
        "@pierre/theme@0.0.29" \
        "@sinclair/typebox@0.34.49" \
        "@slack/bolt@^4.6.0" \
        "@slack/web-api@^7.15.0" \
        "@snazzah/davey@^0.1.11" \
        "@tloncorp/tlon-skill@0.3.2" \
        "@twurple/api@^8.1.3" \
        "@twurple/auth@^8.1.3" \
        "@twurple/chat@^8.1.3" \
        "@urbit/aura@^3.0.0" \
        "@whiskeysockets/baileys@7.0.0-rc.9" \
        "acpx@0.5.2" \
        "ajv@^8.18.0" \
        "commander@^14.0.3" \
        "discord-api-types@^0.38.44" \
        "express@^5.2.1" \
        "fake-indexeddb@^6.2.5" \
        "google-auth-library@^10.6.2" \
        "grammy@^1.42.0" \
        "https-proxy-agent@^9.0.0" \
        "jimp@^1.6.0" \
        "markdown-it@14.1.1" \
        "matrix-js-sdk@41.3.0-rc.0" \
        "mpg123-decoder@^1.0.3" \
        "music-metadata@^11.12.3" \
        "node-edge-tts@^1.2.10" \
        "nostr-tools@^2.23.3" \
        "openai@^6.33.0" \
        "opusscript@^0.1.1" \
        "playwright-core@1.59.1" \
        "silk-wasm@^3.7.1" \
        "undici@8.0.2" \
        "ws@^8.20.0" \
        "yaml@^2.8.3" \
        "zca-js@2.1.2" \
        "zod@^4.3.6"

# SSH keys
ADD .ssh /root/.ssh
RUN chmod 600 /root/.ssh/* 2>/dev/null || true && \
    ssh-keyscan github.com >> /root/.ssh/known_hosts

# Supervisor config for OpenClaw
ADD openclaw.ini /etc/supervisord.d/openclaw.ini
RUN chmod 755 /etc/supervisord.d/openclaw.ini

# Socat relay for OAuth ports (forwards from 0.0.0.0 to 127.0.0.1)
ADD socat-relay.ini /etc/supervisord.d/socat-relay.ini
RUN chmod 755 /etc/supervisord.d/socat-relay.ini

COPY requirements.txt /tmp/requirements.txt

RUN $PYTHON -m venv /var/$PYTHON/openclaw/env && \
    source /var/$PYTHON/openclaw/env/bin/activate && \
    pip install --upgrade pip -i "$PIP_INDEX_URL" && \
    pip install -r /tmp/requirements.txt -i "$PIP_INDEX_URL"

WORKDIR /var/www/projects
