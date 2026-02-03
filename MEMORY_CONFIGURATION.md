# Memory Configuration

*Last updated: 2026-02-02*

This document tracks the memory system configuration and search capabilities in OpenClaw.

## Overview

OpenClaw uses a two-layer memory system:
1. **Short-term**: Session context (compacted when full)
2. **Long-term**: Markdown files + vector search index

## Memory Files

### File Structure
```
workspace/
├── MEMORY.md                    ← Curated long-term memory (main session only)
├── memory/
│   ├── YYYY-MM-DD.md           ← Daily logs (auto-created)
│   ├── index.md                ← Navigation hub
│   ├── WEEKLY_PROCESS.md       ← Consolidation guide
│   ├── KPIs.md                 ← KPI definitions
│   └── topics/                 ← Topic-based memory
│       ├── architecture.md
│       ├── auth.md
│       ├── channels.md
│       ├── deployment.md
│       ├── lessons-learned.md
│       └── preferences.md
```

### What Goes Where

| File Type | Purpose | When to Write |
|-----------|---------|---------------|
| `MEMORY.md` | Durable facts, preferences, decisions | Significant changes |
| `memory/YYYY-MM-DD.md` | Daily notes, running context | End of session |
| `memory/topics/*.md` | Organized knowledge by topic | Weekly consolidation |

## Memory Search Configuration

### Current Settings

```json
{
  "agents": {
    "defaults": {
      "memorySearch": {
        "enabled": true,
        "sources": ["memory", "sessions"],
        "extraPaths": ["docs", "research", "forge-workspace"],
        "experimental": {
          "sessionMemory": true
        },
        "cache": {
          "enabled": true,
          "maxEntries": 50000
        }
      }
    }
  }
}
```

### Indexed Sources

| Source | Path | Notes |
|--------|------|-------|
| **Memory files** | `memory/**/*.md` | Daily logs + topic files |
| **MEMORY.md** | `MEMORY.md` | Curated long-term memory |
| **Documentation** | `docs/` | Extra path |
| **Research** | `research/` | Extra path |
| **Forge Workspace** | `forge-workspace/` | Extra path |
| **Session transcripts** | `~/.openclaw/agents/*/sessions/*.jsonl` | Experimental |

### Search Features

| Feature | Status | Details |
|---------|--------|---------|
| **Vector search** | ✅ Enabled | Semantic similarity (Gemini embeddings) |
| **Hybrid search** | ✅ Default | BM25 + Vector (70/30 weight) |
| **Embedding cache** | ✅ Enabled | 50,000 entries max |
| **Session memory** | ✅ Enabled | Indexes past conversations |
| **sqlite-vec** | ✅ Auto | SQLite extension for fast vector search |

### Embedding Provider

- **Provider**: Gemini (auto-selected)
- **Model**: `gemini-embedding-001`
- **API Key**: Uses `models.providers.google.apiKey`
- **Fallback**: None configured (Gemini is primary)

## Memory Flush (Pre-Compaction)

When a session approaches its context window, OpenClaw triggers a memory flush:

| Setting | Value |
|---------|-------|
| **Mode** | `safeguard` |
| **Soft threshold** | 4,000 tokens before compaction |
| **Reserve tokens** | 20,000 tokens floor |
| **Trigger** | `contextWindow - reserve - threshold` |

**What happens:**
1. System prompts: "Session nearing compaction. Store durable memories now."
2. Model writes to `memory/YYYY-MM-DD.md`
3. Usually replies `NO_REPLY` (invisible to user)
4. Context compacts after flush

## Tools Available

### `memory_search`
- **Purpose**: Semantic search across all indexed memory
- **Returns**: Snippets with file path, line range, relevance score
- **Max results**: Configurable (default: 5)
- **Sources**: MEMORY.md, memory/*.md, extraPaths, sessions (if enabled)

### `memory_get`
- **Purpose**: Read specific memory file content
- **Parameters**: `path`, `from` (line), `lines` (count)
- **Use case**: Pull full context after finding a snippet

## Session Memory (Experimental)

**Status**: ✅ Enabled

Indexes past session transcripts for semantic search:
- **Location**: `~/.openclaw/agents/{agentId}/sessions/*.jsonl`
- **Reindex trigger**: 100 KB new data OR 50 new messages
- **Privacy**: Isolated per agent

**Use case**: Ask "What did we decide about X yesterday?" and get answers from past conversations.

## Performance Optimizations

### Embedding Cache
- **Enabled**: Yes
- **Max entries**: 50,000
- **Benefit**: Avoids re-embedding unchanged chunks
- **Storage**: SQLite (`~/.openclaw/memory/.sqlite`)

### Hybrid Search Weights
- **Vector**: 70% (semantic similarity)
- **BM25**: 30% (keyword relevance)
- **Result**: Better for both paraphrases AND exact tokens (IDs, symbols)

### Delta Sync
- **Session updates**: Debounced (1.5s)
- **Thresholds**: 100 KB or 50 messages
- **Behavior**: Background async indexing

## Cost Considerations

| Operation | Cost | Notes |
|-----------|------|-------|
| **Initial index** | ~$0.01-0.05 | One-time per file |
| **Daily updates** | Negligible | Only new/changed chunks |
| **Session indexing** | ~$0.001-0.01 | Per session (experimental) |
| **Search queries** | Free | No cost for retrieval |

## Future Improvements

Potential upgrades to consider:

1. **Local embeddings** - Avoid API calls entirely
   ```json
   "provider": "local",
   "local": { "modelPath": "hf:ggml-org/embeddinggemma-300M-GGUF/..." }
   ```

2. **Hybrid weight tuning** - Adjust 70/30 split based on usage patterns

3. **Additional extraPaths** - Index other project directories

4. **Batch indexing** - For large backfills (OpenAI Batch API)

## Maintenance

### Weekly (Sundays)
- Review `memory/YYYY-MM-DD.md` files
- Extract key decisions → `memory/topics/`
- Update `MEMORY.md` with curated insights
- Archive old daily files (optional)

### Monthly
- Review search quality
- Check embedding cache hit rate
- Update topic files with fresh context

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Memory search returns nothing | Check `memorySearch.enabled` and API key |
| Old sessions not found | Session indexing is async; may take a moment |
| Wrong file in results | Check `extraPaths` includes the directory |
| Slow searches | Ensure sqlite-vec is loaded (check logs) |

## References

- [OpenClaw Memory Docs](https://docs.openclaw.ai/concepts/memory)
- `IMPROVEMENTS.md` - Memory management improvement tracking
- `memory/index.md` - In-memory navigation hub
