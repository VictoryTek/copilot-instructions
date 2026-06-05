A few usage notes:

Mode to use: 
Plan mode + High effort for this prompt. It's pure read and analysis — no edits — so Plan mode is perfect, and High effort is worth it here since the whole point is thorough examination.

When to run it: 
At the start of a new feature cycle, after a long period of accumulated changes, or when onboarding to an unfamiliar repo. Running it too frequently on a fast-moving project just produces stale reports.

The output file: 
It saves to .github/docs/project_audit_[date].md so you build up a history over time. You can diff two audits to see if the health score is trending up or down between sessions.

Pairing with your CLAUDE.md workflow: If the audit surfaces a Critical or High issue you want to fix, that naturally becomes the next USER REQUEST fed into your 7-phase orchestrator workflow — the audit essentially generates a prioritised backlog for you.
