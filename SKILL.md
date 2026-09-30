---
name: reduce-my-diff
description: Keep a change's diff as small as the task allows. Use before implementing, while implementing, as a sweep of the branch diff against main before opening or updating a PR, or as a sweep of one or more PRs or an entire repo.
---

# Reduce my diff

Every changed line must trace directly to the request. The first rules prevent whole features; the last rules trim lines. For a trivial change, use judgment.

## Scope

- **PRs**: review only the lines the PRs change, and report or change nothing outside their diffs. Feature Consolidation (rule 1) is the exception: it also reads the rest of the repo. Compare a stacked PR against its base branch, not main.
- **Entire repo**: review all the code. Apply rules 2 to 7 to the code and rule 1 to the features. Also look for a dependency that the standard library or the platform replaces, an interface with one implementation, a factory with one product, a wrapper that only delegates, and a flag or config value that nothing sets.
- **No target**: review the current branch diff against main.

## 1. Feature Consolidation

Given requirements A, A^, B and B^, a human weighs the tradeoffs and builds one shared feature. An LLM builds four features, each with its own tests. Also, some requirements need more code than their risk is worth.

Run this rule on every scope. Nothing changes until the user approves. For PRs, read the repo for existing features that do the same job as the new code.

- List the requirements: one per feature, code path, guard, field or endpoint, including ones nobody asked for.
- Group the requirements that overlap, in the PRs and across the PRs and the repo. For each group, propose one shared feature and one set of tests.
- If a PR adds a feature where an existing feature could expand, name the existing feature, its file, and what the expansion needs.
- If a requirement's code is large and its risk is small or unlikely, propose to remove the requirement.
- For each proposal, give the lines it removes, what the user loses, and the risk it leaves.
- Propose only changes that make the implementation substantially smaller. If a simpler approach exists, say so and push back.

- State your assumptions. If the request has more than one reading, present the readings instead of picking one silently. If something is unclear, stop, name the problem, and ask.
- Define done as a check you can run: a failing test that reproduces the bug, or tests that pass before and after a refactor. Stop when the check passes. For multi-step work, write a short plan with a check per step.

## 2. Build only what was asked

- No features, flexibility or configuration that nobody requested.
- No error handling for cases that can't happen. Avoid try/catch unless the code path needs it.
- No abstraction for code used once. Don't add a one- or two-line function, however often it repeats. Extract a repeated value as a named constant. Extract a function only for 3 or more lines of real logic.
- If you wrote 200 lines and 50 would do, rewrite it.

## 3. Change existing code before adding new code

- Read the code the change touches and trace the real flow before you choose an approach. The smallest change in the wrong place is a second bug.
- Before you write new code, ask these questions in order. Stop at the first yes:
  1. Does this need to exist? If not, don't build it (see 2).
  2. Does the codebase already have it? Reuse that helper, util or pattern.
  3. Can data change instead of code? For example a definition, a template or a config value. A data change adds no schema, API or engine branch.
  4. Does the standard library do it? Use it.
  5. Does the platform do it? For example `<input type="date">`, CSS, or a database constraint.
  6. Does an installed dependency do it? Use it. Don't add a new dependency.
  7. If all answers are no, write the minimum code that works.
- When two options have the same size, choose the one that is correct on edge cases.
- Before you add a field that claims parity with another system, search that system for the name. A value that the other system computes and never stores is not parity.
- For a bug fix, fix the shared function once for every caller. A fix on only the ticket's path leaves the other callers broken.
- Modify what the repo has instead of adding a parallel path, and reuse its conventions: for example, one metric key with an outcome tag instead of three keys.
- Don't add code only to support a frozen legacy path. Tag the new path and let existing filters exclude the untagged rows.

## 4. Touch only what you must

- Don't restructure code to add a signal. Wrapping a function body in `try` re-indents every line and turns a 10-line change into 90. Take the smaller signal.
- Don't improve nearby code, comments or formatting, and don't refactor code that works.
- Match the existing style and naming. Apply the project's CLAUDE.md standards only to lines you already change.
- Remove imports, variables and functions that your change made unused. Mention unrelated dead code, but don't delete it.

## 5. Leave whitespace alone

Don't change whitespace on a line you don't otherwise change, unless a lint or format rule that the pipeline enforces requires it.

- Don't re-indent, reflow or rewrap existing lines. Don't add or remove blank lines, trailing whitespace, final newlines or line endings, and don't switch tabs and spaces.
- Format-on-save reformats whole files; format only your lines. In your own additions, follow the file's spacing and add no extra blank lines. Prefer shapes that don't shift existing lines, like a guard clause instead of a new `if` or `try` around a block.
- Minimal whitespace never means cramming code onto one line or breaking the linter.
- Put lint-required reformatting in its own commit so reviewers can skip it. If CI lints entire modified files, run the formatter check (for example `npx prettier --check`) on every file you touch, and fix existing violations in that separate commit.
- Before you push, compare `git diff --stat` with `git diff -w --stat`. Any difference is whitespace-only churn; revert what lint doesn't require. `git diff --check` flags trailing whitespace you added.

## 6. Keep test changes small

- New branching logic needs at least one assertion that fails if the logic breaks.
- Add assertions to existing tests that already drive the path. Write a new test only for a case that no existing test sets up.
- Delete a test that your change made redundant. If a new assertion covers the fallback, the separate fallback test is dead.
- Never test a mock of the thing you changed. That proves only that the mock works; find where the real logic is covered.

## 7. Sweep the branch diff against main

After each change, remove what the branch added that the codebase wouldn't:

- Comments that restate the code, tell the story of the plan, or are longer than 2 lines. Keep a 1-2 line comment that explains why. For a deliberate shortcut, keep one line that names its limit and when to upgrade it.
- Defensive checks or try/catch blocks that are unusual for the code path, unless they protect an item in Guardrails.
- Casts to `any` that only get past a type error.
- Deep nesting in new code. Flatten it with early returns. Leave existing code alone (see 4).
- Duplicate logic, or anything else that doesn't match the file. Consolidate it, and give new code clear names.

Stop when the remaining edits are broad rewrites. A focused edit that removes lines beats a rewrite that moves them.

## Report format

By default, report only. Don't edit files, commit, push or post comments unless the user asks.

Give each PR, branch or repo its own section, and list the lines the rules would remove:

- Start with the PR link and its size, for example `+127 / −3`. For a repo, start with the repo name and the files reviewed.
- Give one item per removal, with the filename, the lines, and the rule. If it spans more than one file, write `<x> files` (for example `3 files`).
- Give line numbers from the file on the PR's head branch, as `path:line`, never the line's position in `gh pr diff` output. Count from the `@@ +start` hunk header or grep the head branch. For a deleted line, give its base-branch number and say so.
- Start each item with the lines it removes, for example `−34`. Sort from most lines to fewest.
- Mark an item that changes a requirement, a feature or behavior as needing approval.
- End with two totals: the lines removed without approval, and the lines removed if the user approves every marked item.
- On request, use the short form: one line per item, `−<lines> <tag>: <what to cut>. <replacement>. path:line`. The tags are `delete` (dead or speculative code), `stdlib` (the standard library does it), `native` (the platform does it), `yagni` (an abstraction with one use) and `shrink` (the same logic in fewer lines).
- After the totals, add a Feature Consolidation section with the rule 1 proposals, sorted by lines removed. If there are no proposals, say so.

## Applying fixes

- Get approval for each change to a requirement, a feature or behavior, including a removed code path, guard, field or endpoint, or a simpler replacement feature. Approval for one item doesn't cover the others. Feature Consolidation proposals always need approval.
- "Fix everything" or "apply all" applies only the items that don't change a feature. List the skipped items in the summary.
- On PRs, edit only the lines those PRs change.
- Run the tests before and after. If a fix makes a test fail, revert it and report it.

## Guardrails

- Keep behavior unchanged unless you fix a clear bug.
- Never remove input validation at a trust boundary, error handling that prevents data loss, a security check, or basic accessibility, even when the check looks unusual for the code path.
- Choose clarity over brevity. No nested ternaries, dense one-liners, or functions that combine unrelated concerns. Keep an abstraction that already helps organize the code.
- Keep the final summary to 1-3 sentences, and mention only changes that affect how someone reads the code.
