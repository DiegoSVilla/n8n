# Use a Node.js base image
FROM node:20

# Set build arguments
ARG N8N_VERSION=latest  # Default to latest version, can be changed to a specific version like "1.1.1"

# Labels for image metadata
LABEL org.opencontainers.image.title="n8n"
LABEL org.opencontainers.image.description="Workflow Automation Tool"
LABEL org.opencontainers.image.source="https://github.com/DiegoSVilla/n8n"
LABEL org.opencontainers.image.url="https://n8n.io"
LABEL org.opencontainers.image.version=${N8N_VERSION}

# Set environment variables
ENV N8N_VERSION=${N8N_VERSION}
ENV NODE_ENV=production
ENV N8N_RELEASE_TYPE=stable

# Install required system dependencies
RUN apt-get update && apt-get install -y \
    git \
    wget \
    build-essential \
    python3 \
    && rm -rf /var/lib/apt/lists/*

# Install pnpm (needed for workspaces)
RUN npm install -g pnpm@10.2.1

# Clone your forked version of n8n from GitHub (or use a local copy)
RUN git clone --branch ${N8N_VERSION} https://github.com/DiegoSVilla/n8n.git /usr/src/app

# Change working directory to the cloned n8n repo
WORKDIR /usr/src/app

# Install dependencies using pnpm (workspaces support)
RUN pnpm install --frozen-lockfile

# Build n8n (if necessary)
RUN pnpm build

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

# Install tini
RUN apt-get update && apt-get install -y tini

# Create necessary directories and ensure correct permissions
RUN mkdir .n8n && \
    chown node:node .n8n

# Set the shell and user
ENV SHELL /bin/sh
USER node

# Set the entrypoint script for the container to run n8n
ENTRYPOINT ["tini", "--", "/docker-entrypoint.sh"]
