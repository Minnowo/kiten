# Project brief: <name>

Status: draft | ready for review | approved (YYYY-MM-DD)
Phases done: none

The single source of truth for what this project is and why. Decisions are
recorded with their reasons so they can be revisited with full context.

## 1. Idea

- **Pitch:** <one paragraph, in the user's terms>
- **User:** <who uses it>
- **Problem:** <what hurts today>
- **v1 is done when:** <observable outcome>
- **Hard constraints:** <language, platform, budget, license, offline, ...>
- **Ambiguous terms:** <word -> agreed meaning>

## 2. Landscape

### Existing software

| Name     | Shape                  | Good at        | Why not (for us)   | License |
|----------|------------------------|----------------|--------------------|---------|
| <tool>   | <where it runs, flow>  | <strengths>    | <gap or mismatch>  | <spdx>  |

### Standards and references

| Standard / spec  | What it covers   | Local copy                 | Version |
|------------------|------------------|----------------------------|---------|
| <name>           | <scope>          | docs/reference/<file>      | <ver>   |

### Architecture patterns

- **<pattern>:** <how it works> - pros: <...> - cons: <...>

### Build vs use

<what exists that already fits, and the reason we are still building, or not>

## 3. Architecture decisions

| Dimension    | Decision      | Options considered     | Reason          | By           |
|--------------|---------------|------------------------|-----------------|--------------|
| Placement    |               |                        |                 | user/default |
| Interfaces   |               |                        |                 |              |
| Data in      |               |                        |                 |              |
| Data out     |               |                        |                 |              |
| State        |               |                        |                 |              |
| Scale        |               |                        |                 |              |
| Platforms    |               |                        |                 |              |
| Security     |               |                        |                 |              |
| Failure      |               |                        |                 |              |
| Config       |               |                        |                 |              |
| Distribution |               |                        |                 |              |
| License      |               |                        |                 |              |

### Non-goals

- <thing v1 will not do>

### Diagram

```
<ascii diagram of components and data flow>
```

## 4. Unknowns surfaced

| Item     | Why it matters    | Decision (v1 / later / never) |
|----------|-------------------|-------------------------------|
| <item>   | <reason>          | <decision>                    |

### Open questions

- <question> - resolve by: <prototype / measurement / ask at milestone N>

## 5. Tech stack

- **Language / runtime:** <choice> - why: <...> - runner-up: <...>
- **Build / test tooling:** <...>

### Dependencies

| Dependency | Purpose  | Why not stdlib / vendored | License | Approved |
|------------|----------|---------------------------|---------|----------|
| <name>     | <...>    | <...>                     | <spdx>  | yes/no   |

### Module layout

```
<dir>/        <purpose>
```

### Test strategy

- <what is unit tested, integration tested, faked, and the fixtures needed>

## 6. Milestones

1. **<name>:** <thin vertical slice> - done when: <test or observable result>
2. ...
