# Research

Notes on how others run AI agents on real projects, with sources. The general
lessons are already folded into `../templates/CLAUDE.md` and `../START.md`.

**Agents using this kit:** you do not need to read every file here. Scan the
index below. If the project you are scoping matches a "Relevant when" entry,
read that file: it holds lessons specific to that kind of project that the
generic template leaves out. Cite what you use in the brief (section 2).

## Index

| File                       | Relevant when                                                              |
|----------------------------|----------------------------------------------------------------------------|
| `storytold-agent-files.md` | Writing agent instruction files; large Rust workspaces; desktop/creative   |
|                            | apps; clean-room clones of an existing product; apps agents must drive     |
|                            | (CLI, control channel, MCP); many agents on one repo for long runs         |
| `bun-zig-to-rust.md`       | Rewriting or porting a large codebase to another language; very large      |
|                            | mechanical changes; many parallel agents for days; reviewing huge volumes  |
|                            | of agent-written code; choosing a language for memory safety               |

## Adding research

One file per source or closely related set of sources, named
`<subject>-<topic>.md`. Start it with this header, then add a row to the index:

```
# <Title>

- **Sources:** <URLs, authors, publication date, date fetched>
- **What it is:** <one paragraph>
- **Relevant when:** <kinds of projects or tasks this helps with>
- **Used in this kit:** <what was folded into the template, or "nothing yet">
```

Then the findings. Keep facts traceable to a source, mark your own opinions
as such, and write in ASCII.
