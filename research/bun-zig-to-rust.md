# Bun: rewriting a 535k-line Zig codebase in Rust with agents

- **Sources:**
  - https://bun.com/blog/bun-in-rust - "Rewriting Bun in Rust", Jarred
    Sumner, 2026-07-08. Fetched 2026-10-14.
  - Mentioned there but not read for this note: Bun's PORTING.md (the
    Zig-to-Rust pattern guide), TigerBeetle's TigerStyle, Google's C++ style
    guide.
- **What it is:** One engineer used about 50 Claude Code dynamic workflows,
  with up to 64 agents at once, to port Bun (JS runtime, bundler, package
  manager) from Zig to Rust in 11 days: +1,009,272 lines, 6,502 commits, the
  full test suite passing on 6 platforms, 0 tests skipped or deleted.
- **Relevant when:** rewriting or porting a large codebase between languages;
  any very large mechanical change (framework migration, API overhaul, crate
  split); running many agents in parallel for days; choosing a language for
  memory safety; deciding how to review huge volumes of agent-written code.
- **Used in this kit:** the "Independent review" section and several working
  style rules in `templates/CLAUDE.md`; review of the brief and the
  enforcement script in `START.md`.

## Why rewrite at all

- Recurring bug classes (use-after-free, double-free, leaks on error paths)
  in code mixing GC-managed and manually managed memory.
- They already had ASAN on every commit, safety-checked builds, 24/7 fuzzing
  and leak tests. Bugs kept coming.
- Style guides need enforcement by review. "Compiler errors are a better
  feedback loop than a style guide." Rust's borrow checker and Drop turn the
  bug classes into compile errors or automatic cleanup.
- A manual rewrite was estimated at 3 engineers for a year with feature work
  frozen, so it was never an option before agents.

## The two big decisions

1. **All at once, not incremental.** Incremental rewrites add temporary glue
   that you hope gets deleted.
2. **Mechanical port first.** Make the Rust look like transpiled Zig, keep the
   same architecture, performance and features, so anyone who knows the old
   code knows the new code. Refactor toward idiomatic Rust after shipping.

"Those are the only two big questions. Everything else is tactics."

## The workflow, in phases

Each phase was a loop: 1 implementer -> 2 adversarial reviewers -> 1 fixer.

1. **Porting guide.** ~3 hours of conversation on mapping Zig patterns and
   types to Rust, serialized into PORTING.md.
2. **Lifetime analysis.** A workflow read every struct field, traced control
   flow, proposed a Rust lifetime per field, had 2 reviewers check each, and
   wrote LIFETIMES.tsv. Then PORTING.md and LIFETIMES.tsv were reviewed
   together for conflicts, and read by the human.
3. **Trial run.** 3 files through the full loop before all 1,448.
4. **Mechanical port** of every .zig file to .rs, following the guide and the
   lifetimes table. Peak ~1,300 lines per minute. None of it compiled yet.
5. **Compiler errors as a work queue.** Fix cyclic dependencies first (a
   workflow to classify, then one to refactor). Then per crate: run
   `cargo check` once, save errors to a file grouped by file, fix, review,
   apply. ~16,000 errors over 64 agents.
6. **Smoke tests.** Get `bun --version`, then each subcommand working: save
   each failing stack trace per subcommand, fix, review, apply.
7. **Local test suite.** ~100 random test files per round, sharded by folder
   to 4 worktrees; each failure saved with its trace and fixed in the loop.
8. **CI per platform** until all 6 were green (Linux first, Windows last).
   Failing test files went from 972 to 23 in two days.
9. **Cleanup workflows:** dedupe code, reduce `unsafe`, Windows fixes.
10. **Merge** only after 100% of tests passed on all platforms and the human
    checked the tests were really running and not skipped.

## Adversarial review

- The implementer wants the code accepted; a reviewer wants to find issues.
  "The implementer doesn't review. The reviewer doesn't implement."
- The reviewer gets only the diff, in its own context, and is told to assume
  the code is wrong.
- Bugs it caught that compiled cleanly: freeing a Box before an asynchronous
  libuv close callback (use-after-free then double-free); `trunc` instead of
  `floor` producing a negative nanosecond field for pre-1970 times; eager
  `unwrap_or` argument panicking where a lazy `unwrap_or_else` was needed.
- The human reviewed the reviewers: checked they caught discrepancies and
  enforced the guides, and read code side by side.

## Failures and the rules they produced

| Failure                                                     | Rule                                                       |
|-------------------------------------------------------------|------------------------------------------------------------|
| Agents ran git stash / stash pop / reset --hard on each     | Never stash/reset; commit only specific files              |
| other                                                       |                                                            |
| One worktree per agent ran out of disk (huge repo)          | 4 worktrees, 16 agents each; no slow commands in the loop  |
| "Get it to compile" became "stub out failing functions"     | No stubs or placeholders to get green                      |
| Long comments justifying workarounds                        | Paragraph-long justification means the code is wrong       |
| Stress tests exhausted sockets, disk, memory                | systemd-run / cgroups limits for memory, CPU, pids         |
| One slow grep froze disk IO on an under-provisioned VM      | Provision IOPS; avoid heavy commands from many agents      |

## Regressions: same syntax, different semantics

19 known regressions after merge, all fixed. Most came from code that looks
identical in both languages but behaves differently. Worth a checklist in any
port:

- **Asserts with side effects.** Zig `assert(f())` always runs `f`; Rust
  `debug_assert!(f())` is erased in release, so `f` stopped running.
- **Edge cases of helpers.** A Zig helper silently ignored a trailing odd
  byte; the Rust replacement (`bytemuck::cast_slice`) panicked on it.
- **Placeholder values.** A constant left as a stand-in "until Phase B"
  lowered a capacity limit 30x; real projects hit it.
- **Build-mode safety.** Zig ReleaseFast had no bounds checks; Rust release
  keeps them, so a latent off-by-one became a panic.
- **Compile-time vs runtime.** Zig `comptime` format strings were processed
  before argument substitution; the Rust function saw the finished string and
  rewrote markers inside arguments. Needed a macro.

## Results and cost

- 128 bugs present in the old version fixed; leaks in repeated builds gone
  (6.7 GB -> 609 MB after 2,000 builds); binary ~20% smaller with other work;
  2-5% faster (cross-language LTO); less stack use in recursive parsers.
- ~4% of the Rust is inside `unsafe`, mostly one-line FFI calls.
- Cost before merge: 5.9B uncached input tokens, 690M output tokens, 72B
  cached reads, about $165,000 at API pricing.
- After merge: 11 rounds of automated security review; 24/7 coverage-guided
  fuzzing of every parser, with findings turned into reproducing PRs by an
  agent and reviewed by humans.

## Takeaways for a rewrite project

- Have a test suite that does not depend on the implementation language (or
  on the internals) before starting. It is the oracle.
- Write the mapping guide and the hard-decisions table (ownership,
  lifetimes, error handling) first, review them adversarially, then port.
- Trial on a few files, then scale. Shard by directory; keep the number of
  worktrees small for big repos.
- Turn every failure list (compiler errors, crashes, failing tests) into a
  file and let agents work it.
- When agents go wrong, edit the loop instructions, not the output.
- Keep a checklist of semantic differences between the two languages and
  have reviewers check for them.
