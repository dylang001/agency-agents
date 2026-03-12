FROM node:22-slim

# Install Claude Code and MCP servers
RUN npm install -g @anthropic-ai/claude-code \
    @modelcontextprotocol/server-filesystem \
    airtable-mcp-server

# Copy repo to bake agents into the image
WORKDIR /app
COPY . .

# Install agents: copy all .md files with YAML frontmatter (first line = ---) to ~/.claude/agents/
RUN mkdir -p /root/.claude/agents && \
    for dir in design engineering game-development marketing paid-media sales \
                product project-management testing support spatial-computing specialized; do \
      if [ -d "/app/$dir" ]; then \
        find "/app/$dir" -name "*.md" -type f -exec sh -c \
          'first=$(head -1 "$1"); [ "$first" = "---" ] && cp "$1" /root/.claude/agents/ && echo "  + $(basename $1)"' \
          _ {} \; ; \
      fi; \
    done && \
    printf "\nAgents installed: %s\n" "$(ls /root/.claude/agents/ | wc -l | tr -d ' ')"

WORKDIR /workspace

ENTRYPOINT ["claude"]
