# AI Stack

Self-hosted large language model environment running on a dedicated workstation separate from the Qotom homelab server.

---

## Hardware

| Component | Spec |
|-----------|------|
| **GPUs** | 2× NVIDIA RTX 5090 |
| **VRAM** | 64 GB GDDR7 (32 GB per card) |
| **RAM** | 128 GB DDR5 |

---

## Inference: Ollama

Local LLM inference via **Ollama**, accessed at `http://host.docker.internal:11434` from within Docker containers.

### Models deployed

| Model | Use case |
|-------|----------|
| `qwen3:32b-q8_0` | General purpose — primary daily driver |
| `llama3.3:70b` | RAG / large context tasks |
| `deepseek-r1:70b` | Reasoning-heavy tasks |
| `nomic-embed-text` | Embeddings for RAG pipelines |

---

## RAG & Chat UI: AnythingLLM

**AnythingLLM** deployed via Docker, connected to Ollama for inference and embeddings.

- Deployed via `docker run` (PowerShell) rather than Docker Desktop GUI for reliable port/env configuration
- RAG chunk size: ~1000–1500 tokens (corrected from default 8192)
- Embeddings: `nomic-embed-text` via Ollama
- TTS: **ElevenLabs** integrated

### AnythingLLM Docker run (reference)

```powershell
docker run -d `
  -p 3001:3001 `
  --cap-add SYS_ADMIN `
  -v "${env:USERPROFILE}\anythingllm:/app/server/storage" `
  -e STORAGE_DIR="/app/server/storage" `
  mintplexlabs/anythingllm
```

---

## Orchestration: Fabric

**[Fabric](https://github.com/danielmiessler/fabric)** used for structured AI prompt pipelines — pattern-based prompt templates applied to inputs via CLI.

Functional analog to LangChain prompt pipelines.

---

## Multi-Agent Framework: MCP

**Model Context Protocol (MCP)** used for multi-agent tool frameworks — connecting AI models to external tools, APIs, and data sources in structured agentic workflows.

Functional analog to LangGraph / LangChain tool-use agents.

---

## AI-Assisted Development Tools

Used daily as part of development and automation workflow:

| Tool | Description |
|------|-------------|
| **Claude Code** | Terminal-based agentic coding assistant (Anthropic) |
| **Gemini CLI** | Google Gemini via command line |
| **Codex** | OpenAI code model |

---

```

## Architecture diagram
┌─────────────────────────────────────────────────┐
│              Workstation (dual RTX 5090)        │
│                                                 │
│  ┌──────────────────┐    ┌────────────────────┐ │
│  │  Ollama          │    │  AnythingLLM       │ │
│  │  :11434          │◄───│  :3001             │ │
│  │                  │    │  (Docker)          │ │
│  │  qwen3:32b       │    │                    │ │
│  │  llama3.3:70b    │    │  RAG pipelines     │ │
│  │  deepseek-r1:70b │    │  ElevenLabs TTS    │ │
│  │  nomic-embed-text│    │                    │ │
│  └──────────────────┘    └────────────────────┘ │
│                                                 │
│  Fabric (CLI prompts)   MCP (multi-agent tools) │
│  Claude Code / Gemini CLI / Codex               │
└─────────────────────────────────────────────────┘

```

---

## Notes

- Ollama handles both inference and embeddings — single endpoint for AnythingLLM
- `qwen3:32b-q8_0` runs comfortably across both GPUs at q8 quantization
- `llama3.3:70b` used specifically for RAG tasks requiring large context windows
- Fabric patterns stored locally and version-controlled separately
