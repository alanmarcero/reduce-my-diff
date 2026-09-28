---
name: reduce-my-diff
description: Keep a change's diff as small as the task allows. Use before implementing, while implementing, and as a sweep of the branch diff against main before opening or updating a PR.
---

# Reduce my diff

Every changed line should trace directly to the request. The rules are ordered by how much diff they save: the first ones prevent whole features, the last ones trim lines.

These rules favor caution over speed. For a trivial change, use judgment.

Sources: [Andrej Karpathy's coding guidelines](https://github.com/multica-ai/andrej-karpathy-skills), Anthropic's [code-simplifier](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier), Cursor's [deslop](https://github.com/cursor/plugins/blob/main/cursor-team-kit/skills/deslop/SKILL.md), and [Alan Marcero](https://github.com/alanmarcero)'s own rules from PR reviews.

## 1. Simplify the requirement before writing code

Not every instruction is a separate, binding requirement. Given requirements A, A^, B and B^, a human weighs the tradeoffs and builds one shared feature. An LLM builds four features, each with its own tests, then says it can't simplify further.

- Look for a simpler requirement that covers the same need. If it makes the implementation substantially smaller, propose it and get approval before building. Don't bring up small simplifications.
- If a simpler approach exists, say so. Push back when it's warranted.
- State your assumptions. If the request has more than one reading, present them instead of picking one silently. If something is unclear, stop, name what's confusing, and ask.
- Define done as a check you can run: a failing test that reproduces the bug, or tests that pass before and after a refactor. Stop when the check passes. For multi-step work, write a short plan with a check per step.

## 2. Build only what was asked

- No features, flexibility or configuration that wasn't requested.
- No error handling for cases that can't happen. Avoid try/catch unless the code path needs it.
- No abstraction for code used once. Don't add a one- or two-line function unless it will be used 4+ times.
- If you wrote 200 lines and it could be 50, rewrite it. Would a senior engineer call it overcomplicated?

## 3. Change existing code before adding new code

- Modify what the repo already has instead of adding a parallel path next to it.
- Reuse the repo's conventions. For example, use one metric key with an outcome tag instead of three keys when the neighbouring code does it that way.
- Don't add code only to support a frozen legacy path. Tag the new path and let existing filters exclude the untagged rows.

## 4. Touch only what you must

- Don't restructure code to add a signal. Wrapping a function body in `try` to catch one more case re-indents every line and turns a 10-line change into 90. Take the smaller signal.
- Don't improve nearby code, comments or formatting, and don't refactor things that aren't broken.
- Match the existing style and naming, even if you'd write it differently. Apply the project's CLAUDE.md standards to lines you're already changing, not to the rest of the file.
- Remove imports, variables and functions that your change made unused. Mention unrelated dead code you find, but don't delete it.

## 5. Leave whitespace alone

Don't change whitespace on any line you aren't otherwise changing, unless a lint or format rule the pipeline enforces requires it.

- Don't re-indent, reflow or rewrap existing lines. Don't add or remove blank lines, trailing whitespace, final newlines or line endings, and don't switch tabs and spaces.
- Watch for editor format-on-save. It reformats whole files; format only the lines you changed.
- Keep whitespace in your own additions to a minimum. Follow the file's existing spacing and add no blank lines beyond what the code around it uses. Prefer shapes that don't shift existing lines, like a guard clause instead of wrapping a block in a new `if` or `try`.
- Stay readable and keep the pipeline passing. Minimal whitespace never means cramming code onto one line or breaking the linter.
- When lint requires reformatting, put it in its own commit so reviewers can skip it. If CI lints entire modified files, run the formatter check (for example `npx prettier --check`) on every file you touch and fix existing violations in that separate commit.
- Check before pushing. Compare `git diff --stat` with `git diff -w --stat`: any difference is whitespace-only churn, so revert what lint doesn't require. `git diff --check` flags trailing whitespace you introduced.

## 6. Keep test changes small

- Add assertions to existing tests that already drive the path. Write a new test only for a case no existing test sets up.
- Delete a test that your change made redundant. If a new assertion in an existing test covers the fallback, the separate fallback test is dead.
- Never test a mock of the thing you changed. A suite that mocks the module under test only proves the mock works; find where the real logic is covered.

## 7. Sweep the branch diff against main

Run the sweep after each change, not only at the end. Remove what the branch added that the codebase wouldn't:

- Every comment the change adds, unless the user asks for one. The code and the test names carry the meaning.
- Defensive checks or try/catch blocks that are unusual for that code path.
- Casts to `any` that only exist to get past a type error.
- Deep nesting in code the branch added. Flatten it with early returns. Leave existing code alone (see 4).
- Duplicate or redundant logic the branch added. Consolidate it, and give new code clear names.
- Anything else that doesn't match the file and the code around it.

Stop the sweep when the remaining edits would be broad rewrites. A focused edit that removes lines beats a rewrite that moves them.

## Guardrails

- Keep behavior unchanged unless fixing a clear bug. Tests pass before and after the sweep.
- Choose clarity over brevity. Don't shrink the diff with clever or dense code: no nested ternaries, dense one-liners, or functions that combine unrelated concerns. Keep an abstraction that already helps organize the code.
- Keep the final summary to 1-3 sentences, and mention only changes that affect how someone reads the code.
- The rules are working when diffs have fewer unrelated changes, fewer rewrites come from overcomplication, and clarifying questions come before implementation instead of after mistakes.
