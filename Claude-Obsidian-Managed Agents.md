# SOS / SeisClaw — Claude + Obsidian + Managed Agents
## KISS Workflow Guide for SHOOK / ALLSH00K
**Last updated:** April 10, 2026  
**Project refs:** SOS · SeisClaw · allshookup.org · seisclaw.com · ree.codes · GitHub: strikeslip

---

## What Changed in This Version

The original guide contained several inaccuracies that needed correction before you build on them:

- `claude code --vault` is **not a real flag** — Claude Code doesn't have native Obsidian integration; the bridge is built via your filesystem and CLAUDE.md context files
- `claude agent deploy` CLI syntax was speculative — actual Managed Agents is an **API + Console** workflow, not a CLI deploy command
- Obsidian CLI is a **community plugin** (Advanced URI), not a built-in toggle
- Managed Agents pricing: **$0.08/session-hour** + standard token rates — factor this into your budget

This v2 is grounded in the actual Managed Agents beta (launched April 8, 2026) and your established M4 Mini setup.

---

## 1. Architecture: What Each Layer Actually Does

```
YOUR OBSIDIAN VAULT (~/Documents/SOS-Vault)
       ↓  filesystem read/write
CLAUDE CODE (terminal, M4 Mini)
  — local agent, computer control, code execution
  — reads CLAUDE.md + vault Markdown as context
  — no Obsidian API needed; files are just files
       ↓  API calls
CLAUDE MANAGED AGENTS (cloud, platform.claude.com)
  — cloud-hosted agent harness
  — secure sandboxed container (Bash, file ops, web search, MCP)
  — long-running sessions (hours), stateful, async
  — sessions persist through disconnections
  — built-in prompt caching + compaction
       ↓  outputs
_agent-outputs/ folder → you review → vault integration
```

**The key principle:** Claude Code is your local co-pilot (fast, interactive, computer control). Managed Agents is your async overnight worker (deep tasks, parallel pipelines, production-grade output). Your vault Markdown files are the shared memory layer.

---

## 2. M4 Mini Security Context

Respecting your three-browser isolation policy and no-digital-API-storage rule:

- **API keys** → written on paper, entered at runtime via env vars, never stored in dotfiles
- **Chrome** — Claude.ai, Anthropic Console (platform.claude.com)
- **Brave** — crypto/wallet (TAO, BTC), Venice.ai
- **Firefox** — stock research, Telegram, X

For Claude Code and API key handling:

```bash
# Set at session start — never in .zshrc or .bashrc
export ANTHROPIC_API_KEY="your-key-here"

# Unset when done
unset ANTHROPIC_API_KEY
```

For Managed Agents sessions, the API key is passed in the request header, not stored in config files. Keep it consistent with your paper-key policy.

---

## 3. Obsidian Vault Setup (30 minutes, do once)

### Install Obsidian
Download from https://obsidian.md — free, local-first, Markdown files on your filesystem.

### Create the vault
```
~/Documents/SOS-Vault/
├── CLAUDE.md                    ← Claude Code reads this first (your KISS rule)
├── SOS-context.md               ← mission, tech stack, current status
├── SeisClaw-context.md          ← engine specifics, algos, API endpoints
├── SOS-hypotheses.md            ← ideas with confidence ratings [0–100]
├── TROJAN-TREMORS-context.md    ← LACMA A+T grant context, Joel Ferree notes
├── _agent-outputs/              ← Managed Agent deliverables land here ONLY
│   └── .gitignore               ← exclude from vault Git if desired
└── daily-notes/
    └── 2026-04-10.md
```

### CLAUDE.md — your master context file

This is what Claude Code reads at the start of every session. Keep it under 500 lines. Example structure:

```markdown
# SOS / SeisClaw — Project Context for Claude Code
Last updated: 2026-04-10

## Core artistic thesis
Earth as composer. Algorithms as translators. Human listener as meaning-completing agent.

## Active projects
- SeisClaw v4 (seisclaw.com) — production seismic sonification engine
  - Pure binary MiniSEED parsing (NO ObsPy)
  - Six-station IU fallback: ANMO→COLA→GUMO→HNR→MAJO→MIDW
  - EarthScope endpoint: service.earthscope.org (migrated March 2026)
- TROJAN TREMORS — LACMA A+T Lab 2026 grant (deadline April 22, Submittable)
- SOS browser instrument — 8-phase FDTD/granular implementation

## Tech stack
- Language: vanilla JS (browser), Python (seismic utils)
- Architecture: KISS always. No frameworks unless essential.
- Claude Code: --permission-mode plan (default)
- GitHub: strikeslip

## Rules for agents
- Outputs go to _agent-outputs/ ONLY
- Never modify core context .md files
- Flag any MiniSEED parser suggestions involving ObsPy
- No dollar amounts or position sizes in any output
```

### Community plugins to install
In Obsidian Settings → Community plugins → Browse:
- **Advanced URI** — enables external URL-based vault access (needed for Managed Agent webhook patterns)
- **Dataview** — query your notes like a database (useful for `/sos-ideas` equivalent)
- **Templater** — templated daily notes

---

## 4. Claude Code Setup on M4 Mini

### Install (if not already done)
```bash
npm install -g @anthropic-ai/claude-code
```
Node.js 18+ required. Confirm with `node --version`.

### Launch against your vault
```bash
cd ~/Documents/SOS-Vault
export ANTHROPIC_API_KEY="your-key-here"
claude --permission-mode plan
```

The `--permission-mode plan` flag is your default per CLAUDE.md conventions — Claude proposes actions, you approve before execution. Override to `auto` only when you've reviewed what it's about to do.

### Custom slash commands

These are CLAUDE.md sections that Claude Code loads as context, not literal CLI commands. Structure them as prompt templates in your vault:

**SOS-commands.md** (or embed in CLAUDE.md):

```markdown
## Slash Command: /sos-context
Load full SOS + SeisClaw state. Read: SOS-context.md, SeisClaw-context.md, 
most recent daily note. Summarize active tasks and open questions.

## Slash Command: /sos-today  
Read today's daily note + open GitHub issues + SOS-hypotheses.md.
Output: prioritized task list for this session. Max 7 items.

## Slash Command: /sos-trace [idea]
Search vault for all notes referencing [idea]. Show evolution timeline 
with dates. Identify unresolved threads.

## Slash Command: /sos-ideas
Scan full vault. Cross-reference SeisClaw-context.md against recent 
daily notes. Propose 5 concrete development directions with rationale. 
Output to _agent-outputs/ideas-YYYY-MM-DD.md.

## Slash Command: /sos-challenge
Identify the 3 most vulnerable assumptions in current SOS/SeisClaw 
architecture. Play devil's advocate. Be specific.

## Slash Command: /sos-close-day
Read today's daily note + _agent-outputs/ from today.
Extract: decisions made, open questions, hypotheses to update.
Draft tomorrow's task seed. Output to daily-notes/[tomorrow].md.
```

To invoke: type the command name in your Claude Code session. Claude reads the CLAUDE.md instruction and executes it.

---

## 5. Claude Managed Agents — Actual Setup

### Prerequisites
- Anthropic API key (you have this)
- API account at platform.claude.com (enable via Chrome per your security setup)
- Beta header required on all requests: `managed-agents-2026-04-01`

### Pricing reality check
Standard token rates + **$0.08 per session-hour** of active runtime. A 2-hour deep SeisClaw development session = ~$0.16 in session overhead plus tokens. Budget accordingly.

### The four concepts (from official docs)
| Concept | What it is for SOS/SeisClaw |
|---------|---------------------------|
| **Agent** | Model + system prompt + tools + your SOS context |
| **Environment** | Container with Python, Node.js, seismic libraries pre-installed |
| **Session** | One running task instance (e.g., "build Kamchatka synth page") |
| **Events** | Your messages in, agent outputs/tool calls out (streamed via SSE) |

### Quickstart — Python SDK

```python
import anthropic

client = anthropic.Anthropic()  # reads ANTHROPIC_API_KEY from env

# Step 1: Create your SOS agent (do this once, save the agent_id)
agent = client.beta.agents.create(
    model="claude-sonnet-4-20250514",
    name="sos-seisclaw-builder",
    system="""You are the SOS/SeisClaw development agent for SHOOK (D.V. Rogers).
    
Core rules:
- All outputs go to the _agent-outputs/ directory only
- Never modify core .md context files
- MiniSEED parsing: pure binary only, never ObsPy
- EarthScope endpoint: service.earthscope.org
- KISS principle always — no frameworks unless essential
- No dollar amounts or position sizes in any output

Active project context:
- SeisClaw v4: production seismic sonification engine
- SOS browser instrument: FDTD wave propagation + granular synthesis
- TROJAN TREMORS: LACMA A+T grant (deadline April 22, 2026)
""",
    betas=["managed-agents-2026-04-01"]
)
print(f"Agent ID: {agent.id}")  # save this

# Step 2: Create environment
environment = client.beta.environments.create(
    name="sos-dev-env",
    setup_commands=[
        "pip install numpy scipy",
        "npm install -g typescript"
    ],
    betas=["managed-agents-2026-04-01"]
)
print(f"Environment ID: {environment.id}")  # save this

# Step 3: Start a session with a task
session = client.beta.sessions.create(
    agent_id=agent.id,
    environment_id=environment.id,
    betas=["managed-agents-2026-04-01"]
)

# Step 4: Send a task event
event = client.beta.sessions.events.create(
    session_id=session.id,
    content="Read the SeisClaw FDTD implementation notes and generate "
            "Phase 2 code: particle velocity field renderer using Canvas "
            "ImageData + requestAnimationFrame. Output to "
            "_agent-outputs/fdtd-phase2-YYYY-MM-DD.js",
    betas=["managed-agents-2026-04-01"]
)

# Step 5: Stream responses
for event_chunk in client.beta.sessions.events.stream(
    session_id=session.id,
    betas=["managed-agents-2026-04-01"]
):
    print(event_chunk)
```

### Using Console (no code required)

1. Open Chrome → https://platform.claude.com/workspaces/default/agent-quickstart
2. Follow the quickstart UI — define agent, configure environment, launch session
3. Monitor via built-in session tracing (every tool call, decision, failure mode visible)

This is the KISS path for one-off tasks.

### Multi-agent coordination (research preview)

Not yet generally available. Request access at https://claude.com/form/claude-managed-agents if you want parallel SOS pipeline experimentation (e.g., simultaneous synth page generation for multiple seismic stations).

---

## 6. Connecting Vault to Managed Agents

The cleanest pattern: upload your context Markdown files as part of the session task prompt, or mount them in the environment.

### Pattern A — Inline context (simplest)

```python
with open("SeisClaw-context.md", "r") as f:
    seisclaw_context = f.read()

event = client.beta.sessions.events.create(
    session_id=session.id,
    content=f"""Here is the current SeisClaw context:

{seisclaw_context}

Task: Generate the next FDTD phase implementation...
""",
    betas=["managed-agents-2026-04-01"]
)
```

### Pattern B — Files API (for larger vaults)

```python
# Upload vault context files once
with open("SOS-context.md", "rb") as f:
    file = client.beta.files.upload(
        file=("SOS-context.md", f, "text/plain")
    )
sos_file_id = file.id  # reference in subsequent sessions
```

### Pattern C — Local automation script

Drop this in `~/Documents/SOS-Vault/scripts/launch-agent-session.py`:

```python
#!/usr/bin/env python3
"""
SHOOK SOS Agent Session Launcher
Usage: python launch-agent-session.py "your task description"
"""
import sys
import os
import anthropic
from datetime import date

AGENT_ID = "your-saved-agent-id"
ENV_ID = "your-saved-env-id"

def launch(task: str):
    client = anthropic.Anthropic()  # ANTHROPIC_API_KEY from env
    
    # Load current vault context
    context_files = ["SOS-context.md", "SeisClaw-context.md"]
    context = ""
    for f in context_files:
        try:
            with open(f) as fh:
                context += f"\n\n---\n{f}:\n{fh.read()}"
        except FileNotFoundError:
            pass
    
    session = client.beta.sessions.create(
        agent_id=AGENT_ID,
        environment_id=ENV_ID,
        betas=["managed-agents-2026-04-01"]
    )
    
    full_task = f"""Today: {date.today()}
Vault context:{context}

Task: {task}

Output to: _agent-outputs/{date.today()}-output.md
"""
    client.beta.sessions.events.create(
        session_id=session.id,
        content=full_task,
        betas=["managed-agents-2026-04-01"]
    )
    print(f"Session launched: {session.id}")
    print(f"Monitor: https://platform.claude.com/sessions/{session.id}")

if __name__ == "__main__":
    launch(" ".join(sys.argv[1:]))
```

Run with:
```bash
export ANTHROPIC_API_KEY="your-key-here"
python scripts/launch-agent-session.py "Generate granular synthesis module for M6+ waveform events"
unset ANTHROPIC_API_KEY
```

---

## 7. Daily Workflow

### Morning (5 min)
```
1. Open Obsidian → write daily note → link to SOS files
2. cd ~/Documents/SOS-Vault && export ANTHROPIC_API_KEY="..."
3. claude --permission-mode plan
4. Type: /sos-today
```

### Active session
Natural language to Claude Code:
- *"Build the FDTD Phase 3 color mapping — seismic amplitude → spectrum shift"*
- *"Review SeisClaw v4 MiniSEED parser against current EarthScope API response format"*
- *"Draft the pre-submission email to Joel Ferree for TROJAN TREMORS"*

### Delegating overnight tasks to Managed Agents
```bash
python scripts/launch-agent-session.py \
  "Read SeisClaw-context.md. Implement the granular synthesis engine \
   (SOS Phase 5) following the 8-phase plan. Pure JS, no frameworks. \
   Output complete annotated code to _agent-outputs/"
```

Then close your terminal. The session runs in the cloud. Check Console in the morning.

### End of day (5 min)
```
/sos-close-day
unset ANTHROPIC_API_KEY
```

Review `_agent-outputs/` — integrate anything worth keeping into vault manually. **You write the vault. Agents write to _agent-outputs/ only.** This is non-negotiable for maintaining vault integrity.

---

## 8. SOS/SeisClaw-Specific Agent Tasks (Ready to Run)

### Generate a new synth station page
```
Task: Using the ANMO station as template, generate a new SeisClaw 
synth page for COLA (College, Alaska). Include: MiniSEED endpoint 
config, surface/deep theme toggle, granular synthesis parameter 
defaults tuned for subduction zone seismicity. 
Pure JS. Output: _agent-outputs/cola-synth-page-[date].html
```

### TROJAN TREMORS signal chain doc update
```
Task: Read TROJAN-TREMORS-context.md. Update the technical SHM 
reference document to reflect the OasisPlus/Kinemetrics platform 
integration. Focus on: 56 triple friction pendulum isolators, 
Dante network routing, Holosonics parametric speaker array. 
Output: _agent-outputs/SHM-technical-ref-v2-[date].md
```

### SOS browser instrument — FDTD phase continuation
```
Task: Read SeisClaw-context.md FDTD section. Continue Phase [N] 
of the 8-phase implementation. 400x300 grid, vanilla JS, 
ArrayBuffer/Canvas ImageData/rAF architecture 
(ref: jtiscione/webassembly-wave pattern).
Output: _agent-outputs/fdtd-phase[N]-[date].js
```

---

## 9. What's Real vs. What's Speculative

| Feature | Status | Notes |
|---------|--------|-------|
| Managed Agents API | **Live** (public beta, April 8 2026) | All API accounts |
| Long-running sessions | **Live** | Hours, persists through disconnection |
| Multi-agent coordination | **Research preview** | Request access separately |
| Outcomes / self-evaluation | **Research preview** | Request access separately |
| Claude Code `--permission-mode plan` | **Live** | Your established default |
| Obsidian CLI (as described in v1) | **Does not exist** | Advanced URI plugin is the real equivalent |
| `claude agent deploy` CLI | **Does not exist** | Sessions created via API or Console |

---

## 10. Quick-Start Checklist

```
□ Obsidian vault created at ~/Documents/SOS-Vault
□ CLAUDE.md written with SOS/SeisClaw context
□ SOS-context.md + SeisClaw-context.md populated  
□ _agent-outputs/ directory created
□ SOS-commands.md built out
□ Claude Code running: cd vault && claude --permission-mode plan
□ API key: export at session start, unset when done (paper-key policy)
□ Anthropic Console account active (Chrome browser)
□ First Managed Agent created via Console quickstart
□ launch-agent-session.py script installed in vault/scripts/
□ First overnight session launched
□ Morning review: _agent-outputs/ → curate → integrate to vault
```

---

## 11. Further Reading

- Managed Agents docs: https://platform.claude.com/docs/en/managed-agents/overview
- Console quickstart: https://platform.claude.com/workspaces/default/agent-quickstart
- Multi-agent access request: https://claude.com/form/claude-managed-agents
- Claude Code docs: https://docs.claude.ai/claude-code
- Pricing (session-hours): https://platform.claude.com/docs/en/about-claude/pricing#claude-managed-agents-pricing

---

*Save as `SOS-Obsidian-Claude-Workflow-v2.md` in vault root. Link from CLAUDE.md.*

*Earth is still composing. The agents work while you sleep. — SHOOK*
