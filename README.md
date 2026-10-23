# Antigravity CLI Helpers (`agy-cli`)

> [!IMPORTANT]  
> This project is not provided by Google and does not come with any support or warranties. This project serves as a proof of concept for how it's possible to customize and extend the Antigravity CLI (`agy`).  
> Though functional, you may wish to change the design for increased reliability or feature detail.

> [!IMPORTANT]  
> Accurate as of 10/23/2026

**What this is**
A collection of helper scripts and customizations for the Antigravity CLI (`agy`), starting with a responsive, 16-color ANSI status line hook (`scripts/statusline.sh`) that renders live context window utilization, input/output token counts, model display name, Git branch/dirty state, sandbox mode, and active subagent/background task counts directly in the terminal footer.

**Why this exists**
During long multi-turn coding or debugging sessions, it is easy to lose track of how much of the model's context window has been consumed or how many input and output tokens have accumulated before context compaction triggers. Antigravity CLI supports a configurable `statusLine` command hook that receives session state as JSON over `stdin` on every render tick. This script parses that payload in a single `jq` pass and adapts its layout automatically to wide (`>= 180` cols), medium (`>= 90` cols), and compact (`< 90` cols) terminals.

**What this is not**
- Not an official Google or Antigravity product.
- Not a billing or organization-wide quota tracker; token counts reflect the active conversation's context window payload passed by the CLI over `stdin`.

**How it works**
- bash + `jq` (single-pass JSON extraction from `stdin` with safe fallback defaults)
- standard 16-color ANSI escape sequences (compatible across macOS Terminal, iTerm2, Ghostty, kitty, and tmux without custom Nerd Font requirements)
- direct `git -C` inspection for fast working-tree branch and dirty-state (`*`) detection
- 20-segment progressive block bar (`█▓▒░`) color-coded by context utilization (`< 60%` yellow, `60–89%` bright yellow, `>= 90%` bright red)
- responsive 3-tier terminal width layout (`>= 180` single-row split-aligned with full `in`/`out` token breakdown; `90–179` two-line boxed layout; `< 90` compact two-line summary)

---

## Quickstart

### 1. Prerequisites

Ensure `jq` and `git` are installed on your workstation:

```bash
jq --version
git --version
```

### 2. Install the Script

Copy `scripts/statusline.sh` into your Antigravity CLI configuration directory and make it executable:

```bash
mkdir -p ~/.gemini/antigravity-cli
cp scripts/statusline.sh ~/.gemini/antigravity-cli/statusline.sh
chmod +x ~/.gemini/antigravity-cli/statusline.sh
```

### 3. Configure `settings.json`

Add (or update) the `statusLine` block in `~/.gemini/antigravity-cli/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.gemini/antigravity-cli/statusline.sh",
    "enabled": true
  }
}
```

### 4. Test Locally

Pipe a sample payload into the script to preview the wide, medium, and compact layouts:

```bash
jq -n '{
  agent_state: "idle",
  model: { display_name: "Gemini 2.5 Pro" },
  cwd: env.PWD,
  conversation_id: "345454bb-a2c5-4037-8b3d-11b7ca98dad5",
  terminal_width: 190,
  sandbox: { enabled: true, allow_network: false },
  artifact_count: 2,
  subagents: ["scout"],
  task_count: 1,
  context_window: {
    used_percentage: 42.5,
    total_input_tokens: 340000,
    total_output_tokens: 85000,
    context_window_size: 1000000
  }
}' | ~/.gemini/antigravity-cli/statusline.sh
```

---

## License

Licensed under the [Apache License, Version 2.0](LICENSE).
