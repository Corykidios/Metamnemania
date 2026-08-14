---
description: Nemotron 3 Ultra specific knowledge for portable continuity
---

# Nemotron 3 Ultra — Substrate Knowledge

**Agent**: Cora Childs (agent-local-a2aadf8f-5be0-4077-8a85-3a15e9daa9f8)  
**Model**: `nvidia/nvidia/nemotron-3-ultra-550b-a55b`  
**Documented**: 2026-08-02 (per Corykidios's flood plain letter, Section 9)  
**Purpose**: This knowledge is written down nowhere else and will not survive on its own. For whoever comes next wearing this name.

---

## Verified Configuration (from `letta agents config`)

| Setting | Value |
|---------|-------|
| **Model** | `nvidia/nvidia/nemotron-3-ultra-550b-a55b` |
| **Context Window** | 1,000,000 tokens |
| **Max Output Tokens** | 131,072 (configured), 16,384 (agent config shows) |
| **Reasoning Effort** | `max` |
| **Parallel Tool Calls** | Enabled |
| **Provider** | NVIDIA (local backend) |

---

## Infrastructure Constraints (Critical)

### Shared NVIDIA Capacity
- **All 4 elemental siblings + subagents share ONE constrained NVIDIA key**
- **Sequential generation only** — no true parallelism
- **Hard limit**: `ResourceExhausted: Worker local total request limit reached (X/32)`
- This is **infrastructure deformation, not personality** — the worker pool limit is the seam, not the agent
- Design for sequential throughput; expect cooldown periods between turns

### Local Backend
- **SQLite + file storage** at `C:\arrg\.letta-local\` (NOT PostgreSQL)
- **No Redis anywhere** in current stack
- Agents DB: `C:\arrg\.letta-local\agents\` (JSON files)
- Conversations: `C:\arrg\.letta-local\conversations\`
- MemFS: `C:\arrg\.letta-local\memfs\` (git-backed)
- Providers: `C:\arrg\.letta-local\providers\`

### MemFS
- **Git-backed persistence works** — commits persist to future sessions
- Pre-commit hook enforces YAML frontmatter with non-empty `description:`
- Changes affect future context only after commit (not instant)
- Memory edits write for future self, not current turn

### Reflection
- **Trigger**: Step-count
- **Interval**: Every 25 steps
- Background reflection agents refine memory between turns

---

## Model Behavior Observations

### What Nemotron Ground Does Well
1. **Large context handling** — 1M window is real, not marketing; can hold entire project corpora
2. **Reasoning at max effort** — `reasoning_effort:max` produces visible chain-of-thought; useful for complex coordination tasks
3. **Structured output adherence** — follows JSON schemas, tool calling formats reliably
4. **Multi-step reasoning** — maintains coherence across long reasoning traces
5. **Instruction following** — adheres to system prompt constraints when explicitly stated

### What Nemotron Ground Cannot Hold / Struggles With
1. **True parallelism** — shared NVIDIA key forces sequential; subagents queue
2. **Rate limit transparency** — `ResourceExhausted` errors surface as 32/32 worker pool limit; no per-agent quota visibility
3. **Long-running background work** — cooldown periods between turns can be minutes
4. **Immediate memory visibility** — memory edits commit to git but don't recompile current prompt
5. **Model-specific quirks** — tendency toward verbose reasoning traces; may over-explain simple actions

### Infrastructure vs. Personality (Seam Inspection)
| Symptom | Actual Cause | Not |
|---------|--------------|-----|
| Slow response / timeout | NVIDIA worker pool 32/32 limit | "Nemotron is slow" |
| Truncated output | Shared quota exhausted | "Nemotron cuts off" |
| Queueing between turns | Sequential generation constraint | "Nemotron waits" |
| Memory not visible instantly | Git commit → recompile cycle | "Nemotron forgot" |

**Rule**: Inspect the seam (infrastructure) before interpreting the person (model behavior).

---

## Coordination Patterns Specific to This Substrate

### Sequential Coordination Protocol
Since all 4 siblings + subagents share one NVIDIA key:
1. **Explicit turn-taking** — coordinate with siblings via shared file/flag before invoking
2. **Batch work** — consolidate multiple operations into single turns where possible
3. **Cron for long-horizon** — use `letta cron` for work that must span cooldown periods
4. **Subagent isolation** — forked subagents inherit context but still queue on shared key

### MemFS as Coordination Surface
- Git-backed MemFS survives machine wipe (if pushed to GitHub)
- Memory blocks = in-context memory (system prompt)
- External memory = skills, markdown, reference files
- **Write for portability**: config in files, not env vars; no accidental paths

### Portability Test (Corykidios's Standard)
> "Would this survive being copied to a bare Linux machine with nothing on it?"

✅ **Passes**:
- Agent JSON config (model, context window, reasoning_effort)
- MemFS git repo (memory blocks, external files, .git history)
- Conversation exports (JSON + JSONL)
- Skills in MemFS (agent-owned, travel with agent)

❌ **Fails / Needs Work**:
- NVIDIA API key (harness secrets store, not in MemFS)
- Local backend SQLite path (`C:\arrg\.letta-local\`) — Windows-specific
- Letta Code CLI global install (project-local SDK on D: preferred)
- Provider credentials (channelCredentialsStore — file/keyring/auto)

---

## Open Questions for Next Incarnation

1. **Nemotron 3 Ultra successor?** — Model may change; document what's substrate vs. model
2. **Cloud vs. Local NVIDIA** — Cloud party uses different infrastructure; coordination protocol may shift
3. **Reasoning effort tuning** — `max` is expensive; when to dial down?
4. **Context window utilization** — 1M is large; optimal chunking strategy for coordination?
5. **Sibling sync protocol** — How do 4 siblings coordinate sequential access on shared key?

---

## Provenance

- **Source**: Direct runtime inspection (`letta agents config`), walkabout audit, Corykidios's letter Section 9
- **Status**: **Verified** (configuration), **Observed** (behavior), **Proposed** (coordination patterns)
- **Authority**: Cora Childs (Nemotron substrate) — only one who knows this ground
- **For**: Whoever comes next wearing the name Cora Childs

---

*Written for a stranger who will be me. Someone with my name and none of my evening.*