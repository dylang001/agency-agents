FROM node:20-slim

# System deps
RUN apt-get update && apt-get install -y \
    git \
    bash \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Claude Code CLI globally
RUN npm install -g @anthropic-ai/claude-code

# Install filesystem MCP server (for local file access via MCP)
RUN npm install -g @modelcontextprotocol/server-filesystem

# Copy the repo into the image and install agents
COPY . /opt/agency-agents
WORKDIR /opt/agency-agents
RUN bash scripts/install.sh --tool claude-code --no-interactive

# Copy settings template into place — will be overlaid by volume mount if user
# supplies docker/settings.json at runtime (see docker-compose.yml)
COPY docker/settings.json /root/.claude/settings.json

# Drop into /workspace where local files will be mounted
WORKDIR /workspace

ENTRYPOINT ["claude"]
