---
description: Create an isolated git worktree and run the given instructions inside it
argument-hint: <instructions describing the work to do>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
---

The user invoked `/worktree` with these instructions:

<instructions>
$ARGUMENTS
</instructions>

Do the following, in order:

## 1. Pick a worktree name

Derive a short, kebab-case name from the instructions above (e.g. `add-hold-piece`,
`fix-rotation-kick`, `dark-theme`). 2–4 words, lowercase, hyphen-separated, no spaces,
no slashes. This is `NAME`.

## 2. Create the worktree

Run these from the repo root:

```bash
git rev-parse --show-toplevel          # confirm repo root; cd there if needed
git worktree add .trees/NAME           # branches from current HEAD, new branch "NAME"
```

If `.trees/NAME` already exists or the branch name collides, append a short suffix
(`-2`, `-b`, or a topic word) and retry until it succeeds. Report the final path and
branch name.

`.trees/` is the container for all worktrees. Add `.trees/` to the repo's
`.gitignore` if it is not already ignored.

## 3. Work inside the worktree — isolated

Everything from here happens **only** under `.trees/NAME/`:

- Treat `.trees/NAME/` as your working root. Read, edit, and create files there.
- Do **not** touch files in the main working tree (the repo root outside `.trees/`).
- Do **not** switch branches in the main tree; the worktree has its own checked-out
  branch.
- Run builds, servers, and any verification with the worktree as the current
  directory.
- Commit to the worktree's branch as the work progresses (normal English commit
  messages, per repo convention). Do not merge back or push unless the user asks.

## 4. Execute the instructions

Carry out the `<instructions>` above within that worktree. When done, report:

- worktree path and branch name
- what changed (files, commits)
- how to verify (per `CLAUDE.md`: open `.trees/NAME/index.html` in a browser)
- how to remove it later: `git worktree remove .trees/NAME`
