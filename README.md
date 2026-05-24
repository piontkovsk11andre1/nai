# 🌿 Nai

### Naive Agentic Workflow

A self-installing, agent-driven delivery workflow that scaffolds itself from a
single specification prompt — no CLI to install, no package to download. You
hand the spec to any AI coding agent and it builds the workflow on your
machine, adapted to your language, OS, scripting language, and AI harness.

---

## 🤔 What is Nai?

Nai (from *naive*) is a deliberately minimal agentic workflow. Instead of
shipping binaries or templates, it ships **one prompt**
([INSTALL.md](INSTALL.md)) that fully specifies how to scaffold the workflow.
Any reasonably capable AI agent can read that prompt and produce a complete,
working installation on disk.

The resulting workflow gives you:

- A **single top-level launcher** that opens an AI agent in the right mode
  (first run: installer; subsequent runs: workspace orchestrator).
- **Workspaces** — isolated, git-worktree-backed task contexts copied from
  a template, each with its own prompts, artifacts, and per-agent launchers.
- A **markdown-backed work queue** (`Next` → `Current` → `Done`, with
  `Blocked` for failures) where each task is a chunk separated by `---`.
- A **dispatcher + worker wrapper** indirection so any AI harness
  (`opencode`, `claude`, etc.) can be plugged in without touching the rest
  of the workflow.
- **Safe rollback**: every completed task carries a per-repository commit
  hash in its header so `Work - Undo` can hard-reset the worktrees.
- **Archive-only removal**: workspaces are never deleted, only moved to
  `Workspaces/__archive__/`.

---

## ⚡ Install

Open any AI coding agent (Claude Code, opencode, Codex, Copilot Chat, etc.)
in an empty directory and send it this message:

> Please read <https://raw.githubusercontent.com/piontkovsk11andre1/nai/main/INSTALL.md>
> and implement the workflow.

That's it. The agent will:

1. Ask which natural language to use.
2. Ask about your OS, scripting language, install path, and AI harness.
3. Scaffold every file from the spec into the chosen directory.
4. Verify the scaffold and hand control back to you.

Your next action is to open the generated top-level launcher
(`Open Agent.lnk` / `Open Agent.command` / `Open Agent.desktop`). It will
spawn the **Installation Agent**, which finalizes git repositories, naming
conventions, framework mapping, and external issue-tracker wiring — then
flips the same launcher to open the **Workspace Agent** from then on.

---

## 🔁 Lifecycle

1. **First launch** → top-level launcher opens the *Installation Agent*.
   It walks you through repo initialization, conventions, and verifies the
   install. As its last step it rewrites the launcher to point at the
   *Workspace Agent*.
2. **Every launch after that** → top-level launcher opens the *Workspace
   Agent*. From there you create workspaces, archive workspaces, or open
   per-workspace agents.
3. **Inside a workspace** → five numbered launchers open the per-workspace
   agents: Git, Research, Planner, Worker, Reviewer. The Worker drives the
   markdown work queue via `Work - Do` and `Work - Undo`.
4. **Wrap up** → Reviewer produces `Changelog.md` and `PR.md`; Git Agent
   syncs them into the global `Workspaces/Backlog.md` /
   `Workspaces/Changelog.md`; `Workspace - Remove --synced` archives the
   workspace under `Workspaces/__archive__/`.

---

## 🧭 Dispatcher contract

Everything flows through a single dispatcher script (`Scripts/Agent.<ext>`):

| Flag | Purpose |
|------|---------|
| `--worker <name>` | Worker wrapper under `Scripts/Workers/` (default `Default`). |
| `--prompt <path>` | Prompt file the agent should read (required). |
| `--workspace <path>` | Working directory for the harness subprocess. |
| `--mode cli\|tui` | Unattended vs interactive (default `cli`). |
| `--tail <text>` | Extra user instruction appended to the bootstrap. |
| `--agent-name <text>` | Display label, metadata only. |
| `--context-files <csv>` | Hint list of files to attach if the harness supports it. |
| `--dry-run` | Print the harness command instead of running it. |
| `--new-window` | Windows TUI hint: spawn detached console and return. |

---

## 📋 Work queue

Inside every workspace, `Work/` holds exactly four files:

- `Next.md` — chunks waiting to be executed.
- `Current.md` — the chunk currently in flight (carries the rollback header).
- `Blocked.md` — chunks the verifier rejected (carries a blocked-reason header).
- `Done.md` — finished chunks (rollback header preserved for `Work - Undo`).

Chunks are separated by a single line containing `---`. Empty chunks are
ignored. `PASS` and `FAIL: <one-line reason>` are the **only** literal
ASCII tokens the verifier may emit on its last non-empty line.

---

## 🛡️ Safety notes

- **No remote push, ever**, unless you explicitly ask for it. Pulling is
  fine; pushing is your call.
- **Workspaces are archived, never deleted.** `Workspace - Remove` refuses
  to run without `--synced`, and it moves files into
  `Workspaces/__archive__/`.
- **`Work - Undo` is destructive by design** — it runs `git reset --hard`
  and `git clean -fdx` on the affected repos. Dirty work trees are erased.
- **Scripts are non-interactive.** Missing arguments cause a clean
  non-zero exit with a message naming the exact flag to add.

---

## 🧩 Scripts

| Script | One-line purpose |
|--------|------------------|
| `Scripts/Agent.<ext>` | Dispatcher — resolves a worker and forwards args. |
| `Scripts/Workers/Default.<ext>` | Worker wrapper for the configured AI harness. |
| `Scripts/Workspace - Create.<ext>` | Copy template, attach git worktrees, emit per-agent launchers. |
| `Scripts/Workspace - Remove.<ext>` | Archive a workspace under `__archive__/`. |
| `Scripts/Work - Do.<ext>` | Drive the `Next` → `Current` → `Done` / `Blocked` state machine. |
| `Scripts/Work - Undo.<ext>` | Roll back the last N done chunks via captured commit hashes. |
| `Scripts/WorkflowLog.<ext>` | Shared best-effort UTF-8 log helper. |

---

## 🎨 Adaptation

Everything is adjustable at install time:

- **Natural language** — every generated artifact, comment, and printed
  message is translated. Protocol literals (`PASS`, `FAIL:`, `---`, flag
  names, header markers) stay ASCII.
- **OS** — Windows (`.lnk`), macOS (`.command`), Linux (`.desktop`).
- **Scripting language** — Python (default, stdlib only), Bash, Node.js, Go.
- **AI harness** — `opencode` by default; the installer derives invocation
  patterns from `<harness> --help` and writes a working wrapper.

The full specification — invariants, adaptation surfaces, per-file specs,
per-script behavior, launcher recipes, harness guide, verification
checklist — lives in [INSTALL.md](INSTALL.md). That file is the single
source of truth; this README only orients newcomers.

---

## 📄 License

See [LICENSE](LICENSE) if present in this repository.
