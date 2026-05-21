# Code with Claude - Microsoft Foundry Workshop

Build an AI agent with **Microsoft Foundry** and the **Microsoft Agent Framework** that uses a Claude model and calls tools through an MCP server.

## What you will build

An agent for **Sparkles**, a friendly cupcake shop, that:

- Uses **Claude Sonnet 4.6** deployed in Microsoft Foundry
- Loads its persona and welcome banner from MCP **prompts**
- Calls live tools from the **Cupcake Store MCP server**

> **Microsoft Foundry** is Microsoft's hosted platform for deploying AI models (Anthropic, Mistral, and more) and exposing them via an endpoint and key.
>
> **MCP** (Model Context Protocol) is an open standard that lets agents discover and call tools, prompts, and resources from a remote server over HTTP.

## Repository layout

```
.
├── workshop/
│   ├── workshop.md              # Step-by-step lab manual
│   ├── skillable.md             # Skillable lab instructions
│   └── sample-code/             # Final reference implementation
│       ├── agent.py
│       ├── requirements.txt
│       └── .env.sample
└── sparkles-agent/              # Your working folder for the workshop
    ├── .env                     # Fill in your Foundry endpoint + key here
    ├── agent.py                 # Empty starter - you build it up during the workshop
    └── requirements.txt         # Python dependencies
```

## Prerequisites

- An Azure subscription with access to **Microsoft Foundry**
- A deployed Claude model (e.g. `claude-sonnet-4-6`)
- Python 3.10+

## Quick start

1. Follow the [workshop lab manual](workshop/workshop.md) to build the agent step-by-step in `sparkles-agent/`.
2. Or, to run the finished reference agent in [workshop/sample-code/](workshop/sample-code/) directly:

   ```bash
   cd workshop/sample-code
   pip install -r requirements.txt
   cp .env.sample .env   # then edit .env with the variables listed below
   python agent.py
   ```

## Environment variables

Configured in `sparkles-agent/.env` (already present - just edit it):

| Variable | Description |
| --- | --- |
| `FOUNDRY_ENDPOINT` | Target URI of your Foundry deployment, e.g. `https://<resource>.services.ai.azure.com/anthropic` |
| `FOUNDRY_API_KEY` | API key for the Foundry deployment |
| `FOUNDRY_MODEL_DEPLOYMENT` | Deployment name (e.g. `claude-sonnet-4-6`) |

## Troubleshooting

See the [Troubleshooting section](workshop/workshop.md#troubleshooting) of the workshop for common issues and fixes.
