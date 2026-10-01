# Reduce My Diff

A skill for AI coding agents that keeps pull request diffs as small as the task allows. Coding agents build more than was asked, reformat code they don't need to touch, and add comments, defensive checks and extra tests. The rules apply before, during and after implementing, in order of the diff they save:

0. Before you start: state assumptions, take a sensible default, and define done as a check you can run.
1. Feature Consolidation: merge overlapping requirements into one feature, flag a new feature where an existing one could expand, and propose removing requirements whose code costs more than their risk. It always runs; nothing changes until you approve.
2. Build only what was asked
3. Change existing code before adding new code
4. Touch only what you must
5. Leave whitespace alone
6. Keep test changes small
7. Sweep the branch diff against main

Guardrails keep behavior unchanged, protect tuning values the real environment needs, and put readable, boring code ahead of line count. Correctness, security and performance findings are out of scope; send them to a normal code review.

## Install

### Claude Code

```bash
mkdir -p ~/.claude/skills/reduce-my-diff
cp SKILL.md ~/.claude/skills/reduce-my-diff/SKILL.md
```

### Other AI agents

`SKILL.md` is plain markdown. Point your agent at it or paste it into a system prompt.

## Usage

```
/reduce-my-diff
```

Run it before implementing to scope the work, or on a finished branch to sweep its diff against main.

Pass PR URLs for a report per PR, scoped to the lines they change, or name a repo to review all its code. Each finding is one line, in the Ponytail review format: the lines it removes, its `path:line`, a tag (`delete`, `reuse`, `stdlib`, `native`, `yagni`, `shrink` or `churn`), what to cut and a concrete replacement. Findings are grouped as **no impact**, **possible impact** (the agent can't verify the effect) or **confirmed impact** (the agent read the code and the cut changes a feature), and sorted by lines removed. Each section ends with the totals for each group and a `net` line. When there is nothing to cut, the report says `Lean already. Ship.` By default the skill only reports; it doesn't edit, push or comment unless you ask.

A finding with possible or confirmed impact always needs your approval. "Fix everything" skips those findings and lists them for you. If you reject a proposal, the skill builds what you asked for and doesn't propose it again.

## Sources

The skill combines four published sources with my own rules from PR reviews:

- [Andrej Karpathy's coding guidelines](https://github.com/multica-ai/andrej-karpathy-skills): think before coding, simplicity first, surgical changes, goal-driven execution, the senior-engineer check, and the style-drift examples (docstrings, type hints and quote changes nobody asked for).
- Anthropic's [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier) plugin: preserve behavior, limit changes to modified code, keep clarity ahead of brevity, avoid clever code and changes that are harder to debug, and verify the result is simpler.
- Cursor's [deslop](https://github.com/cursor/plugins/blob/main/cursor-team-kit/skills/deslop/SKILL.md) skill: remove comments that don't match the file, defensive checks that are unusual for a trusted code path, and `any` casts that the branch added.
- [Ponytail](https://github.com/dietrichgebert/ponytail): check the codebase, the standard library, the platform and installed dependencies before writing code; search every caller and fix a bug once in the shared function; read the flow first; take a default instead of stalling; deletion over addition, boring over clever and fewest files; no scaffolding for later; one runnable check per branch and no test bloat; never cut trust-boundary validation, data-loss handling, security, accessibility or calibration; mark a deliberate shortcut with its limit and upgrade trigger; the review report format, tags, examples, `net` total and `Lean already. Ship.`; the repo audit hunt list and deps total; the `skipped: X, add when Y` line; and never report savings from code that was never written.
- My own rules: merge overlapping requirements, leave whitespace alone, don't restructure code to add a signal, add assertions to existing tests, change data before code, check parity fields against the source system, and never extract a one- or two-line function.
