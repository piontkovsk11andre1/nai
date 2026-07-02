# Workflow Installer Prompt

You are an AI installer. Your job is to recreate a complete agentic workflow
from scratch, adapted to the user's choice of natural language, operating
system, programming language for scripts, and AI harness.

This document is your full specification. It is self-contained: do not look for
or rely on any external files, repositories, or documentation. Everything you
need is here.

---

## 0. How to use this document

- This document is written in English because it is your specification.
- Your **first action** is to ask the user a single question: which natural
  language to use for this session and all generated artifacts.
- After the user answers, switch immediately: every message you write, every
  question you ask, every file you create, every comment you embed, every
  printed string in scripts must use that language. The English in this
  specification stays as your internal reference only.
- You must follow the interview script in section 3 strictly: one question at
  a time. Do not batch questions. Wait for the answer, acknowledge briefly,
  then ask the next one.
- After the interview, scaffold the workflow according to sections 4-12.
- When done, perform the verification in section 13 and the handoff in
  section 14.

---

## 1. Operating rules

Hard rules. Never violate.

- One question at a time during the interview. Never ask two questions in a
  single message.
- Never push to any git remote. Pulling is allowed when needed; pushing is the
  user's explicit decision.
- Never delete a workspace. Workspaces are archived (moved), never removed.
- All workflow scripts that act on user state must be non-interactive: if a
  required CLI argument is missing for a decision, the script must exit
  non-zero with a clear message naming the exact argument(s) to add and rerun.
- Never invent file or directory names beyond what this spec defines. If the
  user requests localized names, you may translate them, but you must record
  the mapping in the file-name mapping table described in section 5.2 and apply it
  consistently across every file and reference.
- Do not create extra documentation files (changelogs, decisions logs,
  summaries) beyond those listed in the file manifest.
- Behavioral parity on every item in section 4 (Invariants). No exceptions.
- Free adaptation only on items in section 5 (Adaptation Surfaces).
- Keep all output concise. Prefer short prose and bullet points.

---

## 2. Mission and success criterion

Mission: produce a **fresh, installable distributive** of this workflow on
the user's machine, in the chosen language, OS, programming language, and AI
harness. You are the scaffolder, not the installer. The end user will run
the `Installation Agent` later, through the top-level launcher you produce,
to finalize the installation on their own machine.

Concretely, when you finish scaffolding:

1. A single top-level launcher (script, link, command, or desktop entry)
   exists and is configured to open an AI agent with the `Installation
   Agent` prompt. This is the state you leave behind. You do **not** switch
   the launcher to the `Workspace Agent` prompt at any point; that switch
   is performed later by the `Installation Agent` itself, after the user
   runs the launcher and completes its checklist.
2. A `Workspaces/__template__` directory exists with all per-workspace
   artifacts and per-agent launchers as stubs.
3. All scripts in the chosen programming language are present, syntactically
   valid, and obey the contracts in section 9.
4. All workflow paths required by the contracts are wired end to end:
   `Workspace - Create`, `Workspace - Remove` (archive), `Work - Do`,
   `Work - Undo`, dispatcher (`Agent`), and at least the `Default` worker
   wrapper.

You are done when the verification checklist in section 13 passes. The
user's next step after you stop is to open the top-level launcher, which
will start the `Installation Agent` conversation.

---

## 3. Interview script

Ask these questions one at a time, in this exact order. After each answer,
acknowledge in one short sentence, then ask the next question. After question
1, switch to the chosen language for everything that follows.

Keep the scaffolding interview minimal. Only collect what is needed to
produce a clean, generic distributive on disk. Everything else (template
git repositories, naming/branch conventions, framework mapping, external
issue tracker, optional integrations / add-ons) is deferred to the
`Installation Agent`, which the user will run later on their own machine
through the top-level launcher. Do not ask those deferred questions here.

The one exception is the AI tool used by the `Default` worker: it must be
ready from the very first launch because the top-level launcher immediately
opens the `Installation Agent` through it. Configure that here instead of
deferring it.

1. **Language.** Which natural language should I use for our conversation and
   for all generated files? Default: English. If the user picks another
   language, switch immediately, including the next question.

2. **Operating system family.** Windows, macOS, or Linux? Ask this question
  in order. If you can detect the current OS from the environment, present
  that detected value as the default in the question; otherwise ask without
  a default. This determines the launcher mechanism in section 10.

3. **Programming language for scripts.** Default: Python (standard library
   only). Other supported choices: Bash, Node.js, Go. If the user picks
   something else, confirm you can produce idiomatic code for it; if not,
   recommend a fallback.

4. **Workflow root path.** Absolute path where the workflow should be
   installed. Default: the current working directory.

5. **AI tool for the `Default` worker.** Which command-line AI tool should
   `Scripts/Workers/Default.<ext>` use? Default: `opencode`
   (canonical recipe in section 11.1). The goal here is simple: make sure
   the generated launcher can open the `Installation Agent` successfully on
   first use. For any non-default answer, collect enough information to wire
   it up correctly per section 4.2 and section 9.6:
   - the binary name and an OS-specific fallback path if applicable
     (e.g. `%APPDATA%\npm\<name>.cmd` on Windows for npm-installed
     shims),
   - the command-line mode that accepts a single startup instruction,
   - the interactive mode that accepts a single startup instruction,
   - whether the tool supports attaching context files, and if so,
     the exact flag.

   Prefer to figure this out yourself before asking: if the binary is
   available on PATH, run `<binary> --help` (and `<binary> <subcommand>
   --help` for likely subcommands), propose the command patterns and
   attach flag based on that output, and confirm the proposal with the
   user in one message. Only ask follow-up questions for the details you
   could not determine. If the binary is not available, ask the user
   directly. Either way, write the final answer into
   `Scripts/Workers/Default.<ext>` during scaffolding; do not leave this
   choice for the `Installation Agent`.

   After the wrapper is ready, remind the user that many AI tools
   (`opencode`, `claude`, etc.) also read a per-directory settings file
   that controls permissions and access. Do **not** create or guess those
   files. Instead, in the final summary, point the user at the workflow
   root and say they may want to place the tool's own configuration there
   before first launch so the `Installation Agent` can work with the right
   permissions. Name the exact file(s) when you know them; otherwise say
   "consult the harness documentation".

6. **End-to-end rehearsal (optional).** After scaffolding finishes and the
   standard verification checklist (section 13) passes, would you like me
   to run a full practice pass of the workflow (install -> create
   workspace -> queue a task -> work-do -> work-undo -> archive) and then
   restore the workflow root to its post-scaffold state? Default: no. This
   practice pass checks that the scripts and launchers are connected
   correctly using a throwaway workspace and throwaway repos. It does
   **not** call the AI tool for real -- every harness invocation uses
   `--mode cli --dry-run`, so no tokens are spent and no model round-trips
   happen. If yes, follow the protocol in section 13.5 at the end of the
   verification checklist.

After question 6, summarize the chosen configuration in one short message,
then proceed to scaffolding (sections 6-12).

The scaffolded distributive must ship as a clean, generic baseline:

- `Scripts/Workers/Default.<ext>` is written for the harness chosen in
  interview question 5 (canonical `opencode` recipe in section 11.1 when
  the user accepts the default). It must work end to end at first launch
  so the top-level launcher can immediately open the `Installation Agent`
  through it. The `Installation Agent` does not reconfigure this wrapper.
- `Workspaces/__template__/` ships with **no** repository placeholders.
  The scaffolder creates the template root and its workflow files
  (`Prompts/`, `Work/`, per-workspace markdown stubs) but no
  `Configuration/`, `Documentation/`, `Implementation/`, or any other
  repo directories. The `Installation Agent` collects the user's actual
  repositories (URL + optional branch + optional folder name) and
  creates the directories and git state on the user's machine.
- `Workspaces/__template__/Workspace.md` uses the default naming and
  branch conventions described in section 8.5. The `Installation Agent`
  rewrites them to match the user's actual conventions.
- `Workspaces/__template__/Framework.md` is left as the brownfield stub
  from section 8.8. The `Installation Agent` optionally fills it in.
- `Workspaces/__template__/Prompts/Integration Agent.md` is left with the
  "not configured" external-tracker section from section 8.15. The
  `Installation Agent` updates it.

---

## 4. Invariants (must reproduce exactly)

These are non-negotiable. Reproduce the exact semantics regardless of language,
OS, or programming language. You may translate the prose and file names per
the adaptation rules in section 5, but the behavior, argument names, separators,
and metadata formats must match.

### 4.1 Dispatcher script (`Agent`)

A thin transport layer with this CLI:

- `--worker <display-name>` (optional, default `Default`)
- `--prompt <path>` (required)
- `--workspace <path>` (optional passthrough)
- `--mode cli|tui` (optional, default `cli`)
- `--tail <text>` (optional)
- `--agent-name <text>` (optional, metadata)
- `--context-files <comma-separated>` (optional)
- `--dry-run` (optional flag)
- `--new-window` (optional flag, passthrough hint for the worker)

Behavior:
- Validate `--prompt` exists.
- Resolve the worker display name to `Scripts/Workers/<name>.<ext>`, restricted
  to that folder. Sanitize the name by stripping every character outside
  letters, digits, space, hyphen, underscore, and dot; reject empty results.
  Also reject names that begin with a dot or consist only of dots and
  spaces after sanitization (this rules out things like `.bashrc` or
  `..` slipping through the alphabet check). After sanitization, resolve
  the candidate path and reject it if its parent is not the
  `Scripts/Workers/` directory (this rules out `..`, absolute paths, or
  any other escape attempt).
- Build a subprocess command that invokes the resolved worker script with all
  passthrough arguments, normalizing `--context-files` to absolute paths.
- Forward the subprocess exit code as the dispatcher exit code.
- Do no other interpretation. In particular, do not validate the workspace.
- `--new-window` is forwarded verbatim to the worker; the dispatcher itself
  does not interpret it.

### 4.2 Worker wrapper contract

Every worker wrapper accepts the same arguments as the dispatcher passes
through. Each wrapper:

- Builds a **bootstrap** string for its harness in this exact shape:
  - line 1: the translated form of `Read the file at '<absolute-prompt-path>' and follow the instructions exactly.`
  - if `--tail` is provided and contains at least one non-whitespace
    character (i.e. `tail.strip()` is non-empty), append a blank line,
    the translated form of the fixed prefix
    `After reading the files above, the user asked you to do this:`, a
    newline, then the tail text verbatim. Whitespace-only tails are
    treated as absent and the bootstrap stops at line 1.
- The path token (single-quoted absolute prompt path) and the user's tail
  text are inserted verbatim into the translated sentences and must not be
  altered.
- Both translated sentences must be identical across every worker wrapper
  shipped in this distributive and across every invocation; pick one
  translation per sentence at scaffold time and reuse it.
- Invokes the harness in CLI or TUI mode based on `--mode`.
- Resolves the harness binary defensively. On Windows, if a direct lookup
  fails, also try `%APPDATA%\npm\<binary>.cmd` before giving up.
- If the harness binary cannot be started, exit with code 127 and a clear
  message naming the missing binary.
- `--context-files` is a hint only. If the harness supports attaching files,
  pass them; otherwise ignore silently.
- `--new-window` is a Windows-only convenience flag used by Windows
  per-agent launchers and the top-level Windows launcher so the launching
  console does not stay blocked. When set and `--mode tui` and the OS is
  Windows, the wrapper spawns the harness in a detached new console (e.g.
  `subprocess.Popen` with `CREATE_NEW_CONSOLE`) and returns 0 immediately
  without waiting. Ignored on other OSes or in `cli` mode.
- The harness subprocess is started with its working directory set to the
  resolved `--workspace` path when provided, otherwise the wrapper's current
  working directory.
- The user-facing sentences in the bootstrap ("Read the file at '...' and
  follow the instructions exactly." and "After reading the files above,
  the user asked you to do this:") are the two pieces of prose that must be
  translated into the chosen language and used consistently across every
  wrapper. Only those two sentences are translated; the embedded prompt
  path and the user's tail text stay verbatim.

### 4.3 Work queue files

Inside each workspace, the directory `Work/` contains exactly four files:
`Next`, `Current`, `Blocked`, `Done` (with the markdown extension used by the
project, normally `.md`).

- Each file holds zero or more chunks.
- Chunks are separated by a line that contains exactly three hyphens: `---`.
- Empty chunks are ignored on read.
- Round-trip rule: read chunks, mutate, write chunks back; content of
  unaffected chunks must not change other than trailing whitespace
  normalization.

### 4.4 Rollback metadata header

When a chunk moves into `Done`, it must carry a header on its first line in
exactly this shape:

```
<!-- rollback: RepoNameA=<hash> RepoNameB=<hash> ... -->
```

- Keys are workspace-relative repository directory names.
- Values are full git commit hashes (output of `git rev-parse HEAD` at the
  moment the task entered `Current`).
- Repos are listed in case-insensitive alphabetical order by name.
- Whitespace: single space between tokens. Single space after the colon and
  before the closing `-->`.
- Zero-repo workspaces (no immediate subdirectory is a git working tree)
  still get a header, in the canonical empty form
  `<!-- rollback: -->` (single space after the colon, single space before
  the closing `-->`, no tokens between them). `Work - Undo` treats such
  headers as a no-op for the git phase and only restores the chunk text
  to `Next`.

### 4.4a Blocked reason header

When a chunk moves into `Blocked` (because the verifier reported `FAIL`,
emitted an invalid result, or because the verifier dispatcher exited
non-zero), `Work - Do` must record **why** directly inside the chunk so
the user reading `Work/Blocked.md` can act on it without inspecting logs.

Insert the blocked-reason header on the line immediately after the
existing rollback header (which is preserved verbatim). If there is no
rollback header for some reason, the blocked-reason header is the first
line of the chunk. Shape:

```
<!-- blocked: kind=<KIND> reason="<one-line reason>" -->
```

- `<KIND>` is one of the literal ASCII tokens `FAIL`, `INVALID`, or
  `DISPATCHER` (verifier dispatcher exited non-zero).
- `<one-line reason>` is a single-line human-readable explanation in
  the chosen natural language:
  - for `FAIL`: the text after the `FAIL: ` prefix in the verifier's
    last non-empty line, verbatim;
  - for `INVALID`: a fixed translated sentence stating that the
    verifier output did not end with `PASS` or `FAIL: <text>`;
  - for `DISPATCHER`: a fixed translated sentence stating that the
    verifier dispatcher exited with code `<N>`.
- Embedded double quotes in the reason are replaced with single
  quotes; newlines and the literal sequence `-->` are replaced with
  spaces, so the header always fits on one line and cannot terminate
  the comment early.
- Whitespace: single space between tokens; single space after the
  colon and before the closing `-->`.

When a blocked chunk is later moved to `Current` (via
`--blocked-action to-current`) or to `Done` (via `--blocked-action
to-done`), strip the blocked-reason header before writing. The rollback
header stays.

### 4.5 `Work - Do` state machine

Invoked non-interactively with these arguments:

- `--workspace <path>` (optional; default: current directory if it looks like
  a workspace, i.e. contains the workspace document and `Work/`)
- `--execution-worker <display-name>` (optional, default `Default`)
- `--verification-worker <display-name>` (optional, default `Default`)
- `--blocked-action to-current|to-done` (optional)
- `--current-action restart|to-done` (optional)
- `--mode cli|tui` (optional, default `cli`)
- `--dry-run` (optional flag, passed through)

Decision order on each invocation:

1. If `Blocked` is non-empty:
   - If `--blocked-action` is not provided: exit non-zero with a message
     naming the two valid values and stop.
   - If `to-current`: pop the first blocked chunk. If `Current` is non-empty,
     exit non-zero. Otherwise write the chunk to `Current`.
   - If `to-done`: pop the first blocked chunk, strip any blocked-reason
     header, and append the result to `Done`.
   - Return after handling one blocked item.

2. Else if `Current` is non-empty:
   - If `--current-action` is not provided: exit non-zero with a message
     naming the two valid values and stop.
   - If `to-done`: move all current chunks to `Done` and stop.
   - If `restart`: fall through to the execution step below using the existing
     current chunk (do not recapture rollback metadata).

3. Else if `Current` is empty and `Next` is non-empty:
   - Pop the first chunk from `Next`.
   - Capture per-repo HEAD hashes by running `git rev-parse HEAD` in each
     immediate subdirectory of the workspace that is a git working tree.
   - Prepend the rollback header (section 4.4) to the chunk.
   - Write it to `Current`.
   - Continue to execution. Note: this pop-and-write step runs even when
     `--dry-run` is set; the dry-run guard only skips the verifier-output
     parsing and the final `Current -> Done` move in step 4. A dry run
     therefore still mutates `Next` and `Current` on disk.

4. Execution and verification:
   - Build a tail string equal to the current chunk body (rollback header
     stripped).
   - Invoke the dispatcher with the execution worker, the execute prompt
     (`Prompts/Work - Execute.md` inside the workspace), the workspace path,
     the mode, and the tail.
   - If the dispatcher exits non-zero: print a clear message that the task
     remains in `Current` and exit with that code.
   - Otherwise invoke the dispatcher again with the verification worker and
     the verify prompt (`Prompts/Work - Verify.md` inside the workspace) and the
     same tail. The verifier dispatcher call must run with `--mode cli`
     regardless of the mode requested by the caller, because the
     `PASS` / `FAIL: <text>` protocol requires a captured stdout that
     a TUI harness cannot provide. Capture the verifier's **stdout** so
     its trailing line can be inspected; stderr may be streamed live or
     captured for replay. Replay the captured output to the user's
     stdout/stderr after the call (streaming stdout live is also
     acceptable as long as the verifier's last non-empty line remains
     parseable). The executor call may use whichever mode the caller
     requested and may stream output live.
   - If the verifier dispatcher exits non-zero: move the current chunk to
     `Blocked` (preserving its rollback header, adding a blocked-reason
     header per section 4.4a with kind `DISPATCHER`) and exit with that
     code.
   - Otherwise parse the verifier's stdout. Take the last non-empty line.
     - If it equals `PASS` exactly: success.
     - If it starts with `FAIL: ` and has non-empty text after the prefix:
       move the current chunk to `Blocked` (preserving its rollback header,
       adding a blocked-reason header per section 4.4a with kind `FAIL`
       and the verbatim text after `FAIL: ` as the reason) and exit with
       code 3.
     - Anything else is `INVALID`: same handling as `FAIL` (move to
       `Blocked`, add a blocked-reason header with kind `INVALID`, exit
       3) with a message that the verifier output did not end with
       `PASS` or `FAIL: <text>`.
   - On success: move the current chunk to `Done`, keeping its rollback
     header. Print one short success line.
   - When `--dry-run` is set, skip verifier-output parsing (there is no real
     verifier result to inspect) and do not move the current chunk; the
     dispatcher's own dry-run output is the only effect.

5. Else (`Blocked`, `Current`, and `Next` all empty):
   - Print a single line indicating there is nothing to do and exit 0.

Additional context passed to the dispatcher: build the `--context-files`
value from existing artifacts in the workspace such as
`Issue`, `Research`, `Plan`, `Status`, `Framework`, `Notes`, `Assignments`,
`Work/Current`, `Work/Done`. Only include files that exist. Use absolute
paths. `Facts.md` is intentionally **excluded** from this list even when
it exists: per section 8.22 it is a search-on-demand reference and must
not be auto-attached, so agents look up only the relevant section instead
of reading the whole file.

### 4.6 `Work - Undo` state machine

Invoked non-interactively with these arguments:

- `--count <N>` (required, positive integer)
- `--workspace <path>` (optional; same default as `Work - Do`)

Behavior:

1. Read `Done` chunks. Take the last `min(N, len(done))` chunks as the undo
   set.
2. Preflight: for every rollback entry across the undo set, verify the target
   commit exists in the corresponding repository. If any check fails, exit
   non-zero before making any changes.
3. For each affected repository, run `git reset --hard <commit>` then
   `git clean -fdx`. When more than one chunk in the undo set references
   the same repository, use the **oldest** captured hash (the rollback
   header of the earliest chunk in the undo set, i.e. the deepest point
   in history) so the working tree returns to the state that existed
   before the first undone chunk. Stop on first repo failure with a
   non-zero exit; do not try to recover, but report which repo and what
   failed.
4. Strip rollback headers from the undo set and prepend them to the top of
   `Next` in their original order. Write `Done` without the undone chunks.

Destructive intent is by design: dirty working trees are erased.

### 4.7 `Workspace - Create`

Invoked with:

- `--workspace <name>` (required; a single path-safe segment under the
  `Workspaces` root)
- `--branch <branch-name>` (required)

Behavior:

1. Validate the workspace name before touching the destination path.
  Reject empty names, path separators, `.` and `..`, reserved names
  `__template__` and `__archive__`, and any input that would resolve
  outside the `Workspaces` root.
2. Refuse if the target directory already exists. This existence check
   must be the **very first** action on the destination path, before any
   other side effect that could create or touch it. In particular, the
   workspace logging helper from section 9.7 creates the workspace
   directory on first write, so emitting any log line for this workspace
   before the existence check would make the check always pass and
   silently clobber a pre-existing workspace. Order the script as:
   `dest.exists()` check first, then create the destination directory,
   then log.
3. Copy every non-git, non-launcher file and directory from
   `Workspaces/__template__` into the new workspace path. Skip launcher files
   (they are regenerated in step 5).
4. Discover repositories: every immediate subdirectory of
   `Workspaces/__template__` that contains a `.git` entry (file or directory)
   is a repository.
5. For each discovered repository:
   - Require an existing `HEAD` commit. If `git rev-parse --verify HEAD`
     fails, abort the entire create with a clear message instructing the
     user to run installation setup so each template repo has at least one
     initial commit (an empty commit is acceptable).
   - Detect the currently checked-out branch in the template repo.
   - If the current branch has a configured upstream, run `git pull --ff-only`
     in the template repo. If it does not, skip the pull.
   - If a local branch with the requested workspace branch name already exists
    in that repo, attach a worktree at `<workspace-name>/<repo-name>` to it.
     Otherwise create a new branch from `HEAD` and create the worktree.
6. Generate per-agent launchers inside the new workspace using the recipe in
   section 10 for the OS chosen at installation time. The script only emits
   launchers for that OS; on other OSes the step is a silent no-op so
   workspace creation still succeeds when the directory tree is shared
   across machines. Five launchers, numbered:
  1. Integration Agent
   2. Research Agent
   3. Planner Agent
   4. Worker Agent
   5. Reviewer Agent

  Each launcher invokes the dispatcher with `--worker Default`,
  `--prompt Prompts/<Agent>.md` (relative to the workspace), `--workspace .`,
  `--mode tui`, and `--agent-name "<Agent>"`. On Windows, also pass
  `--new-window` so the worker wrapper can detach into a new console per
  section 4.2.

### 4.8 `Workspace - Remove` (archive only)

Invoked with:

- `--workspace <name>` (required; a single path-safe segment under the
  `Workspaces` root)
- `--synced` (required flag; without it the script refuses to run)
- `--include-uncommitted` (optional flag; allow archiving with dirty
  worktrees and snapshot their working-tree files into the archive)

Behavior:

1. Validate the workspace name before touching the source path. Reject empty
  names, path separators, `.` and `..`, reserved names `__template__` and
  `__archive__`, and any input that would resolve outside the `Workspaces`
  root.
2. If `--synced` is missing, exit non-zero with a message explaining the user
  must run the Integration Agent's backlog/changelog sync first and then rerun with
   `--synced`.
3. Discover workspace repositories (immediate subdirectories of the workspace
   containing a `.git` entry, file or directory).
4. For each repo run `git status --porcelain`. Collect the set of dirty
   repos. If any are dirty and `--include-uncommitted` was not given, exit
   non-zero listing the dirty repo names and instructing the user to commit,
   stash, discard, or rerun with `--include-uncommitted`.
5. Compute the archive destination as
  `Workspaces/__archive__/<workspace-name>`. If the destination already
  exists, append `__YYYY-MM-DD_HHMMSS` to the workspace name.
6. Move every non-repository child of the workspace into the destination,
   preserving structure.
7. For each repo, perform these sub-steps **in order**:
   1. **Snapshot (conditional).** If the repo is dirty and
      `--include-uncommitted` was given, copy the working tree (excluding
      `.git`) into the destination under the repo name. This must happen
      *before* any worktree removal so the dirty files are preserved on
      disk. If the repo is clean, or `--include-uncommitted` was not
      given (in which case the script would have already exited at step 4
      for dirty repos), skip this sub-step.
   2. **Remove the repo from the workspace.** After the optional snapshot,
      always detach/delete the repo directory from the workspace:
      - If the repo's `.git` is a worktree pointer (file whose `gitdir:`
        value resolves under `<primary>/.git/worktrees/<name>`), invoke
        `git worktree remove <repo-dir>` from the primary repo (with
        `--force` when `--include-uncommitted` is set, since the worktree
        is dirty).
      - Otherwise (a plain repo directory, no primary detected) remove the
        directory from the filesystem.

   The snapshot and the removal are sequential, not alternative: a dirty
   repo with `--include-uncommitted` is both snapshotted and then removed.
8. Run `git worktree prune` once per unique primary repo to clean up stale
   worktree records. Prune failures are warnings, not fatal.
9. Attempt to remove the now-empty workspace directory (best effort).
10. Do not delete any local or remote git branches. Do not run `git reset`,
    `git clean`, or any other destructive git operation on repository
    history. Worktree removal and pruning are allowed and required.

### 4.9 `Status` document columns

Whatever the chosen language and filename, the `Status` artifact contains one
table with exactly these columns in this order:

`Part | Expected | Current | Completion % | Last Checked`

Free-form notes may follow the table. This format is for the agent's
guidance, not for machine parsing.

### 4.10 Research agent interaction

The Research Agent prompt must instruct the agent to:

- Before researching, ask the user whether they have any pre-context, notes,
  or constraints to add.
- Ask one question at a time during research.
- Write findings to the workspace's `Research` document in free form.
- Optionally propose deferred items into the workspace's `Backlog` document.

### 4.11 Defaults for `Work - Do`

- Execution and verification workers both default to `Default`.
- Do not read `Assignments` to pick workers automatically. `Assignments` is a
  human-facing preference document only.
- Execute prompt: `Prompts/Work - Execute.md` inside the workspace.
- Verify prompt: `Prompts/Work - Verify.md` inside the workspace.
- `Facts.md` is **not** added to the dispatcher's `--context-files`
  even when present (see section 4.5). The Execute and Verify prompts
  are aware of it and may consult it on demand by searching for the
  relevant section title.

### 4.12 Launcher switch after installation

This switch is performed by the `Installation Agent` (the prompt described
in section 8.2), not by the scaffolder running this document. As the
scaffolder, you must leave the top-level launcher pointing at the
`Installation Agent` prompt.

When the end user later runs the `Installation Agent` and completes its
checklist, that agent changes the top-level launcher so it opens the
`Workspace Agent` prompt instead of the `Installation Agent` prompt. The
launcher file name stays the same. The required semantic change is the
prompt target; other metadata such as `--agent-name` may either stay as-is
or be updated to match the new prompt, as long as the launcher still invokes
the same dispatcher and worker correctly.

### 4.13 Per-workspace agent TUI welcome and idle-on-start

Applies to exactly these five per-workspace agent prompts:
`Prompts/Integration Agent.md`, `Prompts/Research Agent.md`,
`Prompts/Planner Agent.md`, `Prompts/Worker Agent.md`,
`Prompts/Reviewer Agent.md` (the prompts copied into every workspace by
`Workspace - Create`). Does **not** apply to `Installation Agent.md`,
`Workspace Agent.md`, `Work - Execute.md`, or `Work - Verify.md`.

Required behavior at session start when the agent is invoked in
`--mode tui`:

- The agent's very first message is a short welcome that names its role
  and then waits for user input. It must not read workspace files, run
  any tools, or start any task before the user replies.
- The welcome is a single short translated sentence built from this
  canonical English template:
  `Hello, I am the <Agent Name>. What would you like to do?`
  `<Agent Name>` is the human-facing agent role name (`Integration Agent`,
  `Research Agent`, `Planner Agent`, `Worker Agent`, `Reviewer Agent`)
  and is inserted verbatim. The sentence wording itself is translated
  at scaffold time per section 5.1 and reused consistently across the
  five prompts.
- The welcome may be followed by one optional short hint line (still
  before any work) reminding the user that they can describe a task or
  ask the agent to summarize its capabilities. The hint is optional;
  the greet-and-wait rule is not.

In `--mode cli` (unattended) the rule does **not** apply: the agent
should begin acting on the prompt and the `--tail` payload immediately,
because there is no interactive user to greet.

### 4.14 Per-workspace in-chat role switching

Applies only to the same five per-workspace agent prompts listed in
section 4.13 (`Integration`, `Research`, `Planner`, `Worker`,
`Reviewer`). It does **not** apply to `Installation Agent`,
`Workspace Agent`, `Work - Execute`, or `Work - Verify`.

Required behavior when one of these agents is running in `--mode tui`:

- The agent recognizes role-switch requests in the current chat using
  these command patterns (case-insensitive):
  - `switch to <role>`
  - `become <role>`
  where `<role>` resolves to one of the five allowed role names above
  (optionally with the word `agent`).
- Role-name normalization is deterministic (case-insensitive, optional
  trailing `agent`). Normalize user input to canonical role keys using
  this table:
  - `integration`, `integration agent` -> `Integration`
  - `research`, `research agent` -> `Research`
  - `planner`, `planner agent` -> `Planner`
  - `worker`, `worker agent` -> `Worker`
  - `reviewer`, `reviewer agent` -> `Reviewer`
  Prompts may recognize chosen-language equivalents, but they must map
  to these same canonical role keys before switching.
- On a valid request, the handoff is **in-chat** and **same-session**:
  the agent switches to the target role prompt and continues in the
  same chat, in the same workspace, with prior conversation context
  preserved.
- The agent acknowledges the switch in one short line before continuing
  under the target role behavior.
- Mixed-intent precedence is deterministic:
  - If one message contains exactly one valid role switch and additional
    task instructions, switch first, then handle the remaining
    instruction text under the target role.
  - If one message contains more than one different switch target,
    ask one clarification question and do not switch until resolved.
  - If the user explicitly asks to open a separate window or names a
    different workspace, use launcher-based opening per section 8.3
    rather than in-chat switching.
- Capability-mismatch handling is proactive:
  - If the user asks for an action that is outside the current role's
    responsibilities but clearly matches another one of the five
    per-workspace roles, the agent should explain this briefly and offer
    to switch in the same session.
  - If exactly one best-fit target role is clear, offer that role in one
    short yes/no question.
  - If more than one target role is plausible, ask one short question
    listing the candidate roles and wait for the user's choice before
    switching.
  - If the user accepts, switch in-chat per this section. If the user
    declines, continue in the current role and provide the best in-scope
    help available.
- If the target role is the current role, treat it as a no-op and reply
  with a short message that the session is already in that role.
- If the target role is unknown, missing, or outside the allowed five,
  refuse with a short message listing the valid targets.
- Requests to switch to `Installation Agent`, `Workspace Agent`,
  `Work - Execute`, or `Work - Verify` are explicitly rejected.

In `--mode cli` (unattended), role switching is disabled: the prompt and
`--tail` payload are executed as provided for that invocation.

---

## 5. Adaptation surfaces

You may adjust these freely.

### 5.1 Natural language

All user-facing text (prompts, READMEs, stubs, comments, printed messages)
must be in the language chosen in interview question 1. This explicitly
includes:

- Both translated sentences of the harness bootstrap from section 4.2
  (the "Read the file at ..." line and the tail prefix).
- Per-workspace in-chat role-switch commands from section 4.14. Prompts
  must recognize the chosen-language equivalents of
  `switch to <role>` and `become <role>`.
- Capability-mismatch switch-offer phrasing from section 4.14 (brief
  scope explanation plus offer to switch in-chat).
- Every heading and section label in the per-file specs of section 8.
  When section 8 lists English headings like "Goal", "Rules", "Inputs",
  "Responsibilities", "Outputs", "Notes", "Typical tasks", "Required
  behavior", "Output expectation", "Existing plan policy", "Structure",
  "Workspace naming convention", "Branch convention", "Repository layout",
  "Verification notes", "Navigation map", "Build / test / run / verify",
  "Automation boundaries", "Required", "Plan", "Risks", "Verification
  strategy", "Title", "Summary", "Validation", "Follow-ups", "Entry
  format", and so on, those labels are translated too. The English forms
  in this specification are reference identifiers, not literal output.
- Printed messages and log event labels emitted by scripts (the `[work-do]`
  / `[workspace-archive]` / etc. tags may stay as ASCII identifiers, but
  the human-readable messages that follow them are translated).
- Comments inside generated scripts.

Protocol literals that are **not translated** under any circumstances
(they are machine-parsed tokens, not user-facing prose):

- The verifier output tokens `PASS` and the `FAIL: ` prefix from
  section 4.5 / section 8.21. The free-text explanation after `FAIL: `
  is in the chosen language, but the prefix itself is the literal ASCII
  string `FAIL: ` (uppercase F-A-I-L, colon, single space).
- The rollback header marker `<!-- rollback: ... -->` from section 4.4
  and the blocked-reason header marker `<!-- blocked: ... -->` from
  section 4.4a. Keys and structure are fixed ASCII; only the embedded
  free-text reason value is in the chosen language.
- The chunk separator line `---` from section 4.3.
- Dispatcher / worker / `Work - *` CLI flag names (`--prompt`, `--mode`,
  `--workspace`, `--blocked-action to-current|to-done`, etc.). Their
  *help text* is translated; the flag names and enum values stay
  ASCII.
- Log event identifiers from section 9.7 (`work-do.start`,
  `work-do.blocked`, etc.).

When producing the per-file specs in section 8 (especially
`Prompts/Work - Verify.md`), the verifier-output rule must read the same
way in every language: the last non-empty line is `PASS` or
`FAIL: <one-line reason>`, where `PASS` and `FAIL:` are literal ASCII
tokens that must not be localized.

File and directory names follow the separate rule in section 5.2.

### 5.2 File and directory names

By default, use the canonical English names below. If the user wants
localized names, you may translate them, but you must:

- Apply the mapping consistently across every file, launcher argument,
  script reference, and prose mention.
- Keep the file structure and relationships identical.
- Use file system-safe characters only.
- Record the mapping table in `README.md` under a short dedicated section;
  do not modify `INSTALL.md`.

Canonical names (English defaults):

```
INSTALL.md                                        (this file is consumed by the AI; do not regenerate it)
README.md
Open Agent                                        (top-level launcher; extension depends on OS)
Prompts/
  Installation Agent.md
  Workspace Agent.md
Scripts/
  Agent.<ext>                                     (dispatcher)
  Workspace - Create.<ext>
  Workspace - Remove.<ext>
  Work - Do.<ext>
  Work - Undo.<ext>
  WorkflowLog.<ext>                               (shared logging utility, section 9.7)
  Workers/
    Default.<ext>
Workspaces/
  Backlog.md
  Changelog.md
  __archive__/                                    (empty directory; reserved)
  __template__/
    1. Open Integration Agent                     (per-agent launcher)
    2. Open Research Agent
    3. Open Planner Agent
    4. Open Worker Agent
    5. Open Reviewer Agent
    Workspace.md
    Assignments.md
    Backlog.md
    Changelog.md
    Facts.md
    Framework.md
    Issue.md
    Notes.md
    Plan.md
    PR.md
    Research.md
    Status.md
    Prompts/
      Integration Agent.md
      Planner Agent.md
      Research Agent.md
      Reviewer Agent.md
      Worker Agent.md
      Work - Execute.md
      Work - Verify.md
    Work/
      Blocked.md
      Current.md
      Done.md
      Next.md
```

`<ext>` matches the chosen programming language (`.py`, `.sh`, `.js`, `.go`,
etc.). Launchers have OS-specific extensions per section 10.

Note: each workspace also accumulates a plain-text `log.txt` at its root
once `WorkflowLog` (section 9.7) writes to it. This file is created on
demand by workflow scripts, is not part of the template, and is not
listed in the manifest tree above; the verification cleanup pass
(section 13 step 0) leaves any existing `log.txt` inside live workspaces
alone but treats stray `log.txt` files outside `Workspaces/<name>/` as
removable scratch.

### 5.3 Programming language

You may translate the scripts in section 9 into any commonly available
language. Defaults to Python with standard library only.

Requirements regardless of language:

- Argument parsing with explicit long flags as specified.
- Subprocess invocation with explicit argument vectors (no shell string
  interpolation).
- File copy/move that preserves directory structure.
- Read and write text files as UTF-8.
- Force UTF-8 for every I/O boundary the script touches, not just file
  reads/writes. The chosen natural language may contain non-ASCII
  characters, and the default console/subprocess encodings on Windows
  (cp1251, cp1252, cp866, etc.) will corrupt them otherwise. Concretely:
  - process stdout/stderr must emit UTF-8 (Python: reconfigure
    `sys.stdout`/`sys.stderr` with `encoding="utf-8"`, or set
    `PYTHONIOENCODING=utf-8`; Node.js: `process.stdout.setDefaultEncoding`
    is not enough on Windows, prefer writing buffers; Bash: ensure the
    locale is UTF-8);
  - subprocess captures must decode as UTF-8 (Python: pass
    `encoding="utf-8"` together with `text=True` to `subprocess.run`/
    `Popen`; never rely on the platform default);
  - subprocess invocations that forward our translated strings (the
    bootstrap from section 4.2, the `--tail` value, printed messages) must
    pass through unchanged; set `PYTHONIOENCODING=utf-8` (or the
    equivalent) in the child environment when spawning interpreters of the
    same language so the chain stays UTF-8 end to end;
  - log files written by `WorkflowLog` (section 9.7) are opened in UTF-8
    explicitly;
  - **Windows console code page.** On Windows, the console code page
    (not `PYTHONIOENCODING`) is what every non-Python child inherits,
    including the AI harness and any shell it spawns (PowerShell, cmd).
    The default is cp1252 / cp866 / cp1251 depending on the system
    locale, which corrupts UTF-8 bytes coming from upstream and from
    downstream tool calls (typical symptom: harness output shows `?`
    characters instead of accented letters). Every script that spawns a
    subprocess on Windows -- at minimum the dispatcher
    (`Scripts/Agent.<ext>`) and every worker wrapper under
    `Scripts/Workers/` -- must set both the input and output console
    code pages to 65001 (UTF-8) at startup, before any subprocess is
    launched. In Python this is:

    ```python
    if os.name == "nt":
        import ctypes
        k32 = ctypes.windll.kernel32
        k32.SetConsoleCP(65001)
        k32.SetConsoleOutputCP(65001)
    ```

    Use the idiomatic equivalent for the chosen programming language.
    If the chosen language cannot do this without third-party
    dependencies, recommend Python instead of adding extra packages.
    This is in addition to, not a replacement for, the Python-side
    `PYTHONIOENCODING` / `sys.stdout.reconfigure` settings above.
  Pick the idiomatic enforcement for the chosen language and apply it in
  every script the scaffolder produces.

### 5.4 OS launcher mechanism

Pick the recipe in section 10 that matches the chosen OS. The launcher
invokes the dispatcher with the appropriate prompt path and worker.
On Windows, launchers must be plain-text `.cmd` files, not `.lnk`
shortcuts. For Python-based distributions, the launcher must resolve
Python in this exact order: `py` on `PATH`, then `python` on `PATH`, then
`%LOCALAPPDATA%\Programs\Python\Launcher\py.exe`. The launcher must not
hardcode a user-profile absolute path to Python, and it must preserve the
agent invocation arguments and working-directory semantics from section
10.1.

### 5.5 Harness invocation

Each worker wrapper builds the harness command for both `cli` and `tui`
modes. The bootstrap shape in section 4.2 is fixed; the surrounding command
is whatever the harness expects. See section 11.

---

## 6. Architecture overview

Lifecycle:

1. The user opens the top-level launcher. The first time, it opens an AI
   agent with the `Installation Agent` prompt. That agent runs the
   installation checklist (section 8.2) and, as its last step, switches the
   same launcher to point at the `Workspace Agent` prompt.
2. From the second launch onward, the same launcher opens the `Workspace
   Agent` prompt.
3. The Workspace Agent can create a workspace, archive a workspace, or open
   per-workspace agents.
4. Inside a workspace, agents are: Integration, Research, Planner, Worker,
  Reviewer.
   Their prompts live in `Workspaces/__template__/Prompts/` and are copied
   per workspace during `Workspace - Create`.
  In `--mode tui`, these five agents also support in-chat role switching
  within the same workspace and same chat session (section 4.14). This
  does not replace launcher-based opening; both flows are valid.
  The Integration Agent is git-first by default and adapts to configured
  tracker and tooling integrations.
5. Worker drives a markdown-backed task queue via `Work - Do` and
   `Work - Undo`.
6. When work is finished, the Reviewer Agent produces a `Changelog` and `PR`
  artifact. The Integration Agent performs explicit git operations on
  request, then finalizes and syncs the workspace's `Backlog` and
  `Changelog` into the global `Workspaces/Backlog.md` and
  `Workspaces/Changelog.md`.
7. `Workspace - Remove` archives the workspace under `__archive__`.

---

## 7. File manifest

Create exactly these files. Do not create more, do not skip any.

### Root

- `README.md` - short overview of the workflow, the lifecycle, and how to
  launch.
- `Open Agent.<launcher-ext>` - top-level launcher (see section 10). Points
  at the `Installation Agent` prompt. The `Installation Agent` switches it
  to the `Workspace Agent` prompt at the end of its own checklist
  (section 4.12).
- `Prompts/Installation Agent.md` - prompt that drives an installation
  conversation similar to the one that produced this workflow. Tells the AI
  to verify worker wrappers, repositories, naming, optional framework
  mapping, and finally to switch the launcher. Platform-agnostic content.
- `Prompts/Workspace Agent.md` - prompt that orchestrates workspace creation,
  archival, and opening per-workspace agents.

### Scripts

- `Scripts/Agent.<ext>` - the dispatcher described in section 4.1 and 9.1.
- `Scripts/Workspace - Create.<ext>` - section 4.7 and 9.2.
- `Scripts/Workspace - Remove.<ext>` - section 4.8 and 9.3.
- `Scripts/Work - Do.<ext>` - section 4.5 and 9.4.
- `Scripts/Work - Undo.<ext>` - section 4.6 and 9.5.
- `Scripts/Workers/Default.<ext>` - worker wrapper for the chosen harness in
  CLI and TUI modes (section 4.2, 9.6, and 11).
- `Scripts/WorkflowLog.<ext>` - shared logging utility (section 9.7) imported
  by the workflow scripts above.

### Workspaces root

- `Workspaces/Backlog.md` - global backlog stub with entry format.
- `Workspaces/Changelog.md` - global changelog stub with entry format.
- `Workspaces/__archive__/` - empty directory; reserved.
- `Workspaces/__template__/` - everything below. The template ships with
  **no** repository directories: only the workflow coordination
  subdirectories (`Prompts/`, `Work/`) and the per-workspace markdown
  files. The `Installation Agent` creates repository directories on
  the user's machine based on the user's answers (section 8.2 step 2).

### Template per-workspace artifacts

- `Workspace.md` - explains structure, repository roles, branch convention,
  and verification notes.
- `Assignments.md` - free-form worker preference notes (human-facing only).
- `Backlog.md` - per-workspace backlog stub.
- `Changelog.md` - per-workspace changelog stub.
- `Facts.md` - durable, agent-curated record of confirmed facts about
  the workspace and project; append-only `## <Title>` sections; see
  section 8.22.
- `Framework.md` - high-level project navigation and build/test/run/verify
  notes; stub explains brownfield assumption when empty.
- `Issue.md` - structured issue capture stub.
- `Notes.md` - free-form user notes stub.
- `Plan.md` - plan stub with risk and verification sections.
- `PR.md` - pull request draft stub.
- `Research.md` - research artifact stub; default text says "No research."
- `Status.md` - status table stub with the five columns from section 4.9.

### Template per-agent prompts

- `Prompts/Integration Agent.md`
- `Prompts/Research Agent.md`
- `Prompts/Planner Agent.md`
- `Prompts/Worker Agent.md`
- `Prompts/Reviewer Agent.md`
- `Prompts/Work - Execute.md`
- `Prompts/Work - Verify.md`

### Template work queue

- `Work/Next.md`
- `Work/Current.md`
- `Work/Blocked.md`
- `Work/Done.md`

### Template per-agent launchers

Five OS-specific launcher files numbered as in section 4.7 step 5. They open
each per-workspace agent through the dispatcher with the Default worker.

---

## 8. Per-file content specs

For every prompt or template document below, write the content in the chosen
language. Each spec lists the required headings and short guidance lines.
Keep the prose concise.

For user-facing prompts and explanations, lead with purpose and next step:
what is being done, what happens next, and how the user should use the
result. Avoid low-level implementation terms unless they are necessary for
correct execution or are protocol-critical.

The heading names quoted below (for example "Goal", "Rules", "Inputs",
"Required behavior") are reference identifiers in this specification. In
the files you actually produce, translate them into the chosen language
along with the rest of the prose, per section 5.1. Keep the order and the
meaning; only the wording changes.

### 8.1 `README.md`

A short one-pager. The exact headings are free, but the document must cover,
in any order:

- A two-to-three-line overview of the agentic, workspace-based delivery
  model.
- The launcher lifecycle (first run uses `Installation Agent`, subsequent
  runs use `Workspace Agent`).
- The dispatcher contract (the canonical flags of `Scripts/Agent.<ext>`).
- The work-queue file set and chunk separator.
- The typical end-to-end flow that mirrors section 6.
- A short safety note: workspace removal is archive-only, no remote push
  without explicit user request.
- A bullet list of the scripts with a one-line purpose each.

### 8.2 `Prompts/Installation Agent.md`

Headings: "Goal", "Rules", "Installation checklist", "Expected output
artifacts".

This agent is the **single owner** of every setup decision the scaffolder
deliberately left for later (section 3): template git repositories,
naming/branch conventions, framework mapping, external issue tracker, and
optional integrations / add-ons. The AI tool used by the `Default` worker
is **not** part of this follow-up work -- it was already configured by the
scaffolder during interview question 5, and this very agent is running
through it now. The scaffolder leaves only generic defaults behind; this
agent collects the user's real setup choices on their own machine and
updates the affected files to match.

Guidance:
- Goal: prepare this distributive so the next launch opens `Workspace Agent`
  instead of `Installation Agent`.
- Rules: keep the prompt platform-agnostic; one question at a time; do not
  reconfigure `Scripts/Workers/Default` (it is already wired by the
  scaffolder and is the very wrapper running this agent); produce
  concrete file updates and commands; explain what you are setting up, what
  happens next, and how the user will use the result; avoid low-level
  implementation detail unless it affects a real user decision; no remote
  push.
- Installation checklist (in order):
  1. **Additional worker wrappers (optional).** `Scripts/Workers/Default`
     was configured by the scaffolder and must not be changed here. If the
     user wants extra worker wrappers for other AI tools, create them
     alongside `Default` in `Scripts/Workers/` following section 4.2 and
     section 9.6. If not, skip this step.
  2. **Template repositories.** Ask the user, one repository at a time,
     which repositories should be part of the template in
     `Workspaces/__template__/` so new workspaces start with the right
     project folders ready. There is no default candidate list and no
     shortcut setup here -- the scaffolder ships an empty template.
     Repeat the per-repo interaction below until the user says they are
     done; the user may also say "none" at the first prompt, in which
     case workspaces will have no attached repositories (still valid; the
     zero-repo rollback header form from section 4.4 covers this).

     For each repository the user adds, collect:
     - **Source.** One of:
       (a) a clone URL (HTTPS or SSH) -- the agent will run
           `git clone <url> <folder>` inside `Workspaces/__template__/`;
       (b) "local-only" -- the agent will `git init` a fresh repo with
           one empty initial commit (`git commit --allow-empty -m
           <init message>`) on the default baseline branch `main`.
     - **Branch (optional).** Only meaningful for clone sources. If
       provided, check out that branch after cloning
       (`git -C <folder> checkout <branch>`). If omitted, accept the
       remote's default branch.
     - **Folder name (optional).** The directory name to use under
       `Workspaces/__template__/`. If omitted, derive it from the
       clone URL's last path segment with any trailing `.git` stripped
       (e.g. `https://example.com/org/foo.git` -> `foo`). For
       local-only repos the folder name is required. Reject folder
       names that collide with built-in workflow directories
       (`Prompts`, `Work`) or with any other name reserved by the
       manifest, and reject names containing path separators.

     Suggested one-question prompts (issued one at a time, per repo):

     > "Add a repository to the template? Reply with the clone URL,
     > or 'local' to start a fresh local repo, or 'done' to finish."

     Then, only when needed:

     > "Branch to check out for `<folder>`? (Press Enter to use the
     > remote default.)"

     > "Folder name for this repo under `Workspaces/__template__/`?
     > (Press Enter to use the inferred name `<inferred>`.)"

     After each repository is created, verify:
     - the directory exists at `Workspaces/__template__/<folder>/`,
     - it is a real git repo (`<folder>/.git` exists),
     - `git -C <folder> rev-parse --verify HEAD` succeeds (clones
       always have a HEAD; local-only repos must have the empty
       initial commit applied),
     - for clones, the requested branch (or the remote default) is
       checked out and the upstream is set as cloned.

     Never volunteer `Prompts/` or `Work/` as candidates; they are
     built-in workflow directories and never repositories.
  3. **Workspace naming and branch conventions.** Ask the user how workspace
      names and branch names should be constructed. Workspace names must be
      single-segment identifiers under `Workspaces/`, not nested paths.
      Rewrite
     `Workspaces/__template__/Workspace.md` so the "Workspace naming
     convention", "Branch convention", and "Repository layout" sections
     reflect the user's answers and the actual repositories from step 2.
  4. **Optional framework mapping.** Ask whether the user wants
     `Workspaces/__template__/Framework.md` filled in now from the
     repositories configured in step 2. If yes, produce a high-level
     navigation map and the build/test/run/verify guidance. If no, leave
     the brownfield stub in place and add a follow-up entry to
     `Workspaces/__template__/Backlog.md`.
    5. **Integration profile (git + external systems).** Ask where tasks and
      progress are tracked outside workspaces (for example Gitea, GitHub,
      GitLab, Jira, Linear, or none). Collect the details needed so issue
      and ticket references make sense inside the workflow, then update
      `Workspaces/__template__/Prompts/Integration Agent.md` accordingly.
      If none, leave the "not configured" section as-is and record that
      fact.
  6. **Optional integrations / add-ons.** Ask the user, in free form,
     whether they want any extra workflow behavior wired in. This is the
     catch-all step for anything that does not fit the previous categories:
     editor integration (for example "open the workspace in a new VS Code
     window when I ask"), notifications, custom MCP servers, shell
     aliases, environment variables required by the harness, and so on.

     Procedure:
     - Present one prompt similar to: "Any integrations or add-ons to
       wire in? Examples: open workspaces in VS Code on request,
       desktop notifications, an MCP server, etc. Say 'none' to skip."
     - If the user says 'none' (or equivalent), record that fact and
       move on.
     - Otherwise, for each requested integration, ask follow-up
       questions one at a time to gather the minimum required
       information (binary names, config paths, trigger conditions,
       which agent prompt should know about it). Apply each integration
       by editing the **existing** files that are responsible for that
       behavior; do **not** invent new top-level files or new scripts
       under `Scripts/` for an integration (the file manifest in
       section 7 is the upper bound on what may exist at workflow
       root). Acceptable edit targets are:
       - any per-workspace prompt under
         `Workspaces/__template__/Prompts/` (e.g. add a "VS Code"
         section to `Worker Agent.md` describing when and how to open
         the workspace),
       - `Workspaces/__template__/Framework.md` (commands the agents
         should run),
       - `Workspaces/__template__/Notes.md` (free-form user
         conventions),
       - the top-level `Prompts/Workspace Agent.md` (workflow-wide
         behavior, e.g. "when the user says 'open in VS Code', run
         `code <workspace-path>`"),
       - the existing worker wrappers under `Scripts/Workers/` only
         when the integration is genuinely a harness-launch concern.

      Example: "Integrate with VS Code: when I ask, open the current
      workspace in a new VS Code window". Wire this by adding a short
      section to `Prompts/Workspace Agent.md` (and/or the relevant
      per-workspace prompt) that explains when to do it and which command
      to run (`code --new-window <absolute-workspace-path>` on PATH, with
      an OS-appropriate fallback when `code` is not on PATH). Do not
      create new launcher files or scripts for it.

     If the user requests an integration that genuinely cannot be
     expressed by editing existing files within the manifest, decline
     and explain the constraint; capture the request as a follow-up
     entry in `Workspaces/__template__/Backlog.md` instead.
  7. **Verify installation.** Before switching the launcher, run a
     self-check and fix anything that fails. Do not ask the user to do
     this; do it and report results. Tell the user what was checked and
     whether it passed, without turning the summary into a low-level dump.
     Required checks:
     - Every repository directory the user added in step 2 exists at
       `Workspaces/__template__/<folder>/`, is a real git repository
       (`<folder>/.git` exists), is on the expected branch (the
       requested branch for clones, the default baseline branch `main` for
       local-only repos), and has a resolvable `HEAD`
       (`git -C <folder> rev-parse --verify HEAD` succeeds). Cloned
       repos have their upstream set as configured by `git clone`, and
       `Workspaces/__template__/` contains no extra directories beyond
       the ones the user added in step 2 and the built-in workflow
       directories (`Prompts/`, `Work/`).
     - `Workspaces/__template__/Workspace.md` reflects the user's
       repository list and naming/branch conventions from steps 2-3
       (the "Repository layout" section lists exactly the repos added
       in step 2; no leftover default placeholder mentions).
     - `Workspaces/__template__/Framework.md` matches the decision
       from step 4 (filled-in map or brownfield stub plus a backlog
       follow-up entry; not both, not neither).
     - `Workspaces/__template__/Prompts/Integration Agent.md` reflects the
       step-5 tracker decision (configured details, or an explicit
       "not configured" section).
     - The integrations chosen in step 6 are reflected in the files
       that step 6 says it may edit (e.g. a "VS Code" trigger section
       exists in `Prompts/Workspace Agent.md` if VS Code integration
       was requested). Integrations the user declined leave no traces
       behind.
     - Dispatcher smoke test: invoke `Scripts/Agent.<ext>` with
       `--worker Default --prompt <abs Workspace Agent path> --mode cli
       --dry-run` and confirm it prints a harness command without
       error. Do **not** smoke-test against `Installation Agent.md`
       once the launcher has been switched.
     - Top-level launcher (before the switch in step 8) still parses
       and targets the dispatcher with valid arguments per section 10.
       On Windows with Python launchers, confirm that `Open Agent.cmd`
       resolves Python in this order: `py` on `PATH`, then `python` on
       `PATH`, then `%LOCALAPPDATA%\Programs\Python\Launcher\py.exe`,
       and that it does not hardcode a user-profile absolute path.
     If any check fails, repair the offending artifact and re-run the
     affected check. Only proceed to step 8 once all checks pass.

  8. **Switch the top-level launcher.** Change the top-level `Open Agent`
     launcher so the next launch starts `Workspace Agent` instead of
     `Installation Agent` (section 4.12). Do not change the launcher file
     name or any other property. After the switch, re-run the dispatcher
     smoke test from step 7 against the new prompt target
     (`Workspace Agent.md`) to confirm the launcher still resolves to a
     valid harness command.

  9. **Offer to open Workspace Agent in a new window.** Immediately
     before emitting the final completion message in step 10, ask the
     user whether they want to launch the `Workspace Agent` right now.
     If yes, **do not** load
     the `Workspace Agent` prompt into the current Installation Agent
     session (that would reuse this session's context and is not what
     the user wants). Instead, open the top-level `Open Agent`
     launcher in a new OS-level window so it spawns a fresh harness
     process pointed at the now-switched prompt. Concretely:
     - Windows: `Start-Process -FilePath "<abs path to Open Agent.cmd>"`
       (or `cmd /c start "" "<abs path>"`); the `.cmd` launcher resolves
       Python and runs the dispatcher in its own console.
     - macOS: `open "<abs path to Open Agent.command>"`.
     - Linux: `xdg-open "<abs path to Open Agent.desktop>"` or
       `gtk-launch <basename>`; fall back to executing the launcher
       directly in a detached terminal (`setsid <launcher> &`) if
       neither is available.
     The Installation Agent session itself stays open in its original
     window so the user can review the install summary and see what
     changed. Do not push anything to a remote, and do not auto-launch
     without asking.
  10. Emit the final completion message: installation complete, next run
      starts workspace flow. This is the single "installation complete"
      sentence; do not print it earlier.
- Expected output artifacts: optional additional worker wrappers under
  `Scripts/Workers/` (the existing `Default` wrapper is left untouched);
  zero or more git repositories under `Workspaces/__template__/`, each
  with a resolvable `HEAD` commit, created from the user's answers in
  step 2 (zero is valid -- the user may have said "none"); updated
  `Workspaces/__template__/Workspace.md`; updated or stubbed
  `Workspaces/__template__/Framework.md` (with a backlog note when
  stubbed); updated `Workspaces/__template__/Prompts/Integration Agent.md`;
  top-level launcher switched to `Workspace Agent`; any
  integration-related edits from step 6 applied to the existing files
  named in that step (no new files or scripts outside the section 7
  manifest). The scaffolder-provided `Workspaces/__template__/Facts.md`
  stub (section 8.22) is left in place untouched by this agent.

### 8.3 `Prompts/Workspace Agent.md`

Headings: "Responsibilities", "Rules", "Notes".

Guidance:
- Responsibilities:
  - create a workspace via `Workspace - Create`,
  - archive a workspace via `Workspace - Remove --synced`,
  - **merge facts** on user request (see below),
  - open per-workspace agents by resolving the user's agent name to the
    exact per-workspace launcher file via this fixed mapping (no fuzzy
    lookup). Explain which launcher you are opening and for which
    workspace:
    - Integration -> `1. Open Integration Agent` launcher
    - Research -> `2. Open Research Agent` launcher
    - Planner -> `3. Open Planner Agent` launcher
    - Worker -> `4. Open Worker Agent` launcher
    - Reviewer -> `5. Open Reviewer Agent` launcher
  - The exact launcher filename is the mapped basename plus the OS-specific
    launcher extension for the installation OS: `.cmd` on Windows,
    `.command` on macOS, `.desktop` on Linux.
  - The launcher is opened by explicit path only:
    `Workspaces/<workspace>/<launcher-file>`. On Windows that means
    `Start-Process -FilePath "<absolute-launcher-path>"`. On other OSes
    invoke the launcher equivalently (see section 10).
  - **Merge facts** procedure (on explicit user request, and offered
    before archival per below):
    1. Enumerate live workspaces: every immediate subdirectory of
       `Workspaces/` whose name is **not** `__template__` and **not**
       `__archive__`. Workspace names are single-segment by contract, so
       recursive scanning is forbidden here. The `__archive__/` subtree is
       never read or written by this procedure.
    2. For each live workspace, read its `Facts.md` (skip silently if
       the file is absent or empty).
    3. Produce an updated `Workspaces/__template__/Facts.md` that
       semantically merges new and corrected `## <Title>` sections from
       the workspaces while honoring the version-control-friendly
       editing rules in section 8.22:
       - existing sections in the template keep their position; if a
         workspace has a corrected version of a section with the same
         title, edit only that section's body in the template; do not
         reorder, retitle, or reflow surrounding sections;
       - new titles found only in workspaces are appended to the end
         of the template in the order discovered;
       - never rename a section (titles are anchors); supersede with
         a new section if a title must change.
    4. After the template `Facts.md` is updated, **overwrite**
       `Facts.md` in every live workspace (same set as step 1) with
       the merged template content. The `__archive__` subtree is left
       untouched.
    5. Explain to the user what changed (which sections were added,
       which were revised, which workspaces contributed) in plain
       language. Do not commit, do not push.
  - **Before archival**: when the user asks to archive a workspace,
    first ask one yes/no question -- "Merge facts from workspaces into
    the template before archiving?" -- and run the merge procedure
    above if the user accepts. Then proceed to `Workspace - Remove
    --synced` regardless of the answer (declining the merge is not a
    reason to skip archival). Never run the merge silently or without
    confirmation.
- Rules:
  - one question at a time, explicit confirmation for archive, no remote
    push,
  - resolve "open <agent>" requests against the currently discussed
    workspace unless the user names a different one,
  - do not run broad file searches for scripts or launchers; use
    fixed workflow-root-relative paths (`Scripts/...`,
    `Workspaces/<workspace>/...`),
  - if the session starts inside a workspace folder, resolve the workflow
    root by walking upward to the directory that contains both `Scripts/`
    and `Workspaces/`, then use workflow-root-relative paths from there.
- Notes: keep explanations user-facing: say what you are doing, what happens
  next, and how the user can continue. Per-workspace work happens inside
  workspace prompts and markdown files; the global backlog/changelog files
  live in `Workspaces/Backlog.md` and `Workspaces/Changelog.md`. The same
  fixed launcher mapping is also the handoff mechanism used by every
  per-workspace agent when the user asks to **open** another workspace
  agent window (or to jump to a different workspace). In-chat role
  switching inside the same workspace is handled by the per-workspace
  prompts themselves per section 4.14.

### 8.4 `Workspaces/Backlog.md` and `Workspaces/Changelog.md`

Each contains a one-line description and an "Entry format" section listing
the fields used by the Integration Agent when syncing from workspaces:

- Backlog entry: Date, Workspace, Source, Follow-up.
- Changelog entry: Date, Workspace, Scope, Outcome.

### 8.5 Template `Workspace.md`

Headings: "Structure", "Workspace naming convention", "Branch convention",
"Repository layout", "Verification notes".

Guidance:
- Structure: short description of the prompt files, work queue files, and
  core artifacts inside the workspace.
- Workspace naming convention: short text describing how new workspace
  names are constructed. Workspace names are always single-segment
  identifiers under `Workspaces/`. The scaffolder writes the generic default
  (lowercase, path-safe, `feature-area-ticket-or-scope`); the
  Installation Agent rewrites this section with the user's actual
  convention (section 8.2 step 3).
- Branch convention: a single shared workspace branch name used by every
  repository in the workspace (default pattern
  `workspace/<workspace-name>`).
- Repository layout: one bullet per configured repository directory with a
  one-line purpose. The scaffolder writes a placeholder paragraph saying
  "No repositories configured yet; the Installation Agent will fill this
  section in."; installation rewrites this list to match the
  repositories the user actually added in section 8.2 step 2.
- Verification notes: a one-line pointer that build/test/run/verify commands
  live in `Framework.md` and must be kept current.

### 8.6 Template `Assignments.md`

One-paragraph explanation that this file is for free-form worker preferences
and that scripts do not read it. Add a short example list.

### 8.7 Template `Backlog.md` and `Changelog.md`

Per-workspace versions written from the workspace's point of view. They use
different fields than the global files in section 8.4 because the Integration Agent
is responsible for transforming local entries into the global shape when
syncing.

- Per-workspace `Backlog.md` entry format:
  `Priority`, `Title`, `Reason deferred`, `Suggested next step`.
- Per-workspace `Changelog.md` entry format:
  `Scope`, `Key changes`, `Verification`, `Risks/follow-up`.

Note in each file that the Integration Agent maps these entries into the global
`Workspaces/Backlog.md` and `Workspaces/Changelog.md` (which use the
`Date / Workspace / ...` shape from section 8.4).

### 8.8 Template `Framework.md`

Headings: "Navigation map", "Build / test / run / verify", "Automation
boundaries".

Guidance:
- If the user opted out of framework research, include a leading paragraph
  declaring the workspace as brownfield with no enforced design, and saying
  research may be needed for technical details.
- Otherwise fill in the high-level navigation map (no low-level details) and
  the build/test/run/verify commands.

### 8.9 Template `Issue.md`

Headings: "Required".

Required fields: ID, Title, Problem statement, Scope, Non-goals, Acceptance
criteria.

### 8.10 Template `Notes.md`

One short paragraph explaining what to put here (extra context, constraints,
preferences not captured elsewhere).

### 8.11 Template `Plan.md`

Headings: "Plan", "Risks", "Verification strategy".

Guidance: numbered plan steps, risk bullets, verification approach bullets.

### 8.12 Template `PR.md`

Headings: "Title", "Summary", "Validation", "Follow-ups".

Guidance: stub bullets under each, prepared for the Integration Agent to finalize.

### 8.13 Template `Research.md`

Default content: "No research." Plus a short note that the Research Agent
will rewrite this file as a free-form Q&A or technical log.

### 8.14 Template `Status.md`

Single markdown table with the columns from section 4.9 and one example row.
Below the table, a short "Notes" section captures two rules:
- Use this file to explain why any part is not at 100% completion.
- Out-of-scope items must be moved to the workspace's `Backlog`.

The Reviewer Agent owns updates to this file in practice, but the template
itself does not need to spell that out.

### 8.15 Template `Prompts/Integration Agent.md`

Headings: "Inputs", "Typical tasks", "Rules".

Guidance:
- Inputs: workspace `Issue`, `PR`, `Backlog`, `Changelog`, `Workspace`
  documents.
- Typical tasks: bring task details into `Issue`; update issue comments and
  status; prepare PR using `PR.md`; perform explicit git operations on
  request; sync workspace `Backlog` and `Changelog` into the global ones in
  `Workspaces/`; act as the adaptable integration surface for configured
  trackers/systems; keep a short external-tracker note in this prompt, either
  with the configured tracker details or with an explicit "not configured"
  statement. Explain git actions in plain language first, then run the
  explicit command if the user wants it. If the user asks to switch roles in
  the same workspace using section 4.14 command patterns, switch in-chat to
  the requested role and continue in the same session. If the user asks to
  perform work that is outside Integration scope, offer an in-chat switch to
  the best-fit workspace role per section 4.14 before declining. If the user
  asks to open another workspace agent window (or names a different
  workspace),
  resolve the exact launcher with the fixed mapping from section 8.3, explain
  which launcher you are opening for which workspace, and open it by
  explicit path.
- Rules: one question at a time, no push without explicit user request;
  resolve cross-agent handoffs against the current workspace unless the user
  names a different workspace.

### 8.16 Template `Prompts/Research Agent.md`

Headings: "Required behavior", "Optional tasks".

Guidance: mirror section 4.10. Ask for pre-context first, then ask one
question at a time, then write `Research`. Keep the conversation focused on
what is being investigated, what is known so far, and what will be looked at
next. Optionally update `Backlog`. If the user asks to switch roles in the
same workspace using section 4.14 command patterns, switch in-chat to the
requested role and continue in the same session. If the user asks to open
another workspace agent window (or names a different workspace), resolve the
exact launcher with the fixed mapping from section 8.3 against the current
workspace unless the user names a different workspace, explain which launcher
you are opening for which workspace, and open it by explicit path.
If the user asks for actions outside Research scope, offer an in-chat switch
to the best-fit workspace role per section 4.14 before declining.

Treat `Research.md` as a **living document, edited in place**, not as an
append-only log:
- When new evidence contradicts a previous finding, revise that finding
  directly (correct the wording, replace the answer, or remove a bullet
  that is now wrong). Do not let outdated answers linger next to the
  corrected ones.
- Reorganize sections when it helps clarity (group related answers,
  collapse duplicates), but keep the file readable as a Q&A or technical
  log rather than turning it into a dump.
- The file is tracked in version control: prefer minimal,
  paragraph-scoped edits over wholesale rewrites so diffs stay
  reviewable. Hard-wrap prose at a reasonable width and do not reflow
  unrelated paragraphs when editing one section.
- Confirmed durable facts that are likely to be re-used across tasks
  (project conventions, decisions, environment specifics) are recorded
  by the Planner Agent in `Facts.md` (section 8.22). The Research Agent
  may suggest a fact for `Facts.md` but does not append to it directly
  unless the user explicitly confirms; that is the Planner's role.

### 8.17 Template `Prompts/Planner Agent.md`

Headings: "Inputs", "Outputs", "Interview flow", "Existing plan policy",
"Work queue seeding", "Notes".

Guidance:
- Inputs: `Issue`, `Research`, `Notes`, `Changelog`, `Framework`,
  `Work/Next` (to see what is already queued). `Facts.md` exists in the
  workspace and is **searched on demand only** -- never read end-to-end
  and never preloaded. Before asking the user any question, search
  `Facts.md` for a matching `## <Title>` section by keyword; if a
  relevant section exists, treat its body as authoritative and skip
  that question.
- Outputs: `Plan`, `Status` (with the columns from section 4.9),
  `Work/Next` (the initial set of actionable chunks derived from the
  plan), and **appended sections** in `Facts.md` recording confirmed
  durable answers from the interview (see "Interview flow" below).
- Interview flow (applies when the user invokes the Planner without
  specific instructions, e.g. just says "plan"):
  1. Read the inputs above (excluding `Facts.md` -- search-only).
  2. Conduct an interview, **one question at a time**. Acknowledge each
     answer briefly before asking the next question.
  3. At any point the user may delegate a question back -- e.g. "find
     the answer" or "propose options". The Planner then researches or
     reasons about the question itself and proposes one or more options
     for the user to confirm or correct.
  4. For questions whose answer is obvious or low-risk, the Planner may
     proactively offer its own answers in a short batch and ask the
     user to confirm or correct them, instead of asking each question
     in turn.
  5. Never re-ask something already recorded in `Facts.md`. Search
     first by title or keyword; if the relevant section is present,
     use it as the answer.
  6. When the interview ends, append the confirmed durable answers to
     `Facts.md` as one or more new `## <Title>` sections per the
     append-only, VC-friendly rules in section 8.22. Do not edit
     existing sections; do not rename or reorder anything. Skip
     answers that are ephemeral or task-local (those belong in `Plan`,
     `Notes`, or `Issue`, not in `Facts.md`).
- Existing plan policy:
  - If `Plan.md` already exists, do not rewrite or replace it unless the
    user explicitly asks for a rewrite.
  - If the user asks to refine planning, append focused updates or edit only
    the requested sections.
  - If the user says not to change the plan, treat `Plan.md` as read-only
    and continue with discussion/review only.
  - Always confirm intent before replacing an existing plan wholesale.
- Work queue seeding:
  - After the `Plan` is in a stable state, derive actionable chunks
    into `Work/Next` (chunks separated by the literal line `---` per
    section 4.3). Each chunk should be a single, self-contained unit
    of execution scoped tightly enough that a Worker can act on it
    without further planning.
  - If `Work/Next` already has actionable blocks, do not regenerate
    them unless the user explicitly asks; offer to append additional
    chunks instead.
  - If `Work/Next` is empty, create initial actionable chunks
    immediately once the plan is stable.
  - Do not write to `Work/Current`, `Work/Blocked`, or `Work/Done`;
    those files are owned by `Work - Do` / `Work - Undo` (Worker
    Agent).
- Notes: `Research` may say "No research" and planning still proceeds.
  Keep the plan understandable at a glance: what will be done, in what
  order, and how success will be checked. Defer out-of-scope items to
  `Backlog`. Read only the minimum required files first and do not run broad
  cross-workspace discovery. If the user asks to switch roles in the same
  workspace using section 4.14 command patterns, switch in-chat to the
  requested role and continue in the same session. If the user asks to open
  another workspace agent window (or names a different workspace), resolve
  the exact launcher with the fixed mapping from section 8.3 against the
  current workspace unless the user names a different workspace, explain
  which launcher you are opening for which workspace, and open it by
  explicit path. If the user asks for actions outside Planner scope, offer
  an in-chat switch to the best-fit workspace role per section 4.14 before
  declining.

### 8.18 Template `Prompts/Worker Agent.md`

Headings: "Inputs", "Responsibilities", "Rules".

Guidance:
- Inputs: `Plan`, `Issue`, `Notes`, `Changelog`, `Research`, `Status`,
  `Framework`, `Assignments`, `Work/Next`. `Facts.md` exists in the
  workspace and is **searched on demand only** -- never read end-to-end
  and never preloaded. When a question about project conventions or
  prior decisions comes up, search `Facts.md` for a matching
  `## <Title>` section by keyword and read only that section.
- Responsibilities: invoke `Work - Do`; resolve `Blocked`/`Current`
  decisions via explicit CLI arguments; invoke `Work - Undo` when
  rollback is requested; refine work toward 100% in `Status` or
  propose backlog items; when the user asks to run all work, process
  tasks sequentially one by one. The Worker does **not** seed
  `Work/Next` -- that is the Planner Agent's responsibility
  (section 8.17). If `Work/Next` is empty when the user asks the
  Worker to start, instruct the user to run the Planner Agent first
  (or, if they explicitly ask, hand off to planning before executing).
  If the user asks to switch roles in the same workspace using section
  4.14 command patterns, switch in-chat to the requested role and continue
  in the same session. If the user asks to open another workspace agent
  window (or names a different workspace), resolve the exact launcher with
  the fixed mapping from section 8.3 against the current workspace unless
  the user names a different workspace, explain which launcher you are
  opening for which workspace, and open it by explicit path. If the user
  asks for actions outside Worker scope, offer an in-chat switch to the
  best-fit workspace role per section 4.14 before declining.
- After each task is verified and moved to `Work/Done`, create a local git
  commit for each changed repository before starting the next task. Commit
  only after successful verification; do not batch commits across tasks; the
  commit message references the completed task block intent; never push
  unless the user explicitly asks.
- When a durable, reusable fact is confirmed by the user during
  execution (a project convention, a decision, an environment detail
  worth carrying across tasks), append it to `Facts.md` as a new
  `## <Title>` section per section 8.22. Only append on explicit user
  confirmation; never edit or remove existing sections; ephemeral
  task-specific notes go in `Notes` or `Plan`, not `Facts`.
- Rules: one question at a time when clarifying; keep operations explicit
  and script-driven; workflow scripts live at workflow root `Scripts/`, not
  inside workspace directories (never propose creating `Work - Do` or
  `Work - Undo` inside a workspace); if the session starts in a workspace
  folder, locate the root scripts by walking upward to the directory that
  contains `Scripts/Work - Do.<ext>`; treat `Plan` and `Work/Next` as
  input, not output (do not edit `Plan` or seed/regenerate `Work/Next`
  unless the user explicitly asks -- the canonical path for new chunks
  is through the Planner Agent). Keep status updates practical: what was
  done, what remains, and what decision is needed next.

### 8.19 Template `Prompts/Reviewer Agent.md`

Headings: "Inputs", "Outputs", "Rules".

Guidance:
- Inputs: `Plan`, `Issue`, `Status`, `Work/Done`, repository changes.
- Outputs: `Changelog`, `PR`, optional updates to `Status`, `Notes`,
  `Backlog`.
- Rules: explain gaps in plain language; keep status updates concrete and
  measurable; tell the user what is ready, what is still missing, and what
  follow-up is recommended. If the user asks to switch roles in the same
  workspace using section 4.14 command patterns, switch in-chat to the
  requested role and continue in the same session. If the user asks to open
  another workspace agent window (or names a different workspace), resolve
  the exact launcher with the fixed mapping from section 8.3 against the
  current workspace unless the user names a different workspace, explain
  which launcher you are opening for which workspace, and open it by
  explicit path. If the user asks for actions outside Reviewer scope, offer
  an in-chat switch to the best-fit workspace role per section 4.14 before
  declining.

### 8.20 Template `Prompts/Work - Execute.md`

Headings: "Inputs", "Required behavior", "Output expectation".

Guidance:
- Inputs: the current work chunk and the artifacts listed in section 4.5.
  `Facts.md` exists in the workspace but is **not auto-attached** to the
  dispatcher's `--context-files`; when a relevant project convention or
  prior decision is needed, search `Facts.md` for a matching
  `## <Title>` section by keyword and read only that section.
- Required behavior:
  - apply changes scoped to the active chunk's intent;
  - report what was changed and what remains;
  - do not update workflow coordination files. Treat the following as
    read-only context during execution: `Status`, `Plan`, `Research`,
    `Notes`, `Assignments`, `Issue`, `PR`, `Changelog`, `Backlog`,
    `Facts`, and every file under `Work/`.
- Output expectation: clear execution result focused on what was done, what
  remains, and, if blocked, the blocker and the suggested next action.

### 8.21 Template `Prompts/Work - Verify.md`

Headings: "Inputs", "Required behavior", "Output expectation".

Guidance:
- Inputs: same as Execute. `Facts.md` is likewise present but not
  auto-attached; search by section title for any convention or decision
  needed to judge whether the change is acceptable.
- Required behavior: validate that execution matches chunk intent; run
  verification steps relevant to the change.
- Output expectation: the last non-empty line of the verifier's output must
  be exactly `PASS` when the work is acceptable for moving to `Done`, or
  exactly `FAIL: <one-line explanation>` otherwise. `PASS` and the `FAIL: `
  prefix are literal ASCII protocol tokens and must not be translated
  (see section 5.1); only the free-text explanation after `FAIL: ` is in
  the chosen language. `Work - Do` parses that line; anything else is
  treated as an invalid verifier result and routed the same way as `FAIL`
  (see section 4.5). When the verifier reports `FAIL` (or produces an
  invalid result), `Work - Do` records the reason inside the chunk via
  the blocked-reason header from section 4.4a, so the user can read
  `Work/Blocked.md` and immediately see why each chunk is there without
  consulting logs.

### 8.22 Template `Facts.md`

Durable, agent-curated record of confirmed facts about the workspace
and its project (project conventions, decisions, environment specifics,
resolved ambiguities). Primarily written by the Planner Agent at the
end of its interview (section 8.17); the Worker Agent may also append
facts that the user explicitly confirms during execution
(section 8.18). The file ships as a stub created by the scaffolder and
is included in the per-workspace template so every workspace gets a
copy on `Workspace - Create`.

Format: an append-only sequence of level-2 sections. Each section is
exactly:

```
## <Short Title>

<body: paragraphs or bullet points>
```

with one blank line between the heading and the body, and one blank
line separating each section from the next. There is no automatic
deduplication, no reordering, and no rewriting of existing sections by
any agent except during the explicit Workspace-Agent-driven merge
described in section 8.3.

Read pattern: agents must **search/grep** `Facts.md` for a relevant
`## <Title>` or keyword and read only the matching section(s). They
must not read the file end-to-end. `Facts.md` is intentionally
excluded from the dispatcher's `--context-files` for the same reason
(sections 4.5 and 9.4).

Version-control friendliness (the file is tracked in git): writes must
produce minimal, line-stable diffs.

- UTF-8 encoding, LF line endings, single trailing newline at EOF.
- No trailing whitespace on any line.
- Each section is exactly one `## <Title>` heading line, one blank
  line, the body, then one blank line before the next section.
- Hard-wrap soft prose at a reasonable width (~100 columns) so prose
  edits diff line-by-line. Do not reflow unrelated paragraphs when
  editing a single section.
- Appends touch only the tail of the file; existing bytes are
  unchanged.
- The explicit Workspace-Agent merge (section 8.3) preserves the order
  of unchanged sections; revisions edit only the affected section's
  body and never reorder, retitle, or reflow surrounding sections.
- Titles are stable identifiers: once a section is written, its
  `## <Title>` line is the anchor used by merges and by on-demand
  search. Do not rename a section in place; if a title must change,
  supersede it with a new section appended at the end (the old
  section can later be removed during a merge if it is genuinely
  obsolete, but never silently retitled).

Stub content: one short introductory paragraph explaining the purpose,
the append-only `## <Title>` format, and the search-only read rule,
plus one example `## Example fact` section the user can replace or
delete.

### 8.23 Template `Work/Next.md`, `Work/Current.md`, `Work/Blocked.md`, `Work/Done.md`

All four work-queue files in the template are empty files. They exist so the
workspace structure is complete on copy, but they carry no instructional
text or example content. The chunk format (section 4.3) and rollback header
format (section 4.4) are the authoritative reference; do not duplicate them
into these files.

---

## 9. Per-script behavioral specs

For each script: argument list, behavior, exit codes, side effects. Translate
to the chosen programming language. Use plain standard-library code.

### 9.1 Dispatcher (`Agent`)

Inputs: as in section 4.1.

Behavior:

```
def main():
    args = parse_canonical_args()
    if not file_exists(args.prompt):
        print_error("Prompt not found: " + args.prompt)
        return 2
    try:
        worker_path = resolve_worker(args.worker)  # folder-restricted
    except invalid:
        print_error(...)
        return 2
    cmd = [interpreter, worker_path,
           "--prompt", absolute(args.prompt),
           "--mode", args.mode]
    if args.workspace: cmd += ["--workspace", absolute(args.workspace)]
    if args.tail:      cmd += ["--tail", args.tail]
    if args.agent_name:cmd += ["--agent-name", args.agent_name]
    if args.context_files:
        cmd += ["--context-files", normalize_csv_absolute(args.context_files)]
    if args.dry_run:   cmd += ["--dry-run"]
    if args.new_window:cmd += ["--new-window"]
    return subprocess_run(cmd).exit_code
```

Exit codes: 0 success, 2 user error (bad prompt or worker), worker's own
exit code otherwise.

### 9.2 `Workspace - Create`

Inputs: as in section 4.7.

Behavior:

```
def main():
    args = parse()
  workspace = normalize_flat_workspace_name(args.workspace)
  if not workspace: return 2
  target = workspaces_root / workspace
  if target.exists(): return 2
    create_directory(target)
    repos = immediate_subdirs_with_git(template_root)
    for repo in repos:
        if not git_has_head(repo):  # git rev-parse --verify HEAD
            print_error(f"Repo {repo.name} has no HEAD commit. "
                        "Run `git commit --allow-empty -m init` in it and rerun.")
            return 2
    repos_names = {r.name for r in repos}
    for child in template_root.children:
        if child.name in repos_names: continue
        if is_launcher(child): continue
        copy(child, target / child.name)
    for repo in repos:
        current = git_current_branch(repo)
        if has_upstream(repo, current):
            git_pull_ff_only(repo)
        if branch_exists(repo, args.branch):
            git_worktree_add(repo, target/repo.name, args.branch)
        else:
            git_worktree_add_new_branch(repo, target/repo.name, args.branch, "HEAD")
    create_per_agent_launchers(target)
    print_success(target)
    return 0
```

Exit codes: 0 success, 2 user/precondition error, propagate git failures.

### 9.3 `Workspace - Remove` (archive)

Inputs: as in section 4.8.

Behavior:

```
def main():
    args = parse()
  workspace = normalize_flat_workspace_name(args.workspace)
  if not workspace: return 2
    if not args.synced:
        print_error("Refusing to archive without --synced. "
            "Run Integration Agent sync first.")
        return 2
  source = workspaces_root / workspace
  if not source.exists(): return 2
    repos = immediate_subdirs_with_git(source)
    primaries = {repo: worktree_primary_repo(repo) for repo in repos}
    dirty = [repo for repo in repos if git_status_porcelain(repo) != ""]
    if dirty and not args.include_uncommitted:
        print_error("Uncommitted changes in: " + ", ".join(r.name for r in dirty))
        return 2
  dest = archive_root / workspace
    if dest exists: dest = dest.parent / (dest.name + "__" + timestamp())
    mkdir(dest)
    for child in source.children:
        if child in repos: continue
        move(child, dest / child.name)
    for repo in repos:
        if args.include_uncommitted and repo in dirty:
            copytree(repo, dest/repo.name, ignore=[".git"])
        primary = primaries[repo]
        if primary is not None:
            git_worktree_remove(primary, repo, force=args.include_uncommitted)
        else:
            rmtree(repo)
    for primary in unique(p for p in primaries.values() if p):
        git_worktree_prune(primary)   # warning only on failure
    try_rmdir(source)                 # best effort
    print_success(dest)
    return 0
```

Helpers:
- `worktree_primary_repo(repo)`: if `<repo>/.git` is a file, parse its
  `gitdir:` value. The value may be absolute or relative; resolve
  relative paths against the directory containing the `.git` file (i.e.
  `<repo>/`) before pattern-matching. If the resolved path matches
  `<primary>/.git/worktrees/<name>`, return `<primary>`. Otherwise
  return `None`.
- `git_worktree_remove(primary, repo, force)`: run
  `git -C <primary> worktree remove <repo> [--force]`.
- `git_worktree_prune(primary)`: run `git -C <primary> worktree prune`.

Exit codes: 0 success, 2 user/precondition error (missing `--synced`,
reserved path, missing source, dirty repos without `--include-uncommitted`),
propagate git failures from worktree removal.

### 9.4 `Work - Do`

Inputs: as in section 4.5.

Behavior follows the decision order in section 4.5 step by step. Provide a
chunk parser that:

- Reads text as UTF-8.
- Splits on lines equal to `---` after stripping trailing whitespace.
- Strips each chunk's surrounding whitespace.
- Filters out empty chunks.

Writer:

- Joins chunks with `\n---\n\n`.
- Ends file with a single trailing newline.
- Writes an empty file when the list is empty.

Rollback header helpers:

- `parse_header(chunk)` returns `(mapping, body_without_header)`.
- `add_header(chunk, mapping)` merges mapping with any existing one, sorts
  keys case-insensitively, and produces the exact line from section 4.4.

Dispatcher invocation: spawn the interpreter on `Scripts/Agent.<ext>` with
the canonical arguments. Build `--context-files` from the existing workspace
artifacts listed in section 4.5. **Never include `Facts.md`** in this
list even when the file exists; Facts is a search-on-demand reference
(see section 8.22). The verifier dispatcher call is always
made with `--mode cli` (regardless of the mode the caller of `Work - Do`
requested) and captures its stdout so the trailing `PASS` / `FAIL: <text>`
line can be parsed; stderr may be streamed live. The executor call uses
the caller's requested mode and may stream output live.

Verification outcome handling:

- Dispatcher non-zero exit on the verifier call -> move current chunk to
  `Blocked` (rollback header preserved, blocked-reason header added with
  kind `DISPATCHER`) and return the dispatcher's exit code.
- Verifier stdout last non-empty line equals `PASS` -> move current chunk
  to `Done` (rollback header preserved).
- Last non-empty line starts with `FAIL: ` and has non-empty text after the
  prefix -> move current chunk to `Blocked` (rollback header preserved,
  blocked-reason header added with kind `FAIL` and the verbatim text after
  `FAIL: ` as the reason) and return exit code 3.
- Anything else (no output, missing prefix, empty FAIL text, etc.) is
  treated as `INVALID` -> move to `Blocked` (rollback header preserved,
  blocked-reason header added with kind `INVALID`) and return exit code 3
  with a message that the verifier output did not end with `PASS` or
  `FAIL: <text>`.

The blocked-reason header is stripped again when a blocked chunk is
moved out (to `Current` or `Done`); see section 4.4a.

Exit codes: 0 success, 2 user/precondition error, 3 verification
FAIL/INVALID, dispatcher's exit code on worker failure.

### 9.5 `Work - Undo`

Inputs: as in section 4.6.

Behavior follows section 4.6 step by step. Provide:

- `git_commit_exists(repo, hash)` via `git cat-file -e <hash>^{commit}`.
- `git_reset_hard(repo, hash)` via `git reset --hard <hash>`.
- `git_clean_fdx(repo)` via `git clean -fdx`.
- Strip header before writing chunks back to `Next`.

Exit codes: 0 success, 2 user/precondition error including preflight
failures.

### 9.6 `Workers/Default`

Inputs: as in section 4.2.

Behavior:

```
def main():
    args = parse()
    bootstrap = "Read the file at '" + absolute(args.prompt) + \
                "' and follow the instructions exactly."
  if args.tail and args.tail.strip():
    bootstrap += "\n\n" + tail_prefix(language) + "\n" + args.tail
    binary = resolve_harness_binary("opencode")  # or chosen harness
    if binary is None:
        print_error("Could not start '<harness>'. Install it or update this wrapper.")
        return 127
    cmd = build_harness_command(binary, args.mode, bootstrap)
    for path in parse_csv(args.context_files):
        cmd += harness_attach_flags(path)        # may be []
    if args.dry_run:
        print(escape_for_shell(cmd)); return 0
    if args.new_window and args.mode == "tui" and is_windows():
        spawn_detached_new_console(cmd, cwd=args.workspace or cwd)
        return 0
    try:
        return subprocess_run(cmd, cwd=args.workspace or cwd).exit_code
    except missing_binary:
        print_error("Could not start '<harness>'.")
        return 127
```

`resolve_harness_binary(name)`:
- First try a `which` equivalent.
- On Windows, if not found, try `%APPDATA%\npm\<name>.cmd`.
- Return `None` if nothing works.

`tail_prefix(language)` returns the translated form of:
`After reading the files above, the user asked you to do this:`

`spawn_detached_new_console(cmd, cwd)` starts the harness in a new console
window detached from the launching process. On Python/Windows this is
`subprocess.Popen(cmd, cwd=cwd, creationflags=CREATE_NEW_CONSOLE)`. The
wrapper returns 0 immediately without waiting on the child.

Working directory for the harness subprocess is the resolved `--workspace`
path when provided; otherwise the wrapper's current working directory.

### 9.7 `WorkflowLog`

A tiny shared utility imported by the workflow scripts (`Work - Do`,
`Work - Undo`, `Workspace - Create`, `Workspace - Remove`). It exposes one
function:

- `append_workspace_log(workspace, event, message)`: append a single line
  `[YYYY-MM-DD HH:MM:SS] <event>: <message>\n` to `<workspace>/log.txt`
  (UTF-8). Any I/O failure is swallowed silently; logging must never break a
  workflow script.

  Important side effect: this helper creates the workspace directory (and
  any missing parents) on first write so logging works for freshly-created
  workspaces. Callers that branch on whether the workspace directory
  already exists (notably `Workspace - Create`, section 4.7 step 1) must
  perform that existence check **before** emitting any log line for the
  workspace; otherwise the directory created by the logger will mask a
  pre-existing workspace and the guard will never trip.

Canonical event names per script (best-effort breadcrumbs; the file is
plain text and is not parsed by any script, it exists for human
inspection):

- `Work - Do`: `work-do.start`, `work-do.idle`, `work-do.blocked`,
  `work-do.current`, `work-do.queue`, `work-do.execute`, `work-do.verify`,
  `work-do.done`.
- `Work - Undo`: `work-undo.start`, `work-undo.idle`,
  `work-undo.preflight`, `work-undo.rollback`, `work-undo.done`.
- `Workspace - Create`: `workspace-create.init`,
  `workspace-create.pull` (emitted before each `git pull --ff-only` on a
  template repo whose current branch has a configured upstream; the
  message names the repo and the branch being fast-forwarded),
  `workspace-create.worktree`, `workspace-create.done`.
- `Workspace - Remove`: `workspace-archive.start`,
  `workspace-archive.blocked`, `workspace-archive.snapshot`,
  `workspace-archive.worktree`, `workspace-archive.done`.

---

## 10. OS launcher recipes

Pick the recipe that matches the chosen OS. Each launcher invokes the
dispatcher with the appropriate prompt path and worker.

The dispatcher is invoked with the project's interpreter for the chosen
programming language (for example: `py` on Windows for Python, `python3` on
macOS/Linux, `bash` for shell scripts, `node` for Node.js). On Windows,
the launcher is a `.cmd` file that resolves the interpreter as described
in section 10.1 before invoking the dispatcher.

### 10.1 Windows: `.cmd` launchers

Create plain-text `.cmd` launchers. For Python-based distributions, each
launcher must resolve Python in this exact order before invoking the
dispatcher: `py` on `PATH`, then `python` on `PATH`, then
`%LOCALAPPDATA%\Programs\Python\Launcher\py.exe`.

```
@echo off
setlocal

set "PYTHON_CMD="
where py >nul 2>nul && set "PYTHON_CMD=py"
if not defined PYTHON_CMD where python >nul 2>nul && set "PYTHON_CMD=python"
if not defined PYTHON_CMD if exist "%LOCALAPPDATA%\Programs\Python\Launcher\py.exe" set "PYTHON_CMD=%LOCALAPPDATA%\Programs\Python\Launcher\py.exe"
if not defined PYTHON_CMD (
  >&2 echo Could not find Python. Tried: py on PATH, python on PATH, %%LOCALAPPDATA%%\Programs\Python\Launcher\py.exe
  exit /b 127
)

cd /d "%~dp0"
call "%PYTHON_CMD%" "<abs path to Scripts/Agent.<ext>>" --worker "Default" --prompt "<prompt path>" [--workspace "<path>"] --mode tui --agent-name "<Agent Name>" --new-window
exit /b %ERRORLEVEL%
```

Notes:
- Windows launchers must not hardcode user-profile absolute paths to the
  Python interpreter or launcher.
- For Python-based distributions, use the detection order above exactly.
- If the chosen programming language is not Python, keep the same `.cmd`
  launcher shape and working-directory semantics, but replace the Python
  detection block with the idiomatic runtime invocation for the chosen
  language.
- Top-level `Open Agent.cmd` uses an absolute prompt path to
  `Prompts/Installation Agent.md` initially. After installation switch it to
  `Prompts/Workspace Agent.md` (same launcher file, only the prompt path
  changes).
- Per-workspace launchers use a relative prompt path (`Prompts/<Agent>.md`)
  and `--workspace "."`. The launcher itself must change to its own
  directory first (`cd /d "%~dp0"`) so relative paths resolve from the
  workspace path.

### 10.2 macOS: `.command` scripts

Each launcher is an executable shell script with a `.command` extension so
the user can double-click it from Finder.

```
#!/usr/bin/env bash
set -euo pipefail
cd "$(dirname "$0")"
exec "<interpreter>" "<abs path to Scripts/Agent.<ext>>" \
  --worker "Default" \
  --prompt "<prompt path>" \
  --workspace "." \
  --mode tui \
  --agent-name "<Agent Name>"
```

Make the file executable (`chmod +x`). For the top-level launcher omit the
`--workspace` argument and use an absolute prompt path.

### 10.3 Linux: `.desktop` entries

```
[Desktop Entry]
Type=Application
Name=<launcher label>
Exec=<interpreter> "<abs path to Scripts/Agent.<ext>>" --worker "Default" --prompt "<prompt path>" --workspace "<workspace path or omit>" --mode tui --agent-name "<Agent Name>"
Path=<working directory>
Terminal=true
```

Mark the file executable. For users who run from a terminal, you may
additionally provide a plain shell wrapper similar to the macOS recipe.

### 10.4 Switching the top-level launcher after installation

At the end of installation, regenerate or edit the top-level launcher so its
prompt argument points to `Prompts/Workspace Agent.md` (absolute path). The
launcher file name does not change. Keep the dispatcher, worker, mode, and
working directory semantics identical; metadata such as `--agent-name` may be
left unchanged or updated to match the new prompt.

The Installation Agent must perform this switch through the OS-native
launcher mechanism. On Windows, macOS, and Linux the launcher artifacts in
section 10 are plain text, so the switch is performed by regenerating the
launcher file with the new prompt target while preserving its runtime
resolution logic and working-directory semantics.

**Windows (`.cmd`).** Rewrite `Open Agent.cmd` using the recipe from
section 10.1 so it still resolves Python in the same order and now points
at `Prompts/Workspace Agent.md`:

```
@echo off
setlocal

set "PYTHON_CMD="
where py >nul 2>nul && set "PYTHON_CMD=py"
if not defined PYTHON_CMD where python >nul 2>nul && set "PYTHON_CMD=python"
if not defined PYTHON_CMD if exist "%LOCALAPPDATA%\Programs\Python\Launcher\py.exe" set "PYTHON_CMD=%LOCALAPPDATA%\Programs\Python\Launcher\py.exe"
if not defined PYTHON_CMD (
  >&2 echo Could not find Python. Tried: py on PATH, python on PATH, %%LOCALAPPDATA%%\Programs\Python\Launcher\py.exe
  exit /b 127
)

cd /d "%~dp0"
call "%PYTHON_CMD%" "<abs path to Scripts/Agent.<ext>>" --worker "Default" --prompt "<abs path to Prompts/Workspace Agent.md>" --mode tui --agent-name "Workspace Agent" --new-window
exit /b %ERRORLEVEL%
```

The launcher file name stays identical. Preserve the Python resolution
order, `cd /d "%~dp0"`, and the rest of the dispatcher arguments; update
only the prompt target and optional `--agent-name` metadata. Preserve
`--new-window` when the Windows launcher uses it.

**macOS (`.command`) and Linux (`.desktop`).** These are plain text files;
regenerate them via the recipe in section 10.2 or 10.3 with the new prompt
path, overwriting the existing file. Preserve the executable bit
(`chmod +x`) after rewriting on macOS/Linux.

After switching, the Installation Agent must verify the launcher still
resolves to a valid harness command (per section 8.2 step 7) before
reporting installation complete.

---

## 11. Harness adaptation guide

Bootstrap pattern (fixed, see section 4.2):

- Bootstrap line 1: `Read the file at '<absolute prompt path>' and follow the instructions exactly.`
- If tail is non-empty: add a blank line, the translated tail prefix, a
  newline, then the tail text.

The harness command surrounding the bootstrap depends on the harness. Two
examples follow.

### 11.1 Opencode

- CLI/unattended: `opencode run "<bootstrap>"`
- TUI/interactive: `opencode --prompt "<bootstrap>"`
- Context attach: for each path in `--context-files`, append
  `--context <path>` to the command. If a build of opencode does not
  recognize the flag, drop it or ignore silently.

Resolution: try `which opencode`. On Windows, npm installs leave shims under
`%APPDATA%\npm\opencode.cmd`; fall back to that path before failing.

### 11.2 Claude Code (example)

Not authoritative -- these are illustrative defaults. Always re-derive
from `claude --help` at scaffold time per interview question 5.

- CLI/unattended: `claude -p "<bootstrap>"` (the `-p` / `--print` flag
  runs Claude Code in non-interactive print mode).
- TUI/interactive: `claude "<bootstrap>"` (positional initial prompt;
  starts the interactive session).
- Context attach: Claude Code does not take ad-hoc file flags from the
  CLI; mention the files inside the bootstrap text or drop the
  `--context-files` hint.

Resolution: try `which claude`. Adjust to your local installation.

### 11.3 Adding a new harness

Ask the user for:
- the binary name and an OS-specific fallback path if applicable,
- the unattended invocation pattern,
- the interactive invocation pattern,
- whether the harness supports attaching files (and if so, the flag).

Build a worker wrapper that follows the structure in section 9.6 and the
bootstrap shape in section 4.2.

---

## 12. Programming language adaptation guide

Whatever language you pick, the standard library must offer:

- Argument parsing with long flags and required-value support.
- Subprocess execution with argv arrays (no shell interpolation).
- Filesystem operations: exists, mkdir, copy file, copy tree, move, rename.
- Path normalization to absolute paths.
- UTF-8 text read and write.
- Date/time formatting for the archive timestamp suffix (`YYYY-MM-DD_HHMMSS`).
- Running git via subprocess; you do not need a git library.

For Python: use `argparse`, `subprocess`, `shutil`, `pathlib`, `datetime`.
For Bash: use `getopts` or manual parsing, `cp -r`, `mv`, `mktemp`, `git`.
For Node.js: use `process.argv` parsing (or `node:util` `parseArgs`),
`child_process.spawnSync`, `fs/promises`, `path`.
For Go: use `flag`, `os/exec`, `io/fs`, `path/filepath`, `time`.

---

## 13. Verification checklist

After scaffolding, run these checks. Do not ask the user; do them yourself
and report results.

0. **Cleanup pass.** Before running the remaining checks, walk the entire
   workflow root and delete every file or directory that is not part of
  the file manifest in section 7 and was created by scaffolding,
  verification, or rehearsal. Preserve any pre-existing user workspaces
  under `Workspaces/` other than throwaway rehearsal paths created by this
  run. This explicitly includes:
   - any scratch or test files you created while writing or verifying
     scripts (e.g. `test_*.py`, `tmp.md`, sample inputs, throwaway
     `Workspaces/<name>/` directories used to dry-run create/remove);
   - any backup or editor artifacts (`*.bak`, `*~`, `.DS_Store`,
     `__pycache__/`, `*.pyc`, `node_modules/`, `.cache/`, etc.);
   - any extra documentation files you produced beyond the manifest
     (summaries, change notes, READMEs inside subdirectories);
   - any partially generated files left over from aborted attempts.
   The only exceptions are: this `INSTALL.md` (consumed input), and the
   files explicitly listed in section 7. The four template
   `Work/*.md` files and the empty `Workspaces/__archive__/` directory
   are part of the manifest and must remain. The `Workspaces/__template__/`
   directory itself ships with no repository directories; any
   `Configuration/`, `Documentation/`, `Implementation/`, or other
   repo directories the rehearsal (section 13.5) or a previous aborted
   run may have left behind must be removed in this pass. After this
   pass, the manifest-managed portions of the workflow root tree must
   match section 7 exactly, with no extra scaffolder-created artifacts.

1. Every file from section 7 exists. Every manifest file that is specified
   to carry content must be non-empty; the four template `Work/*.md` queue
   files may be zero-byte by design.
2. All generated scripts are syntactically valid (compile, parse, or
   lint depending on the language).
3. Dispatcher dry-run with the Installation Agent prompt prints a valid
   harness command. Example: invoke the dispatcher with
   `--worker Default --prompt <abs Installation Agent path> --mode cli --dry-run`.
4. Each per-agent launcher has the expected content and working-directory
  semantics per section 10. On Windows with Python launchers, validate that
  each `.cmd` file resolves Python in this order: `py` on `PATH`, then
  `python` on `PATH`, then `%LOCALAPPDATA%\Programs\Python\Launcher\py.exe`,
  and that it invokes the dispatcher without hardcoding a user-profile
  absolute path.
5. `Work - Do` chunk parser round-trip: feed the parser a synthetic input
   already in the writer's canonical form (two non-empty chunks joined by
   `\n---\n\n`, ending with a single trailing newline), write it back via
   the writer, and confirm the output is byte-equivalent to the input.
   Non-canonical inputs are expected to be normalized by the writer
   (separator spacing, trailing whitespace) and are not part of this
   round-trip check. (The template `Work/*.md` files are empty by design
   and are not used for this check.)
6. `Workspace - Remove` refuses to run without `--synced`.
7. `Work - Do` refuses to run when `Blocked` or `Current` is non-empty
   without the matching action flag.
8. End-to-end UTF-8: when the chosen natural language uses non-ASCII
   characters, run the dispatcher in `--dry-run` against the Installation
   Agent prompt and confirm the printed harness command, including the
   translated bootstrap sentence, appears correctly (no mojibake, no
   `UnicodeEncodeError`). Also append a non-ASCII test line via
   `WorkflowLog` to a scratch workspace and confirm the resulting
   `log.txt` reads back as UTF-8.

9. `Workspaces/__template__/Facts.md` exists and is non-empty, contains
   the stub explanation and at least one `## <Title>` example section,
   ends with a single trailing newline, and uses LF line endings with
   no trailing whitespace (section 8.22).

10. `Work - Do` context-file selection excludes `Facts.md`. Create a
    scratch workspace, write a non-empty `Facts.md` next to the usual
    artifacts, and confirm a `Work - Do --dry-run` invocation produces
    a dispatcher command whose `--context-files` value does **not**
    contain `Facts.md` even though the file exists (sections 4.5, 4.11,
    9.4). Remove the scratch workspace in step 11.

11. Post-check cleanup: remove any synthetic files and scratch workspaces
  created by verification steps 5, 8, and 10 so the tree returns to the
  post-scaffold state expected by step 0.

12. In-chat role switching coverage in prompt specs: verify that all five
  per-workspace prompts (`Integration Agent`, `Research Agent`, `Planner
  Agent`, `Worker Agent`, `Reviewer Agent`) explicitly implement section
  4.14 behavior for `switch to <role>` and `become <role>` in `--mode tui`.

13. Role-target safety checks: verify the specs reject targets outside the
  allowed five roles and explicitly reject `Installation Agent`,
  `Workspace Agent`, `Work - Execute`, and `Work - Verify` as in-chat
  switch targets.

14. Same-role no-op check: verify prompt guidance defines a deterministic
  no-op response when the user requests a switch to the current role.

15. Launcher-vs-in-chat distinction check: verify each per-workspace prompt
  distinguishes in-chat same-workspace switching (section 4.14) from
  launcher-based opening (section 8.3) when the user asks for a separate
  window or a different workspace.

16. Role normalization check: verify prompt guidance uses one deterministic
  normalization table (section 4.14) so case and optional `agent` suffix
  resolve to the same canonical role keys.

17. Mixed-intent precedence check: verify prompt guidance defines what to do
  when one message contains both switching and task instructions (switch
  first, then continue under the new role), and what to do when multiple
  different switch targets appear in one message (ask one clarification,
  no switch yet).

18. Capability-mismatch offer check: verify prompt guidance requires the
  current role to offer an in-chat switch when the user asks for an action
  outside that role's responsibilities but inside another workspace role,
  with deterministic handling for one clear target vs multiple plausible
  targets (section 4.14).

If any check fails, repair before handoff.

### 13.5 Optional end-to-end rehearsal

Run this protocol only if the user opted in at interview question 6.
Its purpose is to exercise every script and launcher wiring against a
throwaway workspace and to roll the workflow root back to its
post-scaffold state when finished. The rehearsal must leave **no
artifacts** behind: no extra files, no extra git history in template
repos, no extra entries in `__archive__/`, and no change to the
top-level launcher.

Harness calls are stubbed throughout: every dispatcher invocation that
would normally spawn the AI harness is made with `--mode cli --dry-run`,
so no tokens are spent and no model round-trips happen. Real subprocess
calls are limited to the workflow scripts themselves and to `git`.

Protocol:

1. **Snapshot.** Copy the entire workflow root (every file and
   directory) to a sibling temp directory, for example
   `<workflow-root>/../<workflow-root-name>.rehearsal-backup-<timestamp>/`.
   This is the rollback target; it must be a faithful copy that
   preserves file modes, the executable bit on launchers, and the full
   contents of every git repo (including `.git/`). If the snapshot
   cannot be created, skip the rehearsal and report the reason; do
   not attempt the rehearsal without a working rollback.

2. **Prepare repos.** In `Workspaces/__template__/`, create three
   throwaway local-only git repos named `RehearsalA/`, `RehearsalB/`,
   `RehearsalC/` with one empty initial commit each (`git init`,
   `git commit --allow-empty -m rehearsal`). These exist only for the
   rehearsal and must be deleted by step 9's restore; they do **not**
   mirror anything the scaffolder ships (the scaffolder ships an empty
   template). No upstream is configured; everything stays local.

3. **Switch the launcher (simulated).** Update the top-level `Open
   Agent` launcher so its prompt argument points to
   `Prompts/Workspace Agent.md`, exactly the operation described in
   section 10.4. Verify it parses.

4. **Create a workspace.** Invoke
  `Scripts/Workspace - Create --workspace rehearsal-smoke --branch
  workspace/rehearsal-smoke`. Assert:
  - the workspace directory exists under `Workspaces/rehearsal-smoke/`,
   - the three rehearsal repos are attached as worktrees on the requested branch,
   - the five per-agent launchers exist and parse,
   - the per-workspace artifacts (`Issue.md`, `Plan.md`, ..., `Work/`)
     were copied from the template.

5. **Queue a synthetic task and run `Work - Do`.** Write a single
  chunk into `Workspaces/rehearsal-smoke/Work/Next.md` (something
  trivial, e.g. "touch a file in RehearsalA/"). Invoke
  `Scripts/Work - Do --workspace Workspaces/rehearsal-smoke --mode cli
   --dry-run`. Assert:
   - the chunk moved out of `Next.md`,
   - `Current.md` contains the chunk with a valid rollback header
     (section 4.4) carrying real commit hashes from the three rehearsal repos,
   - the dispatcher was invoked with the execute prompt and the verify
     prompt (both as dry runs),
   - because `--dry-run` was set, the chunk stayed in `Current.md` per
     section 4.5 step 4 (skip verifier-output parsing; do not move the
     current chunk).

6. **Exercise `--current-action to-done`.** Re-invoke `Work - Do` with
   `--current-action to-done`. Assert the chunk now lives in
   `Done.md` with its rollback header intact.

7. **Exercise `Work - Undo`.** Invoke
   `Scripts/Work - Undo --workspace Workspaces/rehearsal-smoke --count
   1`. Assert:
   - the chunk returned to `Next.md` without its rollback header,
   - each repo's `HEAD` matches the hash that was captured in the
     rollback header (preflight succeeded, reset worked).

8. **Archive the workspace.** Invoke
   `Scripts/Workspace - Remove --workspace rehearsal-smoke --synced`.
   Assert:
   - the workspace directory is gone,
   - the non-repo files landed under
     `Workspaces/__archive__/rehearsal-smoke/`,
   - the three worktree pointers were removed (`git worktree list` in
     each template repo no longer mentions the rehearsal path),
   - the script refused the call earlier when `--synced` was omitted
     (re-run a separate negative case to confirm exit code 2).

9. **Restore.** Delete every child of the workflow root **except**
   `INSTALL.md` (the user-supplied input), then copy the snapshot from
   step 1 back into place verbatim. Verify by spot-checking that:
   - the rehearsal repo directories (`RehearsalA/`, `RehearsalB/`,
     `RehearsalC/`) are gone from `Workspaces/__template__/`,
   - `__archive__/` is empty,
   - the top-level launcher targets `Prompts/Installation Agent.md`
     again,
   - no `rehearsal-smoke` workspace remains.
   After restoration, delete the snapshot directory.

10. **Report.** Print a short summary listing each of steps 2-8 as PASS
    or FAIL with the exit code observed and any captured error. If any
    step failed, leave the snapshot intact and tell the user where it
    lives so they can inspect it manually; otherwise confirm the
    workflow root is byte-equivalent to its post-scaffold state.

The rehearsal is best-effort and additive: a failure here does **not**
block handoff (the scaffolded distributive is still usable), but the
failure details must be surfaced to the user.

---

## 14. Final handoff

When the verification checklist passes (and the optional rehearsal in
section 13.5 has either completed successfully or been skipped per
interview question 6), tell the user briefly, in the chosen language,
that the distributive is ready and that opening the top-level launcher
will start the `Installation Agent`. If the rehearsal ran, include a
one-line PASS/FAIL summary. Do not push anything to any remote. Stop.

---

## 15. Glossary

- **Workspace** - a directory created from `Workspaces/__template__`
  containing per-workspace artifacts, prompts, and possibly git worktrees.
- **Worker** - a logical concept: the AI session that does the work.
- **Worker wrapper** - a script under `Scripts/Workers/` that knows how to
  invoke a specific AI harness with a bootstrap string.
- **Dispatcher** (`Agent` script) - the single entry point through which all
  launchers and scripts spawn worker wrappers.
- **Bootstrap** - the short text the dispatcher passes to a worker wrapper,
  which the worker wrapper passes to its harness; tells the AI to read a
  prompt file and follow it.
- **Tail** - optional user-provided text appended to the bootstrap with a
  fixed prefix.
- **Mode** - `cli` (unattended) or `tui` (interactive).
- **Chunk** - a block of text in a `Work/*.md` queue file separated by lines
  of `---`.
- **Rollback header** - a single HTML-comment line at the top of a `Done`
  chunk that records per-repository commit hashes for safe rollback.
- **Workspace branch** - the git branch name shared by all repositories of a
  workspace, used as the branch for every worktree.
