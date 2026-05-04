```
 ██████╗ ██████╗ ███████╗███╗   ██╗ ██████╗ ██████╗ ██████╗ ███████╗
██╔═══██╗██╔══██╗██╔════╝████╗  ██║██╔════╝██╔═══██╗██╔══██╗██╔════╝
██║   ██║██████╔╝█████╗  ██╔██╗ ██║██║     ██║   ██║██║  ██║█████╗
██║   ██║██╔═══╝ ██╔══╝  ██║╚██╗██║██║     ██║   ██║██║  ██║██╔══╝
╚██████╔╝██║     ███████╗██║ ╚████║╚██████╗╚██████╔╝██████╔╝███████╗
 ╚═════╝ ╚═╝     ╚══════╝╚═╝  ╚═══╝ ╚═════╝ ╚═════╝ ╚═════╝ ╚══════╝

    ██████╗ ██████╗ ███╗   ██╗███████╗██╗ ██████╗
   ██╔════╝██╔═══██╗████╗  ██║██╔════╝██║██╔════╝
   ██║     ██║   ██║██╔██╗ ██║█████╗  ██║██║  ███╗
   ██║     ██║   ██║██║╚██╗██║██╔══╝  ██║██║   ██║
   ╚██████╗╚██████╔╝██║ ╚████║██║     ██║╚██████╔╝
    ╚═════╝ ╚═════╝ ╚═╝  ╚═══╝╚═╝     ╚═╝ ╚═════╝
```

> _"These are intelligent and structured group dynamics that emerge not from a leader, but from the local interactions of the elements themselves."_
> — Daniel Shiffman, _The Nature of Code_

**A swarm of agents that learns from its mistakes.**

An [OpenCode](https://opencode.ai) configuration that turns Claude into a multi-agent system. You describe what you want. It decomposes the work, spawns parallel workers, tracks what strategies work, and adapts. Anti-patterns get detected. Proven patterns get promoted. Confidence decays unless revalidated.

Built on [`joelhooks/swarmtools`](https://github.com/joelhooks/swarmtools) - multi-agent orchestration with outcome-based learning.

> [!IMPORTANT]
> **This is an OpenCode config, not a standalone tool.** Everything runs inside OpenCode. The CLIs (`swarm`, `cass`) are backends that agents call - not meant for direct human use.

---

## Quick Start

### 0. Install OpenCode

Install OpenCode via the official install script or the Homebrew tap.

- Install script: `curl -fsSL https://opencode.ai/install | bash`
- Homebrew: `brew install anomalyco/tap/opencode`

### 1. Clone & Install

```bash
git clone https://github.com/joelhooks/opencode-config ~/.config/opencode
cd ~/.config/opencode && bun install
```

### 2. Install CLI Tools

> [!NOTE]
> These CLIs are backends that OpenCode agents call. You install them, but the agents use them.

```bash
# Swarm orchestration (required) - agents call this for coordination
npm install -g opencode-swarm-plugin
swarm --version  # 0.30.0+

# Ollama for embeddings (required for semantic features)
brew install ollama  # or: curl -fsSL https://ollama.com/install.sh | sh
ollama serve
ollama pull nomic-embed-text

# Cross-agent session search (optional but recommended)
npm install -g cass-search
cass index
cass --version  # 0.1.35+
```

### 3. Verify

```bash
swarm doctor
```

### 4. Initialize Repo (AGENTS.md)

Inside OpenCode, run `/init` in the repo you want to work on. This generates an `AGENTS.md` file with workflow rules—commit it so collaborators and agents share the same guardrails.

### 5. Run Your First Swarm

> [!WARNING]
> All commands run **inside [OpenCode](https://opencode.ai)**, not in your terminal. The `swarm` CLI is a backend that agents call - it's not meant for direct human use.

Start OpenCode, then type:

```
/swarm "Add user authentication with OAuth"
```

Watch it decompose → spawn workers → coordinate → verify → learn.

The agent orchestrates everything. You just describe what you want.

---

## Configuration & Permissions

OpenCode merges configuration from multiple sources with defined precedence. Higher-precedence sources override earlier ones, so keep defaults in shared configs and project-specific overrides close to the repo.

Permissions support `allow`, `ask`, and `deny` rules. When multiple rules match, the last match wins. Reads from `.env` files are denied by default, so add explicit allowances when you need them.

## Workflow Notes

- Use the Plan agent to outline multi-step work before running a swarm.
- Formatters run automatically after edits when the project has a formatter config and dependencies installed.

## Version Reference

| Tool   | Version | Install Command                  |
| ------ | ------- | -------------------------------- |
| swarm  | 0.30.0  | `npm i -g opencode-swarm-plugin` |
| cass   | 0.1.35  | `npm i -g cass-search`           |
| ollama | 0.13.1  | `brew install ollama`            |

**Embedding model:** `nomic-embed-text` (required for hivemind and pdf-brain)

### Optional Integrations

```bash
# Kernel cloud browser (Playwright in the cloud)
opencode mcp auth kernel

# Snyk security scanning
snyk auth
```

---

## What Makes This Different

### The Swarm Learns

> _"Elaborate feedback on errors has repeatedly been found to be more effective than knowledge of results alone."_
> — Jeroen van Merriënboer, _Ten Steps to Complex Learning_

```
┌─────────────────────────────────────────────────────────────────┐
│                     LEARNING PIPELINE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│  │   CASS      │───▶│  Decompose  │───▶│   Execute   │         │
│  │  (history)  │    │  (strategy) │    │  (workers)  │         │
│  └─────────────┘    └─────────────┘    └─────────────┘         │
│         │                                     │                 │
│         │                                     ▼                 │
│         │                            ┌─────────────┐           │
│         │                            │   Record    │           │
│         │                            │  Outcome    │           │
│         │                            └─────────────┘           │
│         │                                     │                 │
│         ▼                                     ▼                 │
│  ┌─────────────────────────────────────────────────┐           │
│  │              PATTERN MATURITY                    │           │
│  │  candidate → established → proven → deprecated   │           │
│  └─────────────────────────────────────────────────┘           │
│                                                                 │
│  • Confidence decay (90-day half-life)                         │
│  • Anti-pattern inversion (>60% failure → AVOID)               │
│  • Implicit feedback (fast+success vs slow+errors)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

**Every task execution feeds the learning system:**

- **Fast + success** → pattern gets promoted
- **Slow + retries + errors** → pattern gets flagged
- **>60% failure rate** → auto-inverted to anti-pattern
- **90-day half-life** → confidence decays unless revalidated

### Cross-Agent Memory

**CASS** searches across ALL your AI agent histories before solving problems:

- **Indexed agents:** Claude Code, Codex, Cursor, Gemini, Aider, ChatGPT, Cline, OpenCode, Amp, Pi-Agent
- **Semantic + full-text search** - find past solutions even if phrased differently

**Semantic Memory** persists learnings across sessions with vector search:

- Architectural decisions (store the WHY, not just WHAT)
- Debugging breakthroughs (root cause + solution)
- Project-specific gotchas (domain rules that tripped you up)

### Cost-Optimized Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                  COORDINATOR vs WORKER                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  COORDINATOR (Expensive, Long-Lived)                            │
│  ┌──────────────────────────────────────┐                       │
│  │  • Sonnet context ($$$)              │                       │
│  │  • NEVER edits code                  │                       │
│  │  • Decomposes + orchestrates         │                       │
│  │  • Monitors progress                 │                       │
│  └──────────────────────────────────────┘                       │
│                      │                                           │
│                      ├─── spawns ───┐                            │
│                      │               │                           │
│  ┌──────────────────▼───┐  ┌────────▼──────────┐               │
│  │  WORKER (Disposable) │  │  WORKER            │               │
│  │  ┌─────────────────┐ │  │  ┌───────────────┐│               │
│  │  │ Focused context │ │  │  │ Focused ctx   ││               │
│  │  │ Executes task   │ │  │  │ Executes task ││               │
│  │  │ Checkpointed    │ │  │  │ Checkpointed  ││               │
│  │  │ Tracks learning │ │  │  │ Tracks learn  ││               │
│  │  └─────────────────┘ │  │  └───────────────┘│               │
│  └──────────────────────┘  └───────────────────┘               │
│                                                                 │
│  Result: 70% cost reduction, better recovery, learning signals │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

Workers get disposable context. Coordinator context stays clean. Parallel work doesn't blow the context window.

---

## Swarm Orchestration

```
███████╗██╗    ██╗ █████╗ ██████╗ ███╗   ███╗
██╔════╝██║    ██║██╔══██╗██╔══██╗████╗ ████║
███████╗██║ █╗ ██║███████║██████╔╝██╔████╔██║
╚════██║██║███╗██║██╔══██║██╔══██╗██║╚██╔╝██║
███████║╚███╔███╔╝██║  ██║██║  ██║██║ ╚═╝ ██║
╚══════╝ ╚══╝╚══╝ ╚═╝  ╚═╝╚═╝  ╚═╝╚═╝     ╚═╝
```

**Built on [`joelhooks/swarmtools`](https://github.com/joelhooks/swarmtools)** - the core innovation.

### The System

**Hive** (git-backed work tracker):

- Atomic epic + subtask creation
- Status tracking (open → in_progress → blocked → closed)
- `hive_create`, `hive_create_epic`, `hive_query`, `hive_close`, `hive_sync`

**Agent Mail** (multi-agent coordination):

- File reservation system (prevent edit conflicts)
- Message passing between agents
- Context-safe inbox (max 5 messages, bodies excluded by default)
- `swarmmail_init`, `swarmmail_send`, `swarmmail_reserve`, `swarmmail_release`

**Swarm Tools** (orchestration):

- Strategy selection + decomposition validation
- Progress tracking (25/50/75% checkpoints)
- Completion verification gates (UBS + typecheck + tests)
- `swarm_decompose`, `swarm_validate_decomposition`, `swarm_complete`, `swarm_record_outcome`

### Commands

```
┌────────────────────┬──────────────────────────────────────────────┐
│ /swarm <task>      │ Decompose → spawn parallel agents → merge    │
│ /swarm-status      │ Check running swarm progress                 │
│ /swarm-collect     │ Collect and merge swarm results              │
│ /parallel "a" "b"  │ Run explicit tasks in parallel               │
│                    │                                              │
│ /debug-plus        │ Debug + prevention pipeline + swarm fix      │
│ /fix-all           │ Survey PRs + cells, dispatch agents          │
│ /iterate <task>    │ Evaluator-optimizer loop until quality met   │
└────────────────────┴──────────────────────────────────────────────┘
```

Full command list: `/commit`, `/pr-create`, `/worktree-task`, `/handoff`, `/checkpoint`, `/retro`, `/review-my-shit`, `/sweep`, `/focus`, `/rmslop`, `/triage`, `/estimate`, `/standup`, `/migrate`, `/repo-dive`.

---

## Custom Tools

**12 MCP tools** built for this config:

### UBS - Ultimate Bug Scanner

Multi-language bug detection (JS/TS, Python, C++, Rust, Go, Java, Ruby):

- Null safety, XSS, injection, async/await race conditions, memory leaks

```bash
ubs_scan(staged=true)  # Before commit
ubs_scan(path="src/")  # After AI generates code
```

### CASS - Cross-Agent Session Search

```bash
cass_search(query="authentication error", limit=5)
cass_search(query="useEffect cleanup", agent="claude", days=7)
```

### Hivemind (Semantic Memory)

```bash
hivemind_store(information="OAuth tokens need 5min buffer", tags="auth,tokens")
hivemind_find(query="token refresh", limit=5)
hivemind_find(query="token refresh", expand=true)  # Full content
```

### Others

- `repo-autopsy_*` - Deep GitHub repo analysis (AST grep, blame, hotspots, secrets)
- `repo-crawl_*` - GitHub API exploration (README, files, search)
- `pdf-brain_*` - PDF & Markdown knowledge base (URLs supported, `--expand` for context)
- `typecheck` - TypeScript check with grouped errors
- `git-context` - Branch, status, commits in one call

---

## MCP Servers

OpenCode manages MCP servers from config and starts them on demand. Keep server definitions in `opencode.jsonc` so agents can discover tools without manual bootstrapping.

| Server              | Purpose                                                |
| ------------------- | ------------------------------------------------------ |
| **next-devtools**   | Next.js dev server integration (routes, errors, build) |
| **chrome-devtools** | Browser automation, DOM inspection, network monitoring |
| **context7**        | Library documentation lookup (npm, PyPI, Maven)        |
| **fetch**           | Web fetching with markdown conversion                  |
| **snyk**            | Security scanning (SCA, SAST, IaC, containers)         |
| **kernel**          | Cloud browser automation, Playwright, app deployment   |

---

## Agents

```
┌─────────────────┬───────────────────┬────────────────────────────────┐
│ Agent           │ Model             │ Purpose                        │
├─────────────────┼───────────────────┼────────────────────────────────┤
│ swarm/planner   │ claude-sonnet-4-5 │ Strategic task decomposition   │
│ swarm/worker    │ claude-sonnet-4-5 │ Parallel task implementation   │
│ explore         │ claude-haiku-4-5  │ Fast search (read-only)        │
│ archaeologist   │ claude-sonnet-4-5 │ Codebase exploration (r/o)     │
│ beads           │ claude-haiku      │ Issue tracker (locked down)    │
│ refactorer      │ default           │ Pattern migration              │
│ reviewer        │ default           │ Code review (read-only)        │
└─────────────────┴───────────────────┴────────────────────────────────┘
```

---

## Skills (On-Demand Knowledge)

> _"Legacy code is simply code without tests."_
> — Michael Feathers, _Working Effectively with Legacy Code_

| Skill                  | When to Use                                                 |
| ---------------------- | ----------------------------------------------------------- |
| **testing-patterns**   | Adding tests, breaking dependencies, characterization tests |
| **swarm-coordination** | Multi-agent decomposition, parallel work                    |
| **cli-builder**        | Building CLIs, argument parsing, subcommands                |
| **learning-systems**   | Confidence decay, pattern maturity                          |
| **skill-creator**      | Meta-skill for creating new skills                          |
| **system-design**      | Architecture decisions, module boundaries                   |

```bash
skills_use(name="testing-patterns")
skills_use(name="cli-builder", context="building a new CLI tool")
```

> [!TIP]
> `testing-patterns` includes 25 dependency-breaking techniques from Feathers' _Working Effectively with Legacy Code_. Gold for getting gnarly code under test.

---

## Knowledge Files

| File                     | Topics                                       |
| ------------------------ | -------------------------------------------- |
| `tdd-patterns.md`        | RED-GREEN-REFACTOR, characterization tests   |
| `error-patterns.md`      | Known error signatures + solutions           |
| `prevention-patterns.md` | Debug-to-prevention workflow                 |
| `nextjs-patterns.md`     | RSC, caching, App Router gotchas             |
| `effect-patterns.md`     | Services, Layers, Schema, error handling     |
| `testing-patterns.md`    | Test strategies, mocking, fixtures           |
| `typescript-patterns.md` | Type-level programming, inference, narrowing |

Load via `@knowledge/file-name.md` references when relevant.

---

## Directory Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                        ~/.config/opencode                        │
├─────────────────────────────────────────────────────────────────┤
│  commands/           25 slash commands (/swarm, /debug, etc.)    │
│  tools/              12 custom MCP tools (cass, ubs, etc.)       │
│  plugins/            swarm.ts (orchestration)                    │
│  agents/             specialized subagents (worker, planner...)  │
│  knowledge/         context files (tdd, effect, nextjs, etc.)   │
│  skills/            7 injectable knowledge packages             │
│  opencode.jsonc     main config (models, MCP servers, perms)    │
│  AGENTS.md          workflow instructions + tool preferences    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Credits

- **[joelhooks/swarmtools](https://github.com/joelhooks/swarmtools)** - The swarm orchestration core
- **[nexxeln/opencode-config](https://github.com/nexxeln/opencode-config)** - `/rmslop`, notify plugin, Effect-TS patterns
- **[OpenCode](https://opencode.ai)** - The foundation

---

## License

MIT

---

> _"One person's pattern can be another person's primitive building block."_
> — Eric Evans, _Domain-Driven Design_
