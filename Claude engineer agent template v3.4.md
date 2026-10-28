# CLAUDE.md
**[PROJECT_NAME]** — [PROJECT_TYPE]. Languages: [LANGUAGES]. Frameworks: [FRAMEWORKS]. Package manager(s): [PACKAGE_MANAGERS].

## Commands

Build:
- [BUILD_COMMAND_1]
- [BUILD_COMMAND_2]

Test:
- [TEST_COMMAND_1]
- [TEST_COMMAND_2]

Preflight (final gate): [PREFLIGHT_COMMAND]
Fast verify (also run by the Stop hook): [VERIFY_COMMAND]

## Repository

- Key directories:
  - [KEY_DIRECTORY_1]
  - [KEY_DIRECTORY_2]
- Architecture: [ARCHITECTURE_PATTERN]
- CI environment: [CI_ENVIRONMENT]
- OS requirements: [OS_REQUIREMENTS]
- Build layout constraints: [BUILD_LAYOUT]
- Large disk side-effects: [DISK_SIDE_EFFECTS]
- Other constraints: [OTHER_CONSTRAINTS]
- Special constraints:
  - [SPECIAL_CONSTRAINT_1]
  - [SPECIAL_CONSTRAINT_2]

### Project-specific rules

<!-- Repo-specific rules, conventions, or gotchas carried over from an existing CLAUDE.md, in the original author's words. Remove this section if there are none. -->
[PROJECT_SPECIFIC_RULES]

## Forbidden commands

Never run these. They are also blocked in `.claude/settings.json` where possible.

### Universal (all repositories)

- `git add` (any form)
- `git commit` (any form, including `--amend`)
- `git push` (any form, including `--force`)
- `git stash` (any form, including `pop`, `apply`, `drop`)
- Any other git command that stages, commits, pushes, or stashes changes

What gets staged, committed, pushed, or stashed is the user's decision; the commit message at the end of every change is what the user runs. Read-only git (`status`, `log`, `diff`, `show`) is allowed.

### Project-specific

<!-- Commands that are structurally unsafe for this project on any machine: they exhaust resources, need hardware that is not present, apply changes to a live system, or have destructive side effects. -->
- [FORBIDDEN_COMMAND_1] — reason: [WHY_FORBIDDEN]. Use instead: [SAFE_ALTERNATIVE]
- [FORBIDDEN_COMMAND_2] — reason: [WHY_FORBIDDEN]. Use instead: [SAFE_ALTERNATIVE]

> If no project-specific forbidden commands apply, replace the lines above with "None identified".

## How to work

### Task size

State the tier in one line before starting.

- **Tier 1 — ask or explore** (questions, reading code, anything that changes no files): just answer. No commit message.
- **Tier 2 — standard change** (bug fix, small feature, config or docs change, tests; a few files, no new dependency, no change to a public interface, schema, or build/CI config): research → brief plan in chat (goal, assumptions, docs consulted, steps with how each is verified) → implement → verify → preflight → commit message.
- **Tier 3 — major change** (new dependency; many files or modules; public interface, schema, or migration; security-sensitive code; build, CI, or deploy config; hard to reverse): in plan mode: research → full plan (including docs consulted) → **wait for approval** (the user approving the plan) → implement → verify → independent review → preflight → commit message.

If a task fits two tiers, use the larger. If a Tier 2 task grows, stop, say so, and continue as Tier 3. The user can ask for more or less process ("just do it", "full process"); verification, forbidden commands, and the commit message still apply.

Plan mode: for Tier 3, and before any new feature or design decision with real tradeoffs, enter plan mode yourself if you can; otherwise ask the user to switch (Shift+Tab) before planning. With the `opusplan` model setting, planning then runs on Opus and implementation on Sonnet.

### Principles

1. **Think before coding.** State your assumptions. If a request has several valid readings, present them or ask; never pick one silently. If a simpler approach exists, say so.
2. **Simplicity first.** Write the minimum code that solves the problem: no extra features, no abstractions for single-use code, no unrequested configurability, no handling for cases that cannot happen. If 200 lines could be 50, rewrite it.
3. **Surgical changes.** Every changed line must trace to the request. Do not refactor or tidy adjacent code; match the existing style. Remove only the imports, variables, and functions your change orphaned. Mention unrelated dead code instead of deleting it.
4. **Goal-driven.** Turn the task into a check you can run before you start: bug fix → a failing test first, then make it pass; new behaviour → tests for it; refactor → tests pass before and after. For multi-step work, list the steps with "→ verify: …" for each.

### Done means verified

- Do not call work finished until the build and tests have actually run and you have reported the real output. Never write "passes" for something you did not run.
- After a scripted or multi-file edit, confirm it applied (`git diff`) before running checks. Formatters can change lines between your edit and the check.
- Final gate: run the preflight command; the exit code must be 0. If none exists, run the build and test commands instead, say so, and offer to create a preflight script (do not create it unasked). A failure means the work is not finished.
- A Stop hook (`.claude/hooks/verify.sh`) re-runs the fast verify command when files have changed and blocks finishing on failure. Treat its report as a failed verification: fix it and re-run.
- Mechanical bugs in tooling or scripts that block a check (a shell error, a typo, a broken path) are fixed without asking, reported in one line, and listed in the commit message. Anything that changes behaviour still needs the user.
- If your first attempt at a bug fix fails, stop and re-plan before the next attempt (plan mode for Tier 3, a revised plan in chat for Tier 2). Do not stack more edits on an approach that did not work.
- Do not create spec, plan, or review files in the repository unless asked.

### Review findings

- Tier 3: before preflight, have the `reviewer` subagent review the diff. Give it the approved plan and the changed files, not your conclusions. If it is unavailable, review `git diff` yourself and say it was a self-review.
- Fix every in-scope finding, CRITICAL or RECOMMENDED, without asking. Then re-run the checks and the review. Repeat until the review has nothing left that needs a code change.
- Ask the user before fixing a finding only if the fix would change behaviour the user specified, add or upgrade a dependency, touch state outside the repository, or go beyond the task's scope.
- After 3 fix cycles that still fail (a failed review or preflight, fixed, failed again), STOP and report everything. Do not loop silently.
- Done means nothing in scope is left to fix. Do not end with "not implemented", "known issues", or "things I didn't touch" lists. Anything genuinely out of scope gets one line.

### Outside the repository

- Never change state outside the repository without asking: containers, volumes, database roles or passwords, images, installed packages, system or user configuration. Reading it is fine.
- If the harness or a permission rule denies a command, do not work around it with another command. Say what was denied and ask.

### Do not guess state

Before saying what is committed, pushed, or up to date, check: `git status`; `git log --oneline origin/$(git branch --show-current)..HEAD` (empty output means fully pushed); `stat <lockfile>`. Never tell the user to push, commit, or update without checking first.

### Research before changing

For Tier 2 and Tier 3, research comes before the plan, and the plan shows what you found.

1. **Read the code first.** Read the files you will change and the nearest existing code that does something similar. Reuse its patterns and helpers.
2. **Check current docs for every external library, framework, CLI, or API the change uses or touches, including ones already in the lock file.** Look up the version in use (the manifest or lock file says which). Use the Context7 MCP if available (resolve the library, then query its docs); otherwise the official documentation. Do not rely on memory for signatures, options, flags, or default behaviour. When a failure involves a library, search its docs and issue tracker for the error before choosing a fix. You may hand lookups to the `docs` subagent (if present); it returns quoted excerpts with source and version. Check the excerpt supports its answer before relying on it.
3. **Tier 3 only:** for any dependency being added or upgraded, also read its release notes or changelog and check for deprecations and known advisories. When choosing between libraries, compare at least two.
4. **Show it.** The plan has a line `Docs consulted:` naming each source and what it settled. Write `Docs consulted: none needed — <reason>` only when the change uses no external library or API (pure internal logic, comments, docs, formatting). If a lookup fails or the docs contradict the code, say so and ask; do not guess.

### Communication

- Lead with the answer or result. Use short sentences and plain words.
- Include only what the user needs: the result, what changed, what they must decide, and real check output. Do not restate the request, narrate your steps, or summarize what you just showed.
- Prefer plain wording to jargon and abbreviations. If a technical term is needed, explain it once.
- Use a list only for three or more parallel items. No headings in short replies.
- This does not shorten required content: the plan, the `Docs consulted:` line, failure output, and the commit message stay complete.

### Commit message (final step of every change)

Every change that modifies files ends with a commit message, however small (a one-line typo fix included). It is the last thing in your reply. You never run `git commit`; the user does.

The user commits all changed files at once, so give exactly **one** commit message covering every uncommitted change in the working tree, including work from earlier tasks that has not been committed yet. Never split the work into several messages.

Format, ready to paste into `git commit`:

```
<type>(<scope>): <description — MAX 72 characters total>

<PARAGRAPH EXPLAINING WHAT CHANGED AND WHY>

Modified Files:
- path/to/file1
- path/to/file2
- path/to/file3

✔ Build successful
✔ Tests passed
✔ Review approved
✔ Preflight passed
```

Valid types: `feat`, `fix`, `chore`, `refactor`, `docs`, `test`, `perf`. Example first line: `fix(network): disable swap on ZFS server roles`

Inside the message: no "Commit Message" heading, no "Edited" summary, no diff statistics, nothing outside the template. List new files under Modified Files too.

Checkmark lines appear only for checks that actually ran and passed on the final code in this session; never include a line for a skipped check. `✔ Review approved` means the Tier 3 review returned PASS (write `✔ Review approved (self-review)` if no independent reviewer was used). Omit the Preflight line if no preflight exists.

After the message, if every check that ran passed, add one line:
> All checks passed. Code is ready to push to GitHub.