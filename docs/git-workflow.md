# Git workflow

How git is used on the hospital-management project.

## Branch naming convention
`<issue-number>-<item-id>-<short-description>`, lowercase, words separated by hyphens
E.g. `37-t-pl01-1-spring-boot-skeleton` for issue #37, task T-PL01.1.

- One branch per task, or per small user story without sub-tasks.
- Always created from an up-to-date `main`, preferably from the issue (Development → Create a branch).
- `main` is protected: changes reach it only through a pull request; force pushes and deletion are blocked.

## Commit messages
- Imperative mood, one logical change per commit: `Add patient entity and migration (#73)`.
- Every commit references its issue number, so it appears on the issue timeline.

## Undoing changes
- Unstage but keep changes: `git restore --staged <file>`.
- Fix the last commit before pushing: `git commit --amend`.
- Undo a pushed commit: `git revert <hash>`.

## Stashing
- Untracked files are only stashed with `git stash push -u`.
- `git stash pop` applies and removes the stash; `git stash apply` applies and keeps it.

## Workflow: from issue to release
1. **Issue**: move the task to *In Progress* on the Kanban board.
2. **Branch**: create it from the issue, then `git fetch` and `git switch <branch>`.
3. **Commit**: small commits, each referencing the issue (`#<n>`).
4. **Interrupt**: `git stash push -u -m "WIP ..."`, fix the urgent issue on its own branch, come back and `git stash pop`.
5. **Push**: `git push -u origin <branch>`.
6. **Pull request**: open it into `main` with `Closes #<n>`, move the card to *In Review*, review the diff and leave a review comment.
7. **Merge**: the issue closes and the card moves to *Done* automatically.
8. **Sync**: `git switch main && git pull`, `git branch -d <branch>`, `git fetch --prune`.
9. **Release**: `git tag -a vX.Y.Z -m "..."`, `git push origin vX.Y.Z`, publish a GitHub Release from the tag.

## Example: T-PL01.1 (#37) to release v0.1.0
Task #37 was carried through all nine steps. The interrupt was #113 (manual test checklist, PR #114); the task itself was merged in PR #115.

Release: https://github.com/ivargek23/hospital-management/releases/tag/v0.1.0

Commit graph after the release:

```text
*   3151a1c (HEAD -> main, tag: v0.1.0, origin/main, origin/HEAD) Merge pull request #115 from ivargek23/37-t-pl01-1-spring-boot-skeleton
|\  
| * 3254e3b Switch README to Maven and pin PostgreSQL test image (#37)
| * 6cc7883 Generate Spring Boot project skeleton with Maven (#37)
* |   94bfa7b Merge pull request #114 from ivargek23/113-t-doc-manual-tests
|\ \  
| |/  
|/|   
| * 156ae58 Add manual test checklist (#113)
|/  
* 750b374 Remove leftover conflict markers from workflow doc (#29)
* 3ac5c83 Clarify workflow doc intro (#29)
* 033ef1d Reword workflow doc intro (#29)
* 4cfca74 Add stash note to workflow doc (#29)
* 2385996 Revert "Add undo rules to workflow doc (#29)"
* 5cb9f7a Revert "Link git workflow doc from README"
*   e11458e Merge branch 'conflict-demo'
|\  
| * bc9e136 Reword project title in README
* | 51d46d8 Rename project title in README
|/  
* 74c17de Link git workflow doc from README
* 1ca791f Add undo rules to workflow doc (#29)
*   3147318 Merge branch 'docs-undo-rules'
|\  
| * f5992b5 Add undo rules to workflow doc (#29)
* | 674901d Link git workflow doc from README
|/  
* c8d9bf9 Add commit message convention to workflow doc (#29)
* 805b01d Add branch naming rule to git workflow doc (#29)
* f611606 Add git workflow document
* f23fb10 Remove duplicate sprint process section from README
* 22b5a20 Add link to the GitHub Project board
* 94c85b9 Document branching and commit conventions
* 1dcd281 add issue templates for: user stories, tasks
* c019cd4 add requirement analysis, backlog, decomposition, tech stack
```
