# duet

One shared conversation between you, Claude Code and Codex. Built for open-ended work such as financial modeling, where there is no correct answer and you want two equally competent analysts in the room with you.

```
cd ~/models/acme-deal
duet
```

That opens the room for that directory and drops you into a prompt, like running `claude` or `codex` there. The first thing you type is the brief; both analysts answer it. From then on:

```
you> @codex rebuild the churn assumption from data/cohorts.csv and put it in retention.py
you> @claude check Codex's retention numbers against the raw file, independently
you> /ask what discount rate would you use, and why?
you> going with 12%. both of you: update the model and give me the EV range
you> /go 4
you> /note board wants a downside case too, park it for tomorrow
```

| input | effect |
|---|---|
| `<text>` | to both; each replies in turn, the second seeing the first's reply |
| `@claude <text>`, `@codex <text>` | task or question for one analyst |
| `/ask <text>` | both answer independently from the same snapshot, then both answers land. Use for estimates, so the second cannot anchor on the first |
| `/go [N]` | they work with each other for N turns (default 2). Either can stop early with `STATUS: WAITING` when a decision is yours |
| `/note <text>` | append to the log with no reply (context, decisions, pasted data) |
| Send now (web) | while an analyst is working, Enter queues your message behind the current work; Send now or ⌘Enter cuts the running turn off, drops anything queued, and delivers immediately. The log records which turn was interrupted. In the terminal, Ctrl-C does the cutting off |
| `/writes on\|off` | whether analysts may change files here |
| `/model claude opus`, `/model codex <name>` | switch a model for this room; persists. `/model` shows, `default` resets |
| `/new` | archive this conversation and start a fresh one in this directory |
| `/status`, `/log`, `/quit` | |

Ctrl-C during a turn aborts that turn only. Ctrl-D quits. Run `duet` again in the same directory to pick up where you left off.

## Browser UI

```
cd ~/models/acme-deal
duet --web
```

Starts a local server, opens your browser at http://localhost:8787 (`--no-open` to skip), and serves the same room in a claude.ai-style layout: a sidebar listing this directory's current and archived conversations (archived ones open read-only) with model, first-replier and may-edit-files settings; a reading column with your messages in bubbles and the analysts' replies as rendered markdown with tables and code; and a composer with a target picker (Both, Claude, Codex, Ask blind, Note), a "let them work" control, and abort while someone is working. Hover any entry to edit it in place or copy it. Light and dark follow the system setting.

**Show work.** While an analyst works, its steps stream in under its name: reasoning summaries, each tool call (the command run, the file read or edited, the search made) and each result. Afterwards every reply has a "Show work · N steps" toggle, and archived conversations keep theirs. Steps from subagents are indented and marked sub. Analyst turns run with `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`, because a headless turn ends when the agent replies and nothing can deliver background results afterwards; delegation still works, in the foreground, inside the turn. This is the activity each CLI exposes in its JSON stream, not the model's hidden reasoning; Claude shows thinking blocks when it emits them, Codex shows reasoning summaries. Steps are stored per turn in `.duet/turns/NNN-<agent>.events.jsonl`, and the terminal prints tool calls and reasoning as they happen.

**Models.** The sidebar dropdowns list Claude's aliases (fable, opus, sonnet, haiku) plus your configured default, and Codex's live catalog from `codex app-server` with a reasoning-effort picker (low to ultra). Both analysts get a reasoning-effort picker: Claude's `--effort` (low, medium, high, xhigh, max) and Codex's `model_reasoning_effort` (low to ultra, per model). Custom… accepts any id the CLI takes. In the terminal: `/models`, `/model codex gpt-6-astra`, `/effort claude high`.

**Settings.** The ⚙︎ Settings view in the sidebar holds your personalization: standing instructions added to every analyst prompt, for both Claude and Codex. Two scopes: everywhere (`~/.duet/personalization.md`, every room on this machine) and this project (`.duet/personalization.md`). They are read fresh each turn, so a save applies from the next reply. In the terminal, `/persona` shows what is in effect and where to edit it. Conversations are named automatically: when the brief is posted, Claude (haiku, no tools) is asked for a short title in the background, stored as the first line of `chat.md` so archives keep theirs. Click the title in the web UI to rename, or `/title <text>` in the terminal.

Claude also still loads your `~/.claude/CLAUDE.md` and Codex its `~/.codex/AGENTS.md`, since each runs as its own CLI; duet's personalization is the layer both share. The terminal and the browser share the same `.duet/chat.md`, so you can use either. Add `--host 0.0.0.0` to reach it from another device on your network; there is no authentication, so only do that on a network you trust. `--port` changes the port.

Billing is unchanged in the browser: the server still shells out to `claude -p` and `codex exec` on this machine, so both use the subscriptions. Markdown rendering is vendored in `web/` (marked, DOMPurify); nothing is loaded from the internet.

## How it works

The room lives in `.duet/` inside the project directory. One file matters: `chat.md`, a plain markdown log with numbered entries from Human, Claude and Codex.

Every analyst turn is a fresh headless CLI call whose prompt is a short framing, the entire `chat.md`, and a one-paragraph instruction for this turn. The reply is appended to `chat.md`. There are no hidden per-agent sessions, so both analysts always have the same single context, and the log on disk is the whole state. Edit `chat.md` in another window between turns (paste data, strike a wrong number, record a decision) and the next turn sees it.

Both analysts work in the project directory and both may change files there unless you turn writes off. Turns are sequential, so they never write at the same time.

Cost scales with log length because the whole log is resent each turn. Claude's prompt caching makes the unchanged prefix cheap; `/status` shows the approximate size. When a room gets long, `/new` and paste a summary as the new brief.

No API keys. Both CLIs use whatever they are logged in as (claude.ai Max and ChatGPT here). `ANTHROPIC_API_KEY` and `OPENAI_API_KEY` are stripped from the child environment so nothing routes to API billing. Claude's `total_cost_usd` is reported as a list-price estimate only.

## Flags

- `--first codex` who replies first when both are addressed (default claude).
- `--read-only` analysts may not change files. Toggle later with `/writes`.
- `--commit` git-commit the directory after every analyst turn, so each turn is a diff. Add `.duet/` to `.gitignore` if you do not want the log in the repo.
- `--claude-model opus|sonnet|fable`, `--codex-model <name>`. Codex otherwise uses `~/.codex/config.toml`.
- `--claude-allow 'Bash(pytest:*)'` extra Claude tool patterns that never need approval.
- `--yolo` no permission checks or sandbox at all. Only in a directory you can afford to lose.
- `--cwd DIR` use another directory. `--log` print the log and exit. `--quiet` headers only.
- `--web`, `--port 8787`, `--host 127.0.0.1` browser UI, see above.

## Permissions

In the browser, when Claude wants to do something its permission layer would normally ask about (a bulk move, a command outside the project, anything the automatic classifier won't approve), a card appears above the message box with the exact command and three choices: Allow, Allow everything this turn, Deny. The turn waits for your answer, up to 30 minutes, then is denied. Settings has the policy: Ask me (default), Decide automatically (safe actions allowed, the rest denied without asking), or No checks. Codex is not asked; its OS sandbox is the boundary. In the terminal there is nobody to ask, so Ask me behaves as Decide automatically.

Mechanically, duet registers itself as Claude Code's permission prompt tool over MCP; the tool call blocks until you click.


| | Claude | Codex |
|---|---|---|
| writes on | `--permission-mode auto` (classifier approves safe actions) | `sandbox_mode="workspace-write"` |
| writes off | `auto` with `Edit/Write/MultiEdit/NotebookEdit` removed | `sandbox_mode="read-only"` (OS enforced) |
| `--yolo` | `bypassPermissions` | `--dangerously-bypass-approvals-and-sandbox` |

Claude's writes-off mode is advisory: edit tools are gone and the prompt says not to change files, but Bash remains so it can run things, and Bash can write. Codex's read-only sandbox is enforced by the OS. Anything that would prompt in a headless Claude run is denied, and denied tools are printed after the turn.

## Install

`duet` is a single Python 3.11+ script with no dependencies; it needs `claude` and `codex` on PATH and logged in. It is symlinked at `~/.local/bin/duet`. Prompts live in `prompts/` next to the script (`system.md` is the framing; `reply.md`, `ask.md`, `go.md` are per-turn instructions; plain `string.Template`). `DUET_FAKE=1 duet` uses canned replies to exercise the harness for free. Each turn's exact prompt, CLI invocation and raw output are under `.duet/turns/`.

`examples/saas-memo.md` is a real log from a test brief: both analysts built a 3-year ARR model, disagreed on the spend ramp with reasons, and one of them, when asked alone, isolated a single assumption change without touching the file.
