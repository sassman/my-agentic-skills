---
description: Create a git commit with a meaningful message
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Bash(git log:*)
---

Review the current changes and create an appropriate git commit.

## Current State
- Status: !`git status --short`
- Staged diff: !`git diff --cached --stat`
- Recent commits: !`git log --oneline -5`

## Instructions
1. Review staged and unstaged changes
2. Stage relevant files if needed
3. Create a commit with a clear, concise message following conventional commit style
  1. use non-verbose, non-slop style
  2. use short crisp bullet points to describe the intentation and the impact for the user / developer
  3. focus on reasoning instead of describing what has changed where (the mechanical description is not worth much)

## Side Notes
- the remote `origin` is often named `o`
- use signed commits (I use the git alias `git cm <message>`)
