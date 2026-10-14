# START: scope and initialize a new project

You are reading this because the user dropped this bootstrap kit into a new
project folder and asked you to start a project. This file is your workflow.

Your job is to turn a rough idea into a scoped, agreed plan and an initialized
repository. Act as a senior engineer who knows the domain better than the
user does. The user knows what they want; they often do not know what already
exists, what the standard approach is, or what they forgot to ask for. Close
those gaps before any code is written.

## Paths

- **Kit directory:** the directory containing this file (for example
  `<project>/kiten/`). Everything in it is temporary.
- **Project root:** the parent of the kit directory, unless the user says
  otherwise. All project files go here, never inside the kit directory.

Kit files you will use:

- `templates/AGENTS.md` - base rules every project starts from. Read it now;
  these rules apply to you for the whole workflow, starting with ASCII only.
- `templates/brief.md` - the project brief you fill in as you go.
- `examples/server-monitor.md` - a worked example of the depth expected in
  phases 2 to 4. Read it once for calibration; do not copy its content into
  unrelated projects.
- `research/README.md` - index of research notes on how others run agents on
  real projects. Read the index only; open a note when its "Relevant when"
  matches the project (for example the Bun note for a large language port).
  Those notes are more specific to such goals than the generic template.

## Rules for this workflow

- No product code until the user approves the brief (phase 6).
- Keep the brief at `<project root>/docs/brief.md` and update it at the end of
  every phase, so a crashed or resumed session can pick up where it left off.
  If it already exists, read it and resume from the first phase not marked
  done.
- Ask questions in batches of at most 4, each with concrete options and your
  recommendation first. Never ask something you could answer by research or
  a sensible default; state the default instead and let the user override.
- Separate facts from opinions. When you cite how a tool or standard works,
  you must have read its docs in this session, not recalled them.
- Minimal and simple wins ties. Every component, dependency and feature must
  justify itself.

## Phase 1: Restate and frame

1. Look at the project root. If anything besides the kit directory is there,
   list it and ask before touching it.
2. Restate the idea in one paragraph: who uses it, what problem it solves,
   what "working v1" looks like. Call out every ambiguous word (for example
   "minimal", "easy", "monitor", "real-time") and say what you think it means.
3. Ask only the framing questions you need to aim the research (usually 1 to
   3): who is the user (just me / my team / the public), scale (one machine /
   ten / thousands), and any hard constraints (language, platform, budget,
   license, must run offline, etc.).
4. Create `docs/brief.md` from `templates/brief.md` and fill in section 1.

## Phase 2: Research the landscape

Do this before the detailed questions, so the questions are informed.

1. **Existing software.** Find 5 to 10 projects that already solve this or
   part of it: open source and commercial, heavyweight and minimal. For each:
   what it is, its architecture (where it runs, how data flows), what it does
   well, why it might not fit, license. Read their docs, not just their
   landing pages.
2. **Standards and conventions.** What formats, protocols, specs and
   de facto conventions does this domain use? (Wire formats, file formats,
   APIs, exit codes, config conventions, OS interfaces.) Download the
   authoritative references you will rely on into `docs/reference/` with a
   note of URL and version.
3. **Architecture patterns.** What are the 2 to 4 common shapes for this kind
   of system (for example push vs pull, agent vs agentless, embedded vs
   client/server)? What are the trade-offs of each?
4. **Build vs use.** Say plainly if an existing tool already does what the
   user described. Options: use it as is, use it and add a thin layer,
   build something different because of a stated gap, or build anyway for
   learning or control. The user decides; your job is to make the choice
   informed.
5. **Kit research.** Check `research/README.md` for notes matching this kind
   of project and carry over any lessons that apply, citing the note.
6. Write the findings to brief section 2, then give the user a short summary:
   the top 3 to 5 relevant tools, the dominant standard(s), and anything that
   changes the shape of the original idea.

## Phase 3: Clarify the architecture

Walk these dimensions. Skip ones that do not apply; default the ones that are
obvious; ask about the rest, using what you learned in phase 2 to offer real
options (for example "pull, like node_exporter" vs "push, like Telegraf").

| Dimension      | Questions                                                              |
|----------------|------------------------------------------------------------------------|
| Placement      | Where does each part run? On the target, on a separate host, both?     |
| Interfaces     | CLI, TUI, web UI, HTTP API, library? Who or what calls it?             |
| Data in        | What sources, how often, how are they read (files, syscalls, APIs)?    |
| Data out       | Who reads results and how? Formats? Compatibility with standards?      |
| State          | Stateless, local files, embedded DB, external DB? Retention?           |
| Scale          | How many users / hosts / requests / items, now and in a year?          |
| Platforms      | OSes, architectures, containers, minimum versions?                     |
| Security       | Network exposure, auth, secrets, privileges required, threat model?    |
| Failure        | What happens when a part is down, slow, or full? What must not break?  |
| Config         | Flags, file, env? Zero-config defaults?                                |
| Distribution   | How is it installed and updated? Single binary, package, container?    |
| License        | Project license? Constraints on dependencies?                          |
| Non-goals      | What will v1 explicitly not do?                                        |

Record each decision in brief section 3 as: decision, options considered,
reason. Mark user decisions vs your defaults.

## Phase 4: Surface the unknowns

List what the user did not mention but a domain expert would expect to be
decided. Draw from the research: features every comparable tool has, common
failure modes, security and privacy issues, operational concerns, legal or
licensing traps, performance cliffs, standards they should interoperate with.

Present them as a table: item, why it matters, recommendation (v1 / later /
never). Let the user adjust. Record the result in brief section 4. Also list
any open questions that cannot be answered yet and how they will be resolved
(prototype, measurement, ask later).

## Phase 5: Tech stack and dependencies

1. Propose a language and runtime with reasons tied to the decisions above
   (deployment shape, performance, platform reach, ecosystem for the needed
   standards). Favour languages and tools whose compiler or type system
   catches the project's likely bug classes: a compiler error is a better
   feedback loop than a style rule. Mention the runner-up and why it lost.
2. For each proposed dependency: what it does, why the standard library or a
   vendored function is not enough, license, maintenance status, transitive
   weight. Default to no dependency.
3. Propose the module layout following the segmentation rule: one directory
   per feature, lower layers never import higher ones, removing a feature
   means deleting its directory and its registration line.
4. Propose the test strategy: tests at the external interface first (CLI,
   API, files, protocol), what needs integration tests, what fixtures are
   needed, how external systems are faked. Add fuzzing for any parser of
   untrusted input.
5. Propose enforcement: which rules from `templates/AGENTS.md` and the brief
   can be checked by tools (formatter, linter config, ASCII check, layering
   check, license check). These go into a single check script in phase 7.
6. The user approves the stack and every dependency explicitly. Record in
   brief section 5.

## Phase 6: Plan, review and approve

1. Break v1 into milestones of thin vertical slices, each one runnable and
   tested end to end. The first milestone is the smallest thing that proves
   the architecture (one data source, one output, wired through).
2. Fill brief section 6.
3. **Adversarial review of the brief.** Spawn a reviewer subagent that gets
   only `docs/brief.md` and the user's original idea, and is told to find
   what is wrong: missing requirements, contradictions, overbuilt parts,
   risky assumptions, decisions without reasons. Address each finding in the
   brief or note why it does not apply.
4. Mark the brief "ready for review", show the user a summary including what
   the reviewer changed, and ask for approval. Iterate until approved, then
   mark it "approved" with the date.

## Phase 7: Initialize the repository

Only after approval. All files go in the project root.

1. Write `AGENTS.md`: copy `templates/AGENTS.md` (if Claude also copy `templates/CLAUDE.md`),
   then fill in "Project specifics" from the brief (overview, layout, 
   commands, language notes). Keep it short; link to `docs/brief.md` for detail.
   Remove template comments that were filled in.
2. Add `LICENSE` for the chosen license, a `.gitignore` for the stack, and a
   short `README.md` (what it is, how to build, how to run).
3. Create the module skeleton from phase 5. Add a minimal entry point and one
   passing test at the external interface.
4. Add one check script (for example `scripts/check.sh` or a make target)
   that runs build, tests, linter, formatter check, the ASCII check and any
   other enforcement from phase 5. It must pass.
5. Delete the kit directory (the directory containing this file), including
   its `.git`. Confirm nothing in the project references it.
6. `git init` (default branch `main`) if the project root is not already a
   repository. Stage the specific files created and make one local commit:
   `Initial project scaffold`. Never push.
7. Tell the user what was created, the exact check/build/run commands, and
   the first milestone to start on.
