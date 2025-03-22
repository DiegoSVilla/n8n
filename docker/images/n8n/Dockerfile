# Use a Node.js base image (no reference to n8nio/base, you control the version)
FROM node:20-slim

# Set build arguments
ARG N8N_VERSION=latest  
# Default to latest, can be changed to a specific version/branch

# Labels for image metadata
LABEL org.opencontainers.image.title="n8n"
LABEL org.opencontainers.image.description="Workflow Automation Tool"
LABEL org.opencontainers.image.source="https://github.com/DiegoSVilla/n8n"  
# Reference to your fork
LABEL org.opencontainers.image.url="https://n8n.io"
LABEL org.opencontainers.image.version=${N8N_VERSION}

# Set environment variables
ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable

# Install required system dependencies (you might need more depending on your environment)
RUN apt-get update && apt-get install -y \
    git \
    wget \
    build-essential \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Clone your forked version of n8n from GitHub (using your fork, not the official repo)
RUN git clone --branch ${N8N_VERSION} https://github.com/DiegoSVilla/n8n.git /usr/src/app

# Change working directory to the cloned n8n repo
WORKDIR /usr/src/app

# Install n8n globally using npm
RUN set -eux; \
    npm install -g --omit=dev n8n@${N8N_VERSION} --ignore-scripts && \
    npm rebuild --prefix=/usr/local/lib/node_modules/n8n sqlite3 && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/chat && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/@n8n/design-system && \
    rm -rf /usr/local/lib/node_modules/n8n/node_modules/n8n-editor-ui/node_modules && \
    find /usr/local/lib/node_modules/n8n -type f -name "*.ts" -o -name "*.js.map" -o -name "*.vue" | xargs rm -f && \
    rm -rf /root/.npm

# Optional: Remove unnecessary files to reduce the image size
RUN rm -rf /usr/src/app/test /usr/src/app/docs /usr/src/app/.git

# Setup the Task Runner Launcher (optional, remove if not needed)
ARG TARGETPLATFORM
ARG LAUNCHER_VERSION=1.1.1
COPY n8n-task-runners.json /etc/n8n-task-runners.json

RUN if [ "$TARGETPLATFORM" = "linux/amd64" ]; then export ARCH_NAME="amd64"; \
    elif [ "$TARGETPLATFORM" = "linux/arm64" ]; then export ARCH_NAME="arm64"; fi; \
    mkdir /launcher-temp && \
    cd /launcher-temp && \
    wget https://github.com/DiegoSVilla/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz && \
    wget https://github.com/DiegoSVilla/task-runner-launcher/releases/download/${LAUNCHER_VERSION}/task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256 && \
    echo "$(cat task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz.sha256) task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz" > checksum.sha256 && \
    sha256sum -c checksum.sha256 && \
    tar xvf task-runner-launcher-${LAUNCHER_VERSION}-linux-${ARCH_NAME}.tar.gz --directory=/usr/local/bin && \
    cd - && \
    rm -r /launcher-temp

# Copy the custom docker-entrypoint.sh (ensure this file exists in your fork)
COPY docker-entrypoint.sh /
RUN apt-get update && apt-get install -y tini

# Create necessary directories and ensure correct permissions
RUN mkdir .n8n && \
    chown node:node .n8n

RUN chmod +x /docker-entrypoint.sh

# Set the shell and user
ENV SHELL /bin/sh
USER node

# Define the entrypoint script for the container to run n8n
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
