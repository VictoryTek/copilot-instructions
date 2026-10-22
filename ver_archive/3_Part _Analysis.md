Session 1 — Architecture & Structure (/effort xhigh)
Analyze this entire codebase for architecture and structure only. I want:

1. Architectural anti-patterns or design problems
2. Structural inconsistencies (naming conventions, file/folder organization, module boundaries)
3. Inconsistent patterns — things done one way in some places and differently in others (error handling, async patterns, data flow, etc.)
4. Anything that looks half-implemented or abandoned
5. Dependencies that seem unnecessary, misused, or outdated

For every finding: give it a priority (high/medium/low), the specific file(s) and line numbers involved, and a concrete explanation of why it's a problem. Do not summarize vaguely. When you're done, write the full findings to ANALYSIS_ARCH.md in the project root.






Session 2 — Bugs & Code Quality (/effort xhigh)
Analyze this entire codebase for bugs and code quality issues only. I want:

1. Logic errors or likely bugs, including edge cases that aren't handled
2. Security vulnerabilities or unsafe patterns
3. Performance problems (unnecessary re-renders, inefficient queries, memory leaks, etc.)
4. Dead code, redundant code, or code that does nothing useful
5. Error handling that's missing, inconsistent, or swallows failures silently

For every finding: give it a priority (high/medium/low), the specific file(s) and line numbers involved, and a concrete explanation of the problem and what could go wrong. Do not summarize vaguely. When you're done, write the full findings to ANALYSIS_BUGS.md in the project root.






Session 3 — Feature Recommendations (/effort high)
Analyze this entire codebase and suggest real, specific features that are worth building. I want:

1. Features that are partially stubbed out or clearly intended but never finished
2. Features that would naturally complement what's already here based on the existing code and data models
3. Gaps where users of an app like this would obviously expect something that's missing
4. Any integrations or automations that the current structure is already set up for but not using

For every recommendation: explain what already exists that makes this a natural fit, describe the feature concretely (not vaguely), and give it a priority (high/medium/low) based on effort vs. value. Do not recommend things that would require a complete rearchitecture. When you're done, write the full findings to ANALYSIS_FEATURES.md in the project root.






Session 4 — switch to Sonnet and run this:
Read ANALYSIS_ARCH.md, ANALYSIS_BUGS.md, and ANALYSIS_FEATURES.md. Consolidate them into a single prioritized task list in MASTER_PLAN.md — all high priority items first across all three categories, then medium, then low. Remove any duplicates. Then start with the first high priority item, show me the proposed change before applying it, and check it off the list as we go.