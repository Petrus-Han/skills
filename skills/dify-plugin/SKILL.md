---
name: dify-plugin
description: Guide for creating Dify plugins (Tool, Trigger, Extension, Model, Datasource, Agent Strategy). Use when building integrations for Dify workflows, adding new tools, connecting external services via webhooks, or implementing custom model providers. Supports Python SDK with YAML configurations.
---

# Dify Plugin Development

Build plugins that extend Dify's capabilities through tools, triggers, extensions, models, and more.

## Quick Start

1. **Install Dify CLI**: `brew tap langgenius/dify && brew install dify` (or download from GitHub releases)
2. **Install uv**: `curl -LsSf https://astral.sh/uv/install.sh | sh`
3. **Install SDK**: `uv pip install dify_plugin`
4. **Init plugin**: `dify plugin init` (interactive) or `dify plugin init --quick --name my-plugin --category tool --language python`

## Plugin Types

| Type               | Purpose                       | Use When                                           |
| ------------------ | ----------------------------- | -------------------------------------------------- |
| **Tool**           | Add capabilities to workflows | Integrating external APIs (search, database, SaaS) |
| **Trigger**        | Start workflows from events   | Receiving webhooks (GitHub, Slack, custom)         |
| **Extension**      | Custom HTTP endpoints         | Building APIs, OAuth callbacks                     |
| **Model**          | New AI model providers        | Adding LLM/embedding providers                     |
| **Datasource**     | External data connections     | Connecting to databases, knowledge bases           |
| **Agent Strategy** | Custom agent logic            | Implementing specialized reasoning                 |

## Choose Your Plugin Type

- **Tool Plugin**: See [references/tool-plugin.md](references/tool-plugin.md)
- **Trigger Plugin**: See [references/trigger-plugin.md](references/trigger-plugin.md)
- **Extension Plugin**: See [references/extension-plugin.md](references/extension-plugin.md)
- **Model Plugin**: See [references/model-plugin.md](references/model-plugin.md)
- **YAML Schemas**: See [references/yaml-schemas.md](references/yaml-schemas.md)
- **Debugging & Deploy**: See [references/debugging.md](references/debugging.md)

## Core Structure

All plugins share this structure:

```
my-plugin/
├── _assets/icon.svg       # Plugin icon
├── provider/
│   ├── provider.yaml      # Provider config + credentials
│   └── provider.py        # Provider implementation
├── tools/ or events/      # Type-specific implementations
├── manifest.yaml          # Plugin metadata
├── main.py                # Entry point
└── requirements.txt       # Dependencies
```

## manifest.yaml Template

```yaml
version: 0.1.0
type: plugin
author: your-name
name: my-plugin
created_at: "2025-01-01T00:00:00Z"
label:
  en_US: My Plugin
icon: icon.svg
description:
  en_US: Plugin description

resource:
  memory: 134217728
  permission:
    tool:
      enabled: true

plugins:
  tools: # or triggers/endpoints/models
    - provider/provider.yaml

meta:
  version: 0.1.0
  arch: [amd64, arm64]
  runner:
    language: python
    version: "3.12"
    entrypoint: main
```

## main.py Template

```python
from dify_plugin import DifyPluginEnv, Plugin

plugin = Plugin(DifyPluginEnv())

if __name__ == "__main__":
    plugin.run()
```

## Remote Debugging

1. Get debug key from Dify console → Plugins → Remote Debugging
2. Create `.env`:
   ```
   INSTALL_METHOD=remote
   REMOTE_INSTALL_HOST=https://your-dify.com
   REMOTE_INSTALL_PORT=5003
   REMOTE_INSTALL_KEY=your-debug-key
   ```
3. Run: `python -m main`

## Package & Deploy

```bash
dify plugin package ./my-plugin          # Creates my-plugin.difypkg
dify plugin checksum ./my-plugin.difypkg # Verify package
```

## Official Plugin Examples

Reference: [github.com/langgenius/dify-official-plugins](https://github.com/langgenius/dify-official-plugins)

- **Tools**: arxiv, google_search, slack, github
- **Triggers**: github_trigger, slack_trigger, rsshub_trigger
- **Models**: openai, anthropic, google, azure_openai
