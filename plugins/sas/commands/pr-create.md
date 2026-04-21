---
description: Create a pull request with a meaningful title and description
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git log:*), Bash(git checkout:*), Bash(git push:*), Bash(git cm:*), Bash(gh pr create:*)
---

Review the current changes and create a pull request.

## Current State
- Status: !`git status --short`
- Branch: !`git branch --show-current`
- Staged diff: !`git diff --cached --stat`
- Diff from main: !`git diff main...HEAD --stat`
- Commits since main: !`git log main..HEAD --oneline`
- Recent commits: !`git log --oneline -5`

## Instructions
1. Review all changes since main (commits + unstaged/staged)
2. If on main, create a branch with a fitting name (e.g. `fix/short-topic`, `feat/short-topic`)
3. If there are uncommitted changes, stage and commit them using `git cm` (signed commits)
4. Push the branch with `git push -u o <branch>`
5. Create the PR via `gh pr create` with title and body following the rules below

## PR Title
- Use conventional commit style (`fix: ...`, `feat: ...`, `chore: ...`, etc.)
- Under 70 characters
- The PR title will become the squash merge commit subject, so it must follow conventional commits

## PR Description
The PR description will become the squash merge commit body, so it must follow commit message rules:
1. Use non-verbose, non-slop style
2. Use short crisp bullet points to describe the intention and the impact for the user / developer
3. Focus on reasoning instead of describing what has changed where (the mechanical description is not worth much)
4. If commits reference issues or the context makes it obvious, add `Closes #N` as the last bullet point

## Side Notes
- The remote `origin` is often named `o`
- Use signed commits via the git alias `git cm <message>`
- Use a HEREDOC for the PR body to preserve formatting

$ARGUMENTS
