# Claude Code CLI vs Claude SDK — Categorized Summary

---

## Category 1: Understanding the Two Development Contexts

### Core Distinction
- **Claude Code CLI** is an agentic coding *assistant* — a dev tool you use to build apps
- **Claude SDK** (`@anthropic-ai/sdk` / `anthropic`) is a *library* your application uses at runtime
- They operate at completely different layers and are not interchangeable

### How to Signal Your Intent to Claude Code
- Be explicit at conversation start: mention the package name, language, and architecture intent
- Key phrase: *"I want to build an app using the Anthropic Claude SDK (`@anthropic-ai/sdk`)"*
- This prevents Claude Code from scaffolding the wrong project type

---

## Category 2: File & Project Structure

### CLI-Assisted Project
- No Anthropic dependency in `package.json`
- No runtime AI logic files
- Structure is purely framework-driven

### SDK-Based Project
- `@anthropic-ai/sdk` in `package.json`
- `ANTHROPIC_API_KEY` in `.env`
- Dedicated folders for agents, tools, prompts, memory, MCP

### The `.claude/` Folder Rule
- `.claude/` belongs to **you as a developer** (like `.vscode/`)
- `src/` belongs to **your application** (ships to production)
- `.claude/` never ships to production

---

## Category 3: Terminology Disambiguation (CLI vs SDK)

| Term | CLI Meaning | SDK Meaning |
|---|---|---|
| **Hooks** | Lifecycle scripts in `.claude/settings.json` | Middleware/callbacks you implement yourself |
| **Skills** | Custom slash commands in `.claude/commands/` | Reusable functions you build |
| **Tools** | Built-in CLI capabilities (file read, bash, etc.) | Function calling via the API |
| **MCP Servers** | Dev tool connections in `.claude/settings.json` | Runtime servers your app hosts or connects to |

---

## Category 4: When to Use CLI Features vs SDK Features

### Use CLI Hooks, Skills, MCP when:
- Automating *your own* coding workflow
- The beneficiary is **you, the developer**
- Example: auto-run tests after Claude edits a file

### Use SDK when:
- Building capabilities for *your application's users*
- The beneficiary is **your app at runtime**
- Example: an agent that calls tools, manages state, improves itself

### Decision Rule:
> *"Who needs this feature — me while coding, or my app's users at runtime?"*

---

## Category 5: Your Specific Project Architecture

### Requirements Identified
1. Multiple Claude agents doing specific tasks
2. Self-improving agents that modify Python scripts based on feedback
3. Standalone deployable product

### Verdict
- **Claude Code CLI** → use to *build and debug* your system (dev experience only)
- **Claude SDK** → the only viable choice for production

### Key Architectural Components

| Layer | Component | Purpose |
|---|---|---|
| Orchestration | `orchestrator.ts` | Routes tasks, manages agent coordination |
| Agents | `agents/` | Specialist agents + script runner agent |
| Tools | `tools/` | `executeScript`, `readScript`, `writeScript`, `validateScript` |
| Self-improvement | `feedback/` | Feedback collection, improvement agent, version store |
| Memory | `memory/` | Conversation history, script modification log |
| Safety | `scripts/script_versions/` | Rollback capability |

### Self-Improving Script Loop
```
Feedback → ImprovementAgent → rewrites script → validate
→ versionStore saves old → execute new → result to orchestrator
```

### Critical Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Broken script written by agent | Validate before replacing |
| No rollback | Version store with N history |
| Runaway execution | Timeouts + sandboxed subprocess |
| Conflicting agent edits | Lock files / script ownership model |
| Prompt injection via script output | Sanitize stdout before feeding to Claude |

---

## Category 6: Hooks & Skills in Production — The Core Insight

### The Misconception
> CLI Hooks and Skills do NOT "port" to production — they are not an API your app calls

### The Reality
- They are **patterns** (intercept-and-act, reusable-capability) that CLI provides as convenience abstractions
- In production, you **re-implement the same patterns** using native SDK code

### The Pattern Mapping

| Pattern | CLI Abstraction | Your SDK Implementation |
|---|---|---|
| Pre/post tool interception | `PreToolUse` / `PostToolUse` Hook | Middleware wrapping your tool executor |
| Event-driven reactions | `Stop` / `Notification` Hook | Callbacks in your agent loop |
| Reusable named capability | Skill / slash command | Regular function or class |

### Why SDK Ownership is Better for Your Use Case

| Capability | CLI Hooks/Skills | Your SDK Implementation |
|---|---|---|
| Works in production | ❌ | ✅ |
| Triggered by other agents | ❌ | ✅ |
| Stores state across runs | ❌ | ✅ |
| Modified by agents at runtime | ❌ | ✅ |
| Deployable to a server | ❌ | ✅ |

---

## One-Line Summary Per Category

| # | Category | One-Line Takeaway |
|---|---|---|
| 1 | Two Contexts | CLI is your assistant; SDK is your app's engine |
| 2 | File Structure | `.claude/` is yours; `src/` is your app's |
| 3 | Terminology | Same words mean different things in CLI vs SDK |
| 4 | When to Use What | Ask who benefits — you or your users |
| 5 | Your Project | SDK-only production, CLI-assisted development |
| 6 | Hooks & Skills | Not ported — re-implemented as regular code patterns |
