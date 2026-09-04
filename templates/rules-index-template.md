# Project Situational Rules Map

This index maps operational constraints to situational triggers. 

**Rule for Agents**: Read this table on session start or when beginning a new phase. Open the body of an individual rule file ONLY when your current task matches the trigger condition. Do not bulk-load all rules into your context window.

## Core Rules (Read on match)

| Read this when... | Rule file |
|---|---|
| Before writing or modifying any unit, integration, or contract test | `testing.md` |
| Before creating, altering, or designing API endpoints or data schemas | `api-standards.md` |
| Before staging, authoring git commit messages, or creating pull requests | `git.md` |
| Before executing recursive/force removal shell commands (e.g. `rm -rf`, force reset) | `safety-guard.md` |

