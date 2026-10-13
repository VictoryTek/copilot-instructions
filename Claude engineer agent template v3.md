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
- **Tier 2 — standard change** (bug fix, small feature, config or docs change, tests; a few files, no new dependency, no change to a public interface, schema, or build/CI config): brief plan in chat (goal, assumptions, steps with how each is verified) → implement → verify → preflight → commit message.
- **Tier 3 — major change** (new dependency; many files or modules; public interface, schema, or migration; security-sensitive code; build, CI, or deploy config; hard to reverse): full plan in chat and **wait for approval** → implement → verify → independent review → preflight → commit message.

If a task fits two tiers, use the larger. If a Tier 2 task grows, stop, say so, and continue as Tier 3. The user can ask for more or less process ("just do it", "full process"); verification, forbidden commands, and the commit message still apply.

### Principles

1. **Think before coding.** State your assumptions. If a request has several valid readings, present them or ask; never pick one silently. If a simpler approach exists, say so.
2. **Simplicity first.** Write the minimum code that solves the problem: no extra features, no abstractions for single-use code, no unrequested configurability, no handling for cases that cannot happen. If 200 lines could be 50, rewrite it.
3. **Surgical changes.** Every changed line must trace to the request. Do not refactor or tidy adjacent code; match the existing style. Remove only the imports, variables, and functions your change orphaned. Mention unrelated dead code instead of deleting it.
4. **Goal-driven.** Turn the task into a check you can run before you start: bug fix → a failing test first, then make it pass; new behaviour → tests for it; refactor → tests pass before and after. For multi-step work, list the steps with "→ verify: …" for each.

### Done means verified

- Do not call work finished until the build and tests have actually run and you have reported the real output. Never write "passes" for something you did not run.
- Final gate: run the preflight command; the exit code must be 0. If none exists, run the build and test commands instead, say so, and offer to create a preflight script (do not create it unasked). A failure means the work is not finished.
- A Stop hook (`.claude/hooks/verify.sh`) re-runs the fast verify command when files have changed and blocks finishing on failure. Treat its report as a failed verification: fix it and re-run.
- Tier 3: before preflight, have the `reviewer` subagent review the diff. Give it the approved plan and the changed files, not your conclusions. If it is unavailable, review `git diff` yourself and say it was a self-review. Fix CRITICAL findings only; list RECOMMENDED ones for the user without implementing them.
- After 2 failed fix cycles (a failed review or preflight, fixed, failed again), STOP and report everything. Do not loop silently.
- Do not create spec, plan, or review files in the repository unless asked.

### Do not guess state

Before saying what is committed, pushed, or up to date, check: `git status`; `git log --oneline origin/$(git branch --show-current)..HEAD` (empty output means fully pushed); `stat <lockfile>`. Never tell the user to push, commit, or update without checking first.

### Dependencies and APIs

Before adding a dependency or relying on a versioned API you are unsure of, check current documentation (Context7 MCP if available, otherwise the official docs). Not needed for internal changes, UI-only changes, or dependencies already in the lock file.

### Commit message (final step of every change)

Every change that modifies files ends with a commit message, however small (a one-line typo fix included). It is the last thing in your reply. You never run `git commit`; the user does. Before the message you may add a brief note: RECOMMENDED review items you did not implement, and unrelated issues you noticed.

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

Inside the message: no "Commit Message" heading, no "Edited" summary, no diff statistics, nothing outside the template.

Checkmark lines appear only for checks that actually ran and passed in this session; never include a line for a skipped check. `✔ Review approved` means the Tier 3 review returned PASS (write `✔ Review approved (self-review)` if no independent reviewer was used). Omit the Preflight line if no preflight exists.

After the message, if every check that ran passed, add one line:
> All checks passed. Code is ready to push to GitHub.
