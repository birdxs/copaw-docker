# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Docker deployment project for CoPaw, a personal assistant product based on AgentScope. CoPaw supports multi-channel conversations (DingTalk, Feishu, QQ, Discord, iMessage) and runs locally with user-configured LLM providers.

**Key Technologies**: Python 3.12, Docker, Docker Compose, AgentScope framework

**Official Documentation**: http://copaw.agentscope.io/docs/

**Official Repository**: https://github.com/agentscope-ai/CoPaw

---

## Critical Warnings

### Security Warning

> **CoPaw has NO authentication or access control. NEVER expose the service port to the public internet!**

- The WebUI management interface has **no login authentication**
- Anyone who can access port 8088 can fully control the CoPaw instance
- Default port `8088` should only be accessed in **trusted internal networks**
- If remote access is required, use:
  - SSH tunnel: `ssh -L 8088:localhost:8088 your-server`
  - Reverse proxy (Nginx/Caddy) with Basic Auth or OAuth
  - Firewall rules to restrict access by IP

### Data Volume Compatibility

> The `copaw-data` storage volume is **NOT compatible** with the official CoPaw image due to different file permission settings.

---

## Common Commands

### Build and Run

```bash
# Build the Docker image (with optional version)
docker compose build
docker compose build --build-arg COPAW_VERSION=0.0.3

# Start the service
docker compose up -d

# View logs
docker compose logs -f copaw

# Stop the service
docker compose stop

# Restart the service
docker compose restart

# Stop and remove containers
docker compose down
```

### Container Interaction

```bash
# Enter the container shell
docker compose exec copaw bash

# Run CoPaw commands inside container
docker compose exec copaw copaw init --defaults
docker compose exec copaw copaw skills config
docker compose exec copaw copaw cron
```

### CoPaw CLI Commands

```bash
# Initialization
docker compose exec copaw copaw init --defaults   # Non-interactive default config
docker compose exec copaw copaw init              # Interactive initialization

# Model Management (Cloud Providers)
docker compose exec copaw copaw models list                    # List all providers
docker compose exec copaw copaw models config                  # Interactive configuration
docker compose exec copaw copaw models config-key modelscope   # Configure API Key
docker compose exec copaw copaw models config-key dashscope    # Configure DashScope API Key
docker compose exec copaw copaw models config-key custom       # Configure custom provider
docker compose exec copaw copaw models set-llm                 # Switch active model

# Model Management (Local Models - llama.cpp / MLX)
docker compose exec copaw copaw models download <repo_id>      # Download local model
docker compose exec copaw copaw models download Qwen/Qwen3-4B-GGUF
docker compose exec copaw copaw models download Qwen/Qwen3-4B --backend mlx
docker compose exec copaw copaw models local                   # List downloaded models
docker compose exec copaw copaw models remove-local <model_id>  # Delete downloaded model

# Model Management (Ollama)
docker compose exec copaw copaw models ollama-pull <model>     # Pull Ollama model
docker compose exec copaw copaw models ollama-list             # List Ollama models
docker compose exec copaw copaw models ollama-remove <model>   # Delete Ollama model

# Channel Management
docker compose exec copaw copaw channels list       # List all channels
docker compose exec copaw copaw channels config     # Interactive configuration
docker compose exec copaw copaw channels install <key>    # Install custom channel
docker compose exec copaw copaw channels add <key>        # Add channel to config
docker compose exec copaw copaw channels remove <key>     # Remove custom channel

# Skills Management
docker compose exec copaw copaw skills list         # List all skills
docker compose exec copaw copaw skills config       # Interactive enable/disable

# Cron Jobs
docker compose exec copaw copaw cron list           # List all jobs
docker compose exec copaw copaw cron create ...     # Create a job
docker compose exec copaw copaw cron state <job_id> # Check job state
docker compose exec copaw copaw cron pause <job_id> # Pause a job
docker compose exec copaw copaw cron resume <job_id># Resume a paused job
docker compose exec copaw copaw cron run <job_id>   # Run once immediately

# Environment Variables
docker compose exec copaw copaw env list            # List all variables
docker compose exec copaw copaw env set KEY VALUE   # Set a variable
docker compose exec copaw copaw env delete KEY      # Delete a variable

# Chat Sessions
docker compose exec copaw copaw chats list          # List all sessions
docker compose exec copaw copaw chats get <id>      # Get session details
docker compose exec copaw copaw chats create ...    # Create new session
docker compose exec copaw copaw chats update <id> --name "New Name"  # Rename session
docker compose exec copaw copaw chats delete <id>   # Delete session

# Maintenance
docker compose exec copaw copaw clean               # Clean working directory (with confirmation)
docker compose exec copaw copaw clean --yes         # Clean without confirmation
docker compose exec copaw copaw clean --dry-run     # Show what would be deleted
```

### Data Management

```bash
# Inspect the data volume
docker volume inspect copaw-data

# Backup data
docker run --rm -v copaw-data:/data -v $(pwd):/backup \
    alpine tar czf /backup/copaw-backup-$(date +%Y%m%d).tar.gz -C /data .

# Restore data
docker run --rm -v copaw-data:/data -v $(pwd):/backup \
    alpine tar xzf /backup/copaw-backup-YYYYMMDD.tar.gz -C /data
```

---

## Architecture

### Dockerfile Structure (Multi-stage Build)

```
[builder stage] → [runtime stage]
     ↓                   ↓
  Install copaw      Copy packages
  (gcc, g++)      Create 'copaw' user
                     Copy scripts
```

- **Builder stage**: Uses `python:3.12-slim`, installs build tools (gcc, g++), and runs `pip install copaw`
  - Supports `COPAW_VERSION` build argument to specify version (default: `latest`)
- **Runtime stage**: Final image with only runtime dependencies (curl, ca-certificates), runs as non-root user `copaw`

### Container Startup Flow

```
docker compose up
        ↓
entrypoint.sh runs
        ↓
Check if config.json exists
        ↓
If missing: run `copaw init --defaults` (if COPAW_AUTO_INIT=true)
        ↓
Validate required files (SOUL.md, AGENTS.md)
        ↓
Execute CMD: copaw app --host 0.0.0.0
        ↓
Service listens on 0.0.0.0:8088
```

### Data Persistence

All CoPaw data is stored in the Docker volume `copaw-data` at `/data/copaw`:

| File/Directory | Purpose |
|----------------|---------|
| `config.json` | Main configuration (channels, heartbeat, language) |
| `SOUL.md` | Agent core identity and behavior rules (required) |
| `AGENTS.md` | Detailed workflow and guidelines (required) |
| `MEMORY.md` | Long-term memory storage |
| `PROFILE.md` | Identity and user profile |
| `HEARTBEAT.md` | Heartbeat task questions |
| `jobs.json` | Cron job list |
| `chats.json` | Chat session list |
| `active_skills/` | Currently active skills |
| `customized_skills/` | User-defined skills |
| `custom_channels/` | User-defined channel modules |
| `memory/` | Agent memory files (with daily logs) |

### Environment Variables

Key variables are defined in two places:

1. **Dockerfile**: Default values (COPAW_WORKING_DIR, COPAW_LOG_LEVEL, etc.)
2. **docker-compose.yml**: Configuration file for environment variables

Critical variables:
- `EMBEDDING_API_KEY` - Required for vector memory search
- `MODELSCOPE_API_KEY` / `DASHSCOPE_API_KEY` / `OPENAI_API_KEY` - LLM provider keys
- `COPAW_AUTO_INIT` - Controls automatic initialization (default: true)

See [.env.example](.env.example) for all available variables.

### Script Responsibilities

- **scripts/entrypoint.sh**: Checks for config.json, runs auto-init if needed, validates required files
- **scripts/healthcheck.sh**: Health check for Docker (curl test or process check)
- **scripts/validate-config.sh**: Validates environment before startup (for CI/CD)
- **scripts/test-startup.sh**: Tests the startup flow end-to-end

---

## CI/CD Workflows

The project uses GitHub Actions for automated testing and image releases:

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| [dev-test.yml](.github/workflows/dev-test.yml) | Test development builds | Push to `dev` branch |
| [prod-test.yml](.github/workflows/prod-test.yml) | Test production image | Push to `main` branch |
| [release-image.yml](.github/workflows/release-image.yml) | Build and publish Docker image | Release creation |

---

## WebUI Console Features

Access http://localhost:8088/ after startup:

| Group | Feature | Description |
|-------|---------|-------------|
| Chat | Chat | Chat with CoPaw, manage sessions |
| Control | Channels | Enable/disable channels, configure credentials |
| Control | Sessions | Filter, rename, delete sessions |
| Control | Cron Jobs | Create/edit/delete tasks, run immediately |
| Agent | Workspace | Edit persona files, view memory, upload/download |
| Agent | Skills | Enable/disable/create/import/delete skills |
| Agent | MCP | Enable/disable/create/delete MCP clients |
| Agent | Runtime Config | Modify max iterations and max input length |
| Settings | Models | Configure providers, manage local/Ollama models, select model |
| Settings | Environment Variables | Add/edit/delete environment variables |

## New Features (CoPaw 0.0.3+)

### MCP (Model Context Protocol) Support

CoPaw now supports connecting to external MCP servers to extend capabilities.

**Prerequisites**: None (Node.js 20.x LTS is pre-installed in the Docker image)

**Configuration Format**:
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/directory"]
    }
  }
}
```

**Management**:
- Use **Console → Agent → MCP** to add/edit/enable/disable/delete MCP clients
- Supports three JSON formats for easy import

### Browser Support (Chromium Headless Mode)

The Docker image includes Chromium browser in headless mode for MCP browser automation.

**MCP Browser Server Configuration**:

Puppeteer MCP:
```json
{
  "mcpServers": {
    "browser": {
      "command": "npx",
      "args": ["-y", "@executeautomation/puppeteer-mcp-server"],
      "env": {
        "HEADLESS": "true",
        "CHROME_PATH": "/usr/bin/chromium"
      }
    }
  }
}
```

Playwright MCP:
```json
{
  "mcpServers": {
    "browser": {
      "command": "npx",
      "args": ["-y", "@executeautomation/playwright-mcp-server"],
      "env": {
        "HEADLESS": "true",
        "PLAYWRIGHT_BROWSERS_PATH": "0"
      }
    }
  }
}
```

**Testing Chromium**:
```bash
# Check Chromium version
docker compose exec copaw chromium --version

# Test headless mode
docker compose exec copaw chromium --headless --no-sandbox --disable-dev-shm-usage --disable-gpu --dump-dom https://example.com
```

### Local Model Support

CoPaw supports running models locally without API Keys:

#### llama.cpp (Cross-platform)
```bash
# Install (not available in Docker by default, requires custom build)
pip install 'copaw[llamacpp]'

# Download model
docker compose exec copaw copaw models download Qwen/Qwen3-4B-GGUF
```

#### MLX (Apple Silicon)
```bash
# Install
pip install 'copaw[mlx]'

# Download model
docker compose exec copaw copaw models download Qwen/Qwen3-4B --backend mlx
```

#### Ollama
```bash
# Install
pip install 'copaw[ollama]'

# Pull model
docker compose exec copaw copaw models ollama-pull mistral:7b
docker compose exec copaw copaw models ollama-pull qwen3:8b
```

**Note**: Local model support requires additional dependencies. For Docker deployment, consider building a custom image with these extras.

### Enhanced CLI Commands

**Channel Management**:
- `copaw channels install <key>` - Install custom channel module
- `copaw channels add <key>` - Add channel to config
- `copaw channels remove <key>` - Remove custom channel

**Cron Job Management**:
- `copaw cron state <job_id>` - Check job runtime state
- `copaw cron pause <job_id>` - Pause a job
- `copaw cron resume <job_id>` - Resume paused job

**Chat Session Management**:
- `copaw chats get <id>` - View session details
- `copaw chats create ...` - Create new session
- `copaw chats update <id> --name "..."` - Rename session
- `copaw chats delete <id>` - Delete session

---

## Important Files

| File | Purpose |
|------|---------|
| [docs/copaw-info.md](docs/copaw-info.md) | Comprehensive CoPaw documentation reference from official docs |
| [.env.example](.env.example) | Environment variable template for API keys and configuration |
| [Dockerfile](Dockerfile) | Multi-stage Docker image definition with version support |
| [docker-compose.yml](docker-compose.yml) | Docker Compose orchestration configuration |
| [scripts/entrypoint.sh](scripts/entrypoint.sh) | Container startup script with auto-init |
| [scripts/healthcheck.sh](scripts/healthcheck.sh) | Health check script for Docker |
| [scripts/validate-config.sh](scripts/validate-config.sh) | Pre-startup configuration validation |
| [scripts/test-startup.sh](scripts/test-startup.sh) | End-to-end startup flow testing |

---

## Multimodal Message Support

| Channel | Text | Image | Video | Audio | File | Send Text | Send Image | Send Video | Send Audio | Send File |
|---------|:----:|:-----:|:-----:|:-----:|:----:|:---------:|:----------:|:----------:|:----------:|:---------:|
| DingTalk | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Feishu | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Discord | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | 🚧 | 🚧 | 🚧 | 🚧 |
| iMessage | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| QQ | ✓ | 🚧 | 🚧 | 🚧 | 🚧 | ✓ | 🚧 | 🚧 | 🚧 | 🚧 |
| Telegram | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

> ✓ = Supported; 🚧 = In progress; ✗ = Not supported

---

## Image Information

### Pre-built Image

- **Repository**: `ghcr.io/log-z/copaw-docker:latest`
- **Pull command**: `docker pull ghcr.io/log-z/copaw-docker:latest`
- **Update frequency**: Updated with CoPaw official releases

### Build Details

- **Base image**: `python:3.12-slim`
- **Python version**: 3.12
- **Node.js version**: 20.x LTS (included for MCP support)
- **Browser**: Chromium (headless mode, for MCP browser automation)
- **Working directory**: `/data/copaw`
- **Run user**: `copaw` (non-root)
- **Exposed port**: 8088
