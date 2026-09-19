# Model Recommendations per KC Mode

## Overview

Kilo Code (KC) in VS Code has multiple modes. Each mode benefits from different model characteristics. This guide recommends the best model from the Alibaba DashScope Coding Plan for each mode.

---

## Available Models (Alibaba Coding Plan)

| Model | Capabilities | Reasoning | Vision | Speed | Best For |
|-------|-------------|-----------|--------|-------|----------|
| **qwen3.7-plus** | Text + Reasoning + Vision | ✅ (1024 budget) | ✅ | Fast | General coding, architecture |
| **qwen3.6-plus** | Text + Reasoning + Vision | ✅ (1024 budget) | ✅ | Fast | Vision + reasoning tasks |
| **qwen3.5-plus** | Text + Reasoning | ✅ (1024 budget) | ❌ | Medium | Complex reasoning |
| **qwen3-max-2026-01-23** | Text + Reasoning | ✅ | ❌ | Slow | Deep analysis |
| **qwen3-coder-next** | Text Generation | ❌ | ❌ | **Fastest** | Quick fixes, small changes |
| **qwen3-coder-plus** | Text Generation | ❌ | ❌ | Fast | Focused coding |
| **glm-5** | Text + Reasoning | ✅ (1024 budget) | ❌ | Medium | Alternative reasoning |
| **kimi-k2.5** | Text + Reasoning + Vision | ✅ (1024 budget) | ✅ | Medium | Vision + reasoning |
| **MiniMax-M2.5** | Text + Reasoning | ✅ | ❌ | Medium | Alternative reasoning |

---

## Recommended Models per Mode

### 🎯 **Chat Mode** (General conversation, Q&A)

**Recommended**: `alibaba-coding-plan/qwen3.7-plus`

**Why**: Most balanced model with reasoning + vision. Handles general questions well.

**Current default**: ✅ Already set

---

### 💻 **Code Mode** (Code generation, refactoring)

**Recommended**: `alibaba-coding-plan/qwen3-coder-plus`

**Why**: Optimized for fast, focused coding tasks. Text generation only (no reasoning overhead).

**Current setting**: ✅ Already the small model

---

### 🏗️ **Architect Mode** (System design, complex architecture)

**Recommended**: `alibaba-coding-plan/qwen3-max-2026-01-23`

**Why**: Maximum reasoning power for complex problem-solving and system design.

**Current status**: Available in config, not currently default

---

### 🔍 **Ask Mode** (Code explanation, analysis)

**Recommended**: `alibaba-coding-plan/qwen3.5-plus`

**Why**: Good reasoning for analysis without the overhead of vision capabilities.

**Current status**: Available in config

---

### 🖼️ **Vision Tasks** (Image analysis, UI design review)

**Recommended**: `alibaba-coding-plan/qwen3.6-plus`

**Why**: Supports both text + image input with reasoning. Best for visual tasks.

**Current status**: Available in config

---

### ⚡ **Quick Fix Mode** (Small changes, typos, formatting)

**Recommended**: `alibaba-coding-plan/qwen3-coder-next`

**Why**: Fastest model, optimized for quick tasks without reasoning overhead.

**Current status**: Available in config

---

## Multi-Model Workflow Strategy

For complex tasks, use a sequential approach:

1. **Start**: `qwen3.7-plus` for initial understanding and architecture
2. **Implement**: `qwen3-coder-plus` for focused coding
3. **Analyze**: `qwen3.5-plus` for code review and analysis
4. **Complex**: `qwen3-max-2026-01-23` for deep reasoning when needed

---

## Current Configuration

From `~/.config/kilo/kilo.jsonc`:

```json
{
  "model": "alibaba-coding-plan/qwen3.7-plus",
  "small_model": "alibaba-coding-plan/qwen3-coder-plus"
}
```

**Interpretation**:
- `model` (qwen3.7-plus): Used for Chat mode and general tasks
- `small_model` (qwen3-coder-plus): Used for Code mode and quick tasks

---

## Switching Models in VS Code

To switch models in Kilo Code:

1. Open Command Palette (Cmd+Shift+P / Ctrl+Shift+P)
2. Type: "Kilo Code: Change Model"
3. Select from available models

Or edit `~/.config/kilo/kilo.jsonc` directly and restart VS Code.

---

## Cost Considerations

All models in the Alibaba Coding Plan are included in the free tier (1M tokens/month for most models). No additional cost concerns.

**Priority**: Use `qwen3.7-plus` as default, switch to specialized models only when needed.

---

## Fallback Models

If Alibaba models are unavailable, fallback options:

1. **Requesty (Claude)**: `requesty/bedrock/claude-sonnet-4-6@eu-central-1`
2. **NVIDIA**: `nvidia/meta/llama-3.1-70b-instruct`
3. **Groq**: Fast inference for quick tasks

See [FALLBACK_RECOVERY.md](FALLBACK_RECOVERY.md) for detailed fallback procedures.

---

## Summary Table

| KC Mode | Recommended Model | Reason |
|---------|------------------|--------|
| Chat | `qwen3.7-plus` | Balanced, reasoning + vision |
| Code | `qwen3-coder-plus` | Fast, focused coding |
| Architect | `qwen3-max-2026-01-23` | Maximum reasoning |
| Ask | `qwen3.5-plus` | Good analysis, no vision overhead |
| Vision | `qwen3.6-plus` | Text + image + reasoning |
| Quick Fix | `qwen3-coder-next` | Fastest, no reasoning overhead |

---

## Last Updated

2026-09-19
