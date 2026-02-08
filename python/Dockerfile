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
    pnpm add -g @anthropic-ai/claude-code@latest

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
