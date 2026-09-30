# Reduce My Diff

A skill for AI coding agents that keeps pull request diffs as small as the task allows. Coding agents build more than was asked, reformat code they don't need to touch, and add comments, defensive checks and extra tests. The rules apply before, during and after implementing, in order of the diff they save:

1. Feature Consolidation: merge overlapping requirements into one feature, flag a new feature where an existing one could expand, and propose removing requirements whose code costs more than their risk. It always runs; nothing changes until you approve.
2. Build only what was asked
3. Change existing code before adding new code
4. Touch only what you must
5. Leave whitespace alone
6. Keep test changes small
7. Sweep the branch diff against main

Guardrails keep behavior unchanged and put readability ahead of line count.

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

Pass PR URLs for a report per PR, scoped to the lines they change, or name a repo to review all its code. Each item gives the lines to remove and their `path:line`, sorted by lines removed. By default the skill only reports; it doesn't edit, push or comment unless you ask.

A change to a requirement or a feature always needs your approval. "Fix everything" skips those changes and lists them for you.

## Sources

The skill combines four published sources with my own rules from PR reviews:

- [Andrej Karpathy's coding guidelines](https://github.com/multica-ai/andrej-karpathy-skills): think before coding, simplicity first, surgical changes, goal-driven execution.
- Anthropic's [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier) plugin: preserve behavior, limit changes to modified code, keep clarity ahead of brevity.
- Cursor's [deslop](https://github.com/cursor/plugins/blob/main/cursor-team-kit/skills/deslop/SKILL.md) skill: remove comments, defensive checks and `any` casts that the branch added.
- [Ponytail](https://github.com/dietrichgebert/ponytail): check the codebase, the standard library, the platform and installed dependencies before writing code; fix a bug once in the shared function; tag findings in a short report; read the flow first; never cut trust-boundary validation, data-loss handling, security or accessibility; mark a deliberate shortcut with its limit.
- My own rules: merge overlapping requirements, leave whitespace alone, don't restructure code to add a signal, add assertions to existing tests, change data before code, check parity fields against the source system, and never extract a one- or two-line function.
