# Model Configuration

*Last updated: 2026-02-02*

This document tracks how models are configured and routed in OpenClaw.

## Model Routing Summary

| Task | Model | Provider | Config Path |
|------|-------|----------|-------------|
| **General chat / Content creation** | Kimi K2.5 | Moonshot | `agents.defaults.model.primary` |
| **Coding tasks** | MiniMax-M2.1 | MiniMax | `agents.defaults.subagents.model` |
| **Heartbeat checks** | Haiku | Anthropic | `agents.defaults.heartbeat.model` |
| **Voice (Talk mode)** | GPT-4o Realtime | OpenAI | `talk.modelId` |
| **Image understanding** | Gemini 2.5 Flash | Google | `agents.defaults.imageModel.primary` |
| **Web search / Browser** | Kimi K2.5 | Moonshot | Uses primary model |

## Configured Providers

### Moonshot (Primary)
- **API Key**: `sk-2w...` (configured)
- **Models**: Kimi K2.5, Kimi K2 Thinking
- **Aliases**: `kimi`, `kimi-think`
- **Use case**: General conversation, content creation
- **Base URL**: `https://api.moonshot.ai/v1`

### MiniMax (Coding)
- **API Key**: `sk-cp-hNe...` (configured)
- **Models**: MiniMax-M2.1
- **Alias**: `minimax`
- **Use case**: Coding tasks (via subagents)
- **Base URL**: `https://api.minimaxi.chat/v1`
- **Cost**: $15/M input, $60/M output

### DeepSeek
- **API Key**: `sk-d74a...` (configured)
- **Models**: DeepSeek V3
- **Alias**: `deepseek`
- **Use case**: Available for general use
- **Base URL**: `https://api.deepseek.com/v1`
- **Cost**: $0.27/M input, $1.10/M output

### Anthropic (Heartbeats)
- **API Key**: Via auth profile
- **Models**: Haiku, Opus 4.5
- **Alias**: `opus`
- **Use case**: Heartbeat checks (cost-efficient)

### OpenAI (Voice)
- **API Key**: `sk-proj-w113...` (configured)
- **Models**: GPT-4o Realtime
- **Use case**: Voice/Talk mode
- **Voice**: Alloy
- **Output**: PCM16

### Google (Images)
- **API Key**: `AIzaSyCUp...` (configured)
- **Models**: Gemini 2.5 Flash
- **Alias**: `gemini-flash`
- **Use case**: Image understanding
- **Base URL**: `https://generativelanguage.googleapis.com/v1beta`
- **Cost**: $0.15/M input, $0.60/M output
- **Features**: Text + Image input

## Performance Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `timeoutSeconds` | 120 | Max run time before auto-abort |
| `heartbeat.every` | 1h | How often heartbeats run |
| `maxConcurrent` | 4 | Max main agent runs |
| `subagents.maxConcurrent` | 8 | Max subagent runs |

## Aliases Available

Use these shorthand aliases when switching models:
- `/model kimi` → Kimi K2.5
- `/model kimi-think` → Kimi K2.5 Thinking
- `/model minimax` → MiniMax-M2.1
- `/model deepseek` → DeepSeek V3
- `/model opus` → Claude Opus 4.5
- `/model gemini-flash` → Gemini 2.5 Flash

## Cost Tracking

Models with cost data configured:
- MiniMax: $15/M input, $60/M output
- DeepSeek: $0.27/M input, $1.10/M output
- Gemini: $0.15/M input, $0.60/M output

## Notes

- Heartbeats run every 1 hour using Haiku (low cost)
- Coding tasks automatically spawn subagents with MiniMax
- Images automatically route to Gemini Flash
- Voice mode uses OpenAI Realtime
- Run timeout set to 120 seconds to prevent long-running processes
