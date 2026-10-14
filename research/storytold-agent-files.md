# storytold craft apps: agent instruction files

- **Sources:** AGENTS.md / CLAUDE.md files from four public repositories,
  fetched 2026-10-14:
  - https://github.com/storytold/artcraft - `AGENTS.md`, `crates/AGENTS.md`,
    and per-module files under `crates/schema/public/{enums,tokens}/`,
    `crates/api_clients/artcraft/{artcraft_api_defs,artcraft_client}/`,
    `crates/api_clients/artcraft/artcraft_router/src/generate/generate_video/`,
    `crates/desktop/artcraft/src/services/storyteller/commands/`,
    `frontend/libs/components/pagescene/`
  - https://github.com/storytold/photocraft - `AGENTS.md`
  - https://github.com/storytold/lightcraft - `AGENTS.md`, `CLAUDE.md`
  - https://github.com/storytold/filmcraft - `AGENTS.md`, `CLAUDE.md`,
    `docs/agents.md`
- **What it is:** A family of agent-built Rust apps: an AI image/video
  desktop app (Tauri + TypeScript) and clean-room clones of Photoshop,
  Lightroom and Premiere (native egui, wasm builds), developed largely by
  autonomous and parallel agents.
- **Relevant when:** writing agent instruction files in general; large Rust
  workspaces; desktop or creative apps; clean-room clones of an existing
  product ("parity" with a target app); apps that agents must be able to drive
  (CLI / control channel / MCP); many agents working on one repo for long
  unattended runs.
- **Used in this kit:** most of `templates/CLAUDE.md` beyond the author's own
  rules (see table below).

## Repositories at a glance

| Repo       | Files                                                   | Product                                  |
|------------|---------------------------------------------------------|------------------------------------------|
| artcraft   | root AGENTS.md + 8 nested per-module AGENTS.md          | Tauri desktop app for AI image/video     |
| photocraft | root AGENTS.md (~18 KB)                                 | Photoshop-parity image editor            |
| lightcraft | root AGENTS.md and CLAUDE.md (near-identical, drifting) | Lightroom-parity photo library/developer |
| filmcraft  | AGENTS.md (rules), CLAUDE.md (workflow), docs/agents.md | Premiere-parity video editor             |

## Structure patterns

- **CLAUDE.md as a pointer.** In artcraft every CLAUDE.md is a 9-byte file
  containing `AGENTS.md` (a symlink). One source of truth, both tools read it.
  lightcraft keeps two copies and they have drifted apart; avoid that.
- **Nested per-module files.** artcraft puts a short AGENTS.md in each crate or
  module with rules local to it: how to add a new enum, a new token type, a new
  provider; "never change stored enum string values"; dependency allowlists for
  a crate. Keeps the root file small.
- **Rules vs workflow split.** filmcraft: AGENTS.md holds rules "that must never
  be broken" and wins on conflict; CLAUDE.md holds working instructions.
- **Shared standards repo.** The craft apps link to a common `craftrules` repo
  for cross-project standards (never-crash, fonts, progress docs). Same idea as
  this kit.
- **Orientation table.** photocraft opens with a "read this, because" table of
  docs. Cheap for agents to follow.
- **Start-of-session protocol.** lightcraft/filmcraft: read the status file,
  the next task, the relevant architecture section and crate README, then
  follow an autonomous operation protocol that lists which decisions are the
  user's.

## Rules borrowed into the kit

| Rule                                                              | Source             | Template section      |
|-------------------------------------------------------------------|--------------------|-----------------------|
| Never crash: no unwrap/expect/panic outside tests, return errors  | photo/light/film   | Keep it simple        |
| Input-derived numbers are hostile (bounds, div by zero, caps)     | photo/light/film   | Keep it simple        |
| Every crash fix gets a regression test that failed before         | photo/light/film   | Fix bugs test-first   |
| Never weaken a check to make a commit pass                        | lightcraft         | Fix bugs test-first   |
| Implement from public specs, record the spec edition used         | filmcraft          | Read the spec         |
| Permissive licenses only, no GPL code copied                      | filmcraft          | Minimal dependencies  |
| One endpoint / command per file, no catch-all files               | artcraft           | Segmentation          |
| Layering enforced; lower layers never depend on higher            | photo/light/film   | Segmentation          |
| File layout top to bottom, callers above callees                  | artcraft           | File layout           |
| Test layout: constants, cases, helpers last                       | artcraft           | File layout           |
| Logging macros, never print outside tests                         | artcraft           | Working style         |
| Space-padded markdown tables                                      | artcraft           | Working style         |
| Look at the UI result (screenshot) after UI changes               | photo/light/film   | Working style         |
| Report honestly: measured vs estimated, stubs are not done        | filmcraft          | Working style         |
| Commit small and often, only green states                         | lightcraft         | Working style         |
| Ask a human only about: licenses, money, publishing, personal     | filmcraft          | Ask a human first     |
| data, destructive git                                             |                    |                       |
| Preserve untracked/ignored files; delete only tracked paths       | artcraft           | Ask a human first     |
| No personal details in request headers / user agents              | photocraft         | Ask a human first     |
| Parallel agents: own worktree, own target dir, own port           | photo/light/film   | Parallel agents       |
| Do not fix another agent's in-progress files                      | photocraft         | Parallel agents       |

## Project-specific patterns (not in the template, reuse when relevant)

- **Everything is a command.** Every user-visible action is a command in an
  engine registry (id, label, menu path, shortcut, params doc, enabled, run).
  UI, CLI, control channel and MCP server all dispatch by id. Makes the app
  fully drivable and testable by agents. Strong pattern for any app agents
  should operate.
- **Agent-drivable UI.** UI state is serializable; every widget has an
  automation id; a localhost JSON-lines control channel can click, type and
  take screenshots; headless snapshot rendering for unattended runs. Never
  send OS-level synthetic input (it lands in whatever window has focus).
- **MCP conventions** (filmcraft `docs/agents.md`): a small core tool set
  shared across sibling apps, MCP hints (readOnly/destructive/idempotent) on
  every tool, unknown arguments rejected with the list of accepted ones,
  progress notifications and cancellation for long jobs.
- **Parity tracking** for clones of an existing product: a ranked gaps list,
  per-dimension parity checklists, a generated scorecard of measured numbers,
  a parity floor that may only go up. Two numbers: "feature breadth" and
  "ready for real work".
- **Never-crash engineering** (Rust): `#![deny(clippy::unwrap_used, ...)]` on
  production crates, `clippy.toml` allowing them in tests only, poisoned
  locks recovered, workers under `catch_unwind`, a panic-hunt test that fuzzes
  every command with adversarial params.
- **Clean-room rules:** study the target only by behaviour and public docs;
  never read its binaries or copy assets; implement formats from specs; no
  GPL code; external tools (ffmpeg) only as test oracles, never linked.
- **Asset licensing:** every asset gets an attribution sidecar and an index
  row; a CI check fails on unattributed assets or suspicious file names.
- **Dev log** appended after each task (what landed, numbers, what is still
  open) so a crashed session can be resumed by the next agent.
- **Real-file test corpora** fetched at pinned commits, sha256-verified,
  never committed.
- **Parallel agents on Cargo:** keep every `Cargo.toml` valid at all times
  (one broken manifest breaks every agent's build); write manifests to a temp
  file and `mv` them into place.

## Anti-patterns observed

- Heavy use of em dashes, arrows and other unicode in the files themselves.
- Very long root files (photocraft ~18 KB, pagescene ~45 KB). The 45 KB
  pagescene file is mostly feature-in-progress notes; that belongs in a plan or
  devlog, not the instruction file.
- Duplicate CLAUDE.md / AGENTS.md copies that drift.
