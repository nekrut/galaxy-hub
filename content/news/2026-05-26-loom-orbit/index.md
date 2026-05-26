<!-- TODO: add screenshots to this directory -->
---
title: "Loom & Orbit: an AI research assistant for Galaxy bioinformatics"
date: "2026-05-26"
tease: "Loom turns a Galaxy working directory into a durable, git-tracked co-scientist project. Orbit is the Electron desktop shell that wraps it."
tags: [AI, LLM, Galaxy, tools, bioinformatics, release]
subsites: [all, global]
contributions:
  authorship:
    - nekrut
---

# Loom & Orbit

**Loom** is an AI research harness for [Galaxy](https://galaxyproject.org) bioinformatics. You open it in an analysis directory, describe your experiment in plain language, and Loom drafts a step-by-step plan that routes heavy compute to Galaxy and lightweight tasks to your local machine. Plans, decisions, results, and interpretations accumulate in a single `notebook.md` that is versioned with git—your durable, reproducible project log.

**Orbit** is the Electron desktop shell for Loom: a three-pane window (file tree, chat, tabbed artifact pane) that wraps the same brain you can also run directly from the terminal with `loom`.

![Orbit welcome screen showing provider setup](./orbit-welcome.png)

---

## Installation

### Desktop app (Orbit)

Download the latest installer from the [Releases page](https://github.com/galaxyproject/loom/releases).

| Platform | File | Notes |
|----------|------|-------|
| macOS (Apple Silicon) | `Orbit-<version>-arm64.dmg` | M1/M2/M3/M4, late 2020 onward |
| macOS (Intel) | `Orbit-<version>-x64.dmg` | Intel Macs |
| Linux (Debian/Ubuntu) | `orbit_<version>_amd64.deb` | also Linux Mint, Pop!_OS |
| Linux (Fedora/RHEL) | `orbit-<version>.x86_64.rpm` | also CentOS, openSUSE |
| Linux (any distro) | `Orbit-linux-x64-<version>.zip` | extract and run the `orbit` binary |
| Windows | via WSL2 (see below) | native builds not yet available |

Not sure which macOS build to pick? Open Apple menu → About This Mac. "Chip: Apple M..." → arm64. "Processor: Intel..." → x64.

**macOS install steps:**

1. Download the matching `.dmg`.
2. Double-click the DMG, drag **Orbit** to the **Applications** folder.
3. Eject the DMG.

**Gatekeeper workaround (unsigned alpha builds):** The first time you launch Orbit, macOS will block it with "Orbit can't be opened because Apple cannot check it for malicious software." To work around this:

1. Open **Applications** in Finder.
2. Right-click (or Control-click) **Orbit** → choose **Open**.
3. Click **Open** in the dialog.

macOS remembers the decision; subsequent launches work normally. If the **Open** option does not appear in the right-click menu, run:

```bash
xattr -dr com.apple.quarantine /Applications/Orbit.app
```

**Linux .deb:**

```bash
sudo dpkg -i orbit_<version>_amd64.deb
sudo apt-get install -f
```

**Linux .rpm:**

```bash
sudo rpm -i orbit-<version>.x86_64.rpm
```

**Windows via WSL2:** Install WSL2 with Ubuntu (`wsl --install` from an elevated PowerShell), then inside the Ubuntu terminal install the `.deb` package as above. WSLg (bundled with Windows 11 WSL2) provides native GUI support—no X server setup required. Keep your analysis data inside `~/` rather than `/mnt/c/` for best I/O performance.

### Loom CLI (no desktop)

Run the Loom brain directly from the terminal without Orbit. Requires Node 20+ and [`uv`](https://docs.astral.sh/uv/).

```bash
npm install -g @galaxyproject/loom
loom
```

Or without installing:

```bash
npx @galaxyproject/loom
```

---

## Getting API keys

Loom needs at least one LLM provider key. Galaxy is optional but enables routing heavy steps to the cluster.

### Anthropic (Claude)

1. Go to [console.anthropic.com](https://console.anthropic.com/) and create an account.
2. Navigate to **API keys** → **Create key**.
3. Copy the key (starts with `sk-ant-`).

### OpenAI (GPT-4o, o3, etc.)

1. Go to [platform.openai.com](https://platform.openai.com/) and create an account.
2. Navigate to **API keys** → **Create new secret key**.
3. Copy the key (starts with `sk-`).

### Google Gemini

1. Go to [aistudio.google.com](https://aistudio.google.com/) and sign in with a Google account.
2. Click **Get API key** → **Create API key**.
3. Copy the key.

### Galaxy API key

1. Log in to your Galaxy server (e.g., [usegalaxy.org](https://usegalaxy.org)).
2. Click your username in the top-right → **User** → **Preferences** → **Manage API Key**.
3. Click **Create a new key** if none exists, then copy it.

---

## Entering API keys in Orbit

Open Preferences with **Cmd+,** (macOS) or **Ctrl+,** (Linux/Windows). In the Preferences modal:

1. Select your LLM provider from the **Provider** dropdown (Anthropic, OpenAI, Google Gemini, or a local provider such as Ollama or LiteLLM).
2. Paste your API key into the **API key** field.
3. Optionally enter your Galaxy server URL and Galaxy API key in the **Galaxy** section.
4. Click **Save**.

Orbit stores LLM and Galaxy API keys encrypted via the operating system keychain (macOS Keychain, Linux Secret Service). On first launch, the welcome screen walks you through the same setup and can be skipped—configure later from Preferences whenever you are ready.

![Orbit Preferences modal showing provider, API key, and Galaxy fields](./orbit-preferences.png)

---

## Interface overview

Orbit has a three-pane layout:

| Pane | Location | What it shows |
|------|----------|---------------|
| **Files** | Left | File tree for the current working directory. Click any file to open it in the File tab. |
| **Chat** | Center | Streaming conversation with the Loom agent. Markdown-rendered responses with proper tables and code blocks. |
| **Artifacts** | Right | Tabbed: **Notebook**, **Activity**, **File** |

At narrow widths (below 900 px the file tree collapses, below 700 px the artifact pane collapses) so the chat stays usable. Toolbar buttons re-expand them.

The **footer** shows:

- A **Galaxy connection dot**: green when connected with a valid API key, red otherwise. Click it to open Preferences.
- A **cost/token counter**: live in-flight cost for the current session, computed from per-event usage data reported by the LLM provider.

![Orbit three-pane layout: Files left, Chat center, Artifacts right](./orbit-layout.png)

### Keyboard shortcuts

| Shortcut | Action |
|----------|--------|
| `Cmd/Ctrl+,` | Open Preferences |
| `Cmd/Ctrl+\` | Toggle the artifact pane |
| `Cmd/Ctrl+B` | Toggle the file tree |
| `Cmd/Ctrl+O` | Switch working directory (starts a fresh session for that directory) |
| `↑` / `↓` in chat input | Recall previous prompts (per-directory, persistent) |
| `/` in chat input | Open the slash-command autocomplete popup |
| `Tab` | Accept the highlighted slash command |
| `Esc` | Dismiss modals; close slash popup; cancel a streaming response |

---

## Slash commands

Type `/` in the chat input to open the autocomplete popup. Tab completes the highlighted command; Enter still submits whatever is in the input.

| Command | What it does |
|---------|-------------|
| `/model <name>` | Switch the LLM model, e.g. `/model sonnet`, `/model claude-opus-4-6` |
| `/new` | Start a fresh session (confirms before clearing the existing notebook) |
| `/resume` | Restart the agent and replay the prior session's chat |
| `/chat` | Restore the chat pane from the session transcript without restarting the agent |
| `/plan` | Show current plan summary |
| `/status` | Show Galaxy connection status and notebook path |
| `/notebook` | Show notebook info in the Notebook tab |
| `/summarize [N [M]]` | Append a summary of prompts N through M into the notebook |
| `/cost` | Append the session token/cost breakdown to the notebook |
| `/decisions` | Show the decision log |
| `/connect [name]` | Open Galaxy connection settings, or switch to an existing profile |
| `/help` | Show the full slash-command list |

---

## The Notebook

`notebook.md` in your working directory is the project log. Everything accumulates there: ad-hoc exploration notes, drafted and approved plans, executed steps, results, interpretations, and follow-up plans. The agent reads and writes it directly—there is no separate structured state store.

When Loom starts in a directory that is not yet a git repo, it runs `git init`, adds a bioinformatics-friendly `.gitignore` (excluding FASTQ, BAM, VCF, and other large files), and enables auto-commit on every notebook write. This gives you full undo history and a timestamped, reproducible record of every decision. If you start Loom in an existing git repo, auto-commit stays off by default; opt in with `git config loom.managed true`.

View the live notebook in the **Notebook** tab of the artifact pane. It auto-refreshes whenever the file changes on disk.

Plans in the notebook look like this:

```markdown
## Plan A: chrM Variant Calling [hybrid]

Goal: identify variants in chrM across 4 paired-end samples.

### Steps

- [ ] 1. **QC FASTQ** — fastp adapter trim + per-base QC
  - Routing: local
- [x] 2. **Reference index** — bwa index of chrM
  - Routing: local
- [ ] 3. **Read alignment** — BWA-MEM, paired collection
  - Routing: Galaxy
  - Tool: bwa-mem2/2.2.1
```

The agent drafts each plan in chat first and waits for your approval before writing anything to `notebook.md`. After plan approval, it shows the parameter table and waits again. Only after both gates pass does it write the plan and begin execution.

---

## Activity tab

The **Activity** tab in the artifact pane is split horizontally:

- **Top half**: the agent shell stream—live tool calls, status messages, and stdout from any subprocess the agent launches.
- **Bottom half**: a process monitor showing live CPU usage, memory, and runtime for every subprocess the agent has spawned in the current session.

This lets you watch a Galaxy job or a local alignment command progress in real time without switching windows.

---

## Galaxy connection

When a Galaxy API key is configured, Loom registers the Galaxy MCP server on startup. The **green/red dot** in the Orbit footer reflects this connection:

- **Green**: Galaxy is reachable and the API key is valid.
- **Red**: No API key is configured, or the server is unreachable. Click the dot to open Preferences.

Use `/connect` in the chat to add or switch Galaxy server profiles interactively. Credentials are saved to `~/.loom/config.json` automatically (encrypted in Orbit; plaintext with `0600` permissions in the CLI).

Once Galaxy is connected, the agent surveys the [Intergalactic Workflow Commission (IWC)](https://iwc.galaxyproject.org) workflow registry and the tool catalog when drafting a plan. Steps that match IWC workflows are proposed as Galaxy invocations; heavy compute steps without a full workflow match are routed to Galaxy tool-by-tool if the tool is installed; lightweight tasks (parsing, small scripts) run locally. The three operating modes—**local**, **hybrid**, and **remote**—are an outcome of the plan, not a configuration setting.

---

## Skills system

Loom can fetch operational know-how from curated GitHub repositories. The default, [`galaxyproject/galaxy-skills`](https://github.com/galaxyproject/galaxy-skills), is seeded on first use and covers:

- Paired-collection construction from PE FASTQ
- Galaxy MCP usage patterns and gotchas
- Nextflow → Galaxy workflow conversion
- Galaxy tool development
- Workflow report templates
- ToolShed tool revision updates

Add additional skill repositories in **Preferences → Skills**. During the alpha, the URL allowlist is restricted to `https://github.com/galaxyproject/*` to prevent prompt-injection via untrusted SKILL.md content.

---

## Cost tracking

Orbit's footer shows live in-flight cost for the active session. For historical, per-project, and per-model reporting across all sessions, run [CodeBurn](https://github.com/getagentseal/codeburn) in a separate terminal:

```bash
npx codeburn          # interactive TUI dashboard
npx codeburn today    # today's spend
npx codeburn month    # this month
```

CodeBurn auto-detects Loom sessions under `~/.pi/agent/sessions/` (Pi is a first-class supported provider—no configuration needed) and pulls model pricing from LiteLLM's catalog.

---

## Related resources

- [Loom/Orbit source and releases](https://github.com/galaxyproject/loom)
- [galaxy-mcp](https://github.com/galaxyproject/galaxy-mcp) — MCP server for the Galaxy API
- [galaxy-skills](https://github.com/galaxyproject/galaxy-skills) — curated operational skills
- [CodeBurn](https://github.com/getagentseal/codeburn) — AI-coding cost dashboard (Pi first-class)
- [Pi coding agent](https://github.com/badlogic/pi-mono) — the underlying agent framework
