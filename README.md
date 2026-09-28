# Reduce My Diff

A skill for AI coding agents that keeps pull request diffs as small as the task allows. Coding agents tend to build more than was asked, reformat code they didn't need to touch, and leave behind comments, defensive checks and extra tests. This skill gives the agent rules to follow before, during and after implementing, ordered by how much diff each one saves:

1. Simplify the requirement before writing code
2. Build only what was asked
3. Change existing code before adding new code
4. Touch only what you must
5. Leave whitespace alone
6. Keep test changes small
7. Sweep the branch diff against main

It ends with guardrails that keep behavior unchanged and keep readability ahead of line count.

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

Pass one or more PR URLs to get a report for each PR. The report lists the lines the skill would remove, sorted from most lines removed to fewest. By default the skill only reports; it doesn't edit, push or comment unless you ask.

## Sources

This skill consolidates three published sources with my own rules from PR reviews:

- [Andrej Karpathy's coding guidelines](https://github.com/multica-ai/andrej-karpathy-skills): think before coding, simplicity first, surgical changes, goal-driven execution.
- Anthropic's [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier) plugin: preserve behavior, limit changes to modified code, keep clarity ahead of brevity.
- Cursor's [deslop](https://github.com/cursor/plugins/blob/main/cursor-team-kit/skills/deslop/SKILL.md) skill: remove comments, defensive checks and `any` casts that the branch added.
- My own rules: simplify overlapping requirements into one feature, leave whitespace alone, avoid restructuring code to add one signal, reuse repo conventions, and fold new assertions into existing tests.
