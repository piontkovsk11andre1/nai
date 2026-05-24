# 🌿 Nai

### Naive Agentic Infrastructure

A self-installing, agent-driven delivery workflow. There is no CLI to
install and no package to download — you hand a single specification
prompt ([INSTALL.md](INSTALL.md)) to any capable AI coding agent and it
scaffolds the whole infrastructure on your machine, adapted to your
language, OS, scripting runtime, and AI harness.

This README describes **what you end up with after installation** and
**how to live with it day to day**. For the underlying spec, see
[INSTALL.md](INSTALL.md).

---

## ⚡ Install in one step

Open any AI coding agent (Claude Code, opencode, Codex, Copilot Chat, …)
in an empty directory and send it one message:

> Please read <https://raw.githubusercontent.com/piontkovsk11andre1/nai/main/INSTALL.md>
> and implement the workflow.

The agent will interview you (natural language, OS, scripting language,
install path, AI harness), scaffold every file, run a verification pass,
and stop. Your next action is to **double-click the top-level launcher**
it produced.

---

## 📦 What you get on disk

After scaffolding finishes, the install path looks roughly like this:

```
Open Agent.(lnk|command|desktop)     ← top-level launcher
Prompts/
  Installation Agent.md              ← runs on first launch
  Workspace Agent.md                 ← runs from second launch onward
Scripts/
  Agent.<ext>                        ← dispatcher
  Workspace - Create.<ext>
  Workspace - Remove.<ext>
  Work - Do.<ext>
  Work - Undo.<ext>
  WorkflowLog.<ext>
  Workers/
    Default.<ext>                    ← wrapper around your AI harness
Workspaces/
  Backlog.md, Changelog.md
  __archive__/                       ← archived workspaces land here
  __template__/                      ← cloned for every new workspace
    Configuration/  Documentation/  Implementation/
    Workspace.md  Plan.md  Issue.md  Status.md  …
    Prompts/   (Git / Research / Planner / Worker / Reviewer)
    Work/      (Next / Current / Blocked / Done .md)
```

Concretely, you get:

- **One launcher** at the top that opens an AI agent in the right mode.
- **A `Default` worker wrapper** that already knows how to call your AI
  harness in both unattended (`cli`) and interactive (`tui`) modes.
- **A workspace template** with per-agent prompts, per-agent launchers,
  and a markdown-backed work queue.
- **A scripts layer** (`Workspace - Create / Remove`, `Work - Do / Undo`,
  dispatcher) — small, non-interactive, fail-loud on missing flags.

Everything is in your chosen natural language; only protocol literals
(`PASS`, `FAIL:`, `---`, CLI flag names, header markers) stay ASCII.

---

## 🚀 First launch vs. every launch after

The top-level launcher is the only thing you interact with from the
filesystem. Its behavior changes once:

1. **First launch** → opens the **Installation Agent**. It finalizes the
   parts that were intentionally deferred during scaffolding: template
   git repositories (and their initial commits), workspace naming and
   branch conventions, optional framework mapping, and external issue
   tracker wiring. As its last step it **rewrites the launcher itself**
   to point at the Workspace Agent.
2. **Every launch after** → opens the **Workspace Agent**, which is your
   day-to-day orchestrator: create a workspace, archive a workspace, or
   open a per-workspace agent.

The launcher file name never changes; only the prompt it targets does.

---

## 🧑‍💻 Day-to-day usage

### Create a workspace

Ask the Workspace Agent:

> Create a workspace for ticket `FOO-123` about the auth refactor.

It calls `Scripts/Workspace - Create` with a `--workspace` path and a
`--branch` name. The script:

- copies `Workspaces/__template__/` into the new workspace path,
- attaches each template repo as a **git worktree** on the requested
  branch (creating the branch if needed),
- drops five numbered launchers inside the workspace:
  `1. Open Git Agent`, `2. Open Research Agent`, `3. Open Planner
  Agent`, `4. Open Worker Agent`, `5. Open Reviewer Agent`.

### Work inside a workspace

Open the launcher for the role you need. The intended flow:

- **Research Agent** — asks for pre-context, then asks one question at a
  time, writes findings into `Research.md`.
- **Planner Agent** — reads `Issue` / `Research` / `Notes` / etc.,
  produces `Plan.md` and a `Status.md` table.
- **Worker Agent** — derives actionable chunks into `Work/Next.md` (one
  blank-line-separated chunk per task, `---` between them), then drives
  the work queue.
- **Reviewer Agent** — writes `Changelog.md` and `PR.md`.
- **Git Agent** — finalizes commits, syncs the workspace's
  `Backlog` / `Changelog` into the global files in `Workspaces/`, and
  handles explicit git operations on request.

### Run the work queue

The Worker Agent (or you, directly) invokes:

- `Scripts/Work - Do` — moves the next chunk through
  `Next` → `Current` → `Done` (or `Blocked` on failure). Before
  executing, it captures a `<!-- rollback: RepoA=<hash> RepoB=<hash> -->`
  header so the chunk can be safely rolled back later. The verifier's
  last non-empty line must be exactly `PASS` or `FAIL: <one-line
  reason>`. Anything else is treated as a blocked-with-reason chunk.
- `Scripts/Work - Undo --count N` — `git reset --hard` + `git clean
  -fdx` on every repo recorded in the rollback headers of the last N
  done chunks, then returns those chunks to the top of `Next`.

`Work - Do` and `Work - Undo` are deliberately **non-interactive**: a
missing decision flag (e.g. `--blocked-action`, `--current-action`,
`--count`) makes the script exit non-zero with a message naming the
exact flag to add.

### Archive a workspace

When the work has shipped:

```
Scripts/Workspace - Remove --workspace <path> --synced
```

`--synced` is mandatory — it forces you to run the Git Agent's
backlog/changelog sync first. The script then moves everything to
`Workspaces/__archive__/<same-path>/`, detaches the worktrees, and
prunes stale worktree records. **Nothing is ever deleted.**

---

## 🧭 Dispatcher contract

Every launcher and every script ultimately calls a single dispatcher
(`Scripts/Agent.<ext>`) with these flags:

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

The dispatcher does nothing else — it validates `--prompt`, resolves the
worker, and forwards. Swapping AI harnesses is just a matter of adding a
new file under `Scripts/Workers/`.

---

## 🛡️ Safety guarantees

- **No remote push, ever**, unless you explicitly ask for it.
- **Workspaces are archived, never deleted.** `Workspace - Remove`
  refuses to run without `--synced`.
- **`Work - Undo` is destructive by design** — `git reset --hard` plus
  `git clean -fdx`. Dirty work trees on rolled-back repos are erased.
  That is the price of the safety net.
- **Scripts are non-interactive.** Missing arguments fail with a clean
  non-zero exit and a message naming the exact flag.
- **UTF-8 end to end**, including the Windows console code page, so
  translated strings survive every subprocess hop.

---

## 🎨 What's adaptable

Decided at install time, never hard-coded:

- **Natural language** — every prompt, comment, and printed message is
  translated. Protocol literals stay ASCII.
- **OS** — Windows (`.lnk`), macOS (`.command`), Linux (`.desktop`).
- **Scripting language** — Python (default, stdlib only), Bash, Node.js,
  Go.
- **AI harness** — `opencode` by default; the installer derives
  invocation patterns from `<harness> --help` and writes a working
  wrapper. Additional wrappers can be added later under
  `Scripts/Workers/`.

The full specification — invariants, per-file specs, per-script
behavior, launcher recipes, harness guide, verification checklist —
lives in [INSTALL.md](INSTALL.md). That file is the single source of
truth; this README only orients you once the scaffolding is on disk.

---

## 📄 License

[MIT](LICENSE).
