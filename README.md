# kiten

A starting point for AI-assisted projects. Drop it into an empty
project folder, point the agent at it, and it scopes the project with you, writes
the AGENTS.md, initializes the repo, then deletes itself.

| Path                         | What it is                                                     |
|------------------------------|----------------------------------------------------------------|
| `START.md`                   | The workflow the agent follows (research, questions, init)     |
| `templates/AGENTS.md`        | Opinionated base rules for the agent                           |
| `templates/CLAUDE.md`        | A CLAUDE.md that points to the AGENTS.md                       |
| `templates/brief.md`         | Project brief the agent fills in during scoping                |
| `examples/server-monitor.md` | Worked example of the expected research and question depth     |
| `research/`                  | Sourced notes on how others run agents; read when relevant     |

## Usage

```sh
mkdir myproject && cd myproject
git clone <this repo> kiten
claude
```

Then tell your agent:

```
Read kiten/START.md. I'm starting a project: it's a xyz...
```

The kit lives only in `kiten/` during scoping. Project files (brief,
references, AGENTS.md, code) are written to the project root, and `kiten/`
is deleted before the first commit.
