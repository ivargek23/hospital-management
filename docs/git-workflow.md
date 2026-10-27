# Git workflow
How we work with git in this repository.

Branches follow `<issue-number>-<item-id>-<short-description>`.

## Commit messages
- Imperative mood, one logical change per commit: `Add patient entity and migration (#73)`.
- Every commit references its issue number, so it appears on the issue timeline.

## Undoing changes
- Unstage but keep changes: `git restore --staged <file>`.
- Fix the last commit before pushing: `git commit --amend`.
- Undo a pushed commit: `git revert <hash>`.

## Stashing
- Untracked files are only stashed with `git stash push -u`.
