# Day 22 Notes: Git Basics

## 1) What is Git?

- Git is a version control system.
- It tracks changes in files over time.
- It helps teams work on the same project without overwriting each other.

## 2) What is the difference between `git add` and `git commit`?

- `git add` sends changes to the staging area.
- `git commit` saves the staged changes as a permanent snapshot in the repository.
- Simple idea: `git add` is like packing items into a box, and `git commit` is sealing the box and labeling it.

## 3) What does the staging area do?

- The staging area is a holding place before commit.
- It lets me choose exactly which changes should be included in the next commit.
- Git does not commit directly because I may want to combine, separate, or review changes first.

## 4) What does `git log` show?

- `git log` shows commit history.
- It includes commit ID, author, date, and commit message.
- It helps me see how the project changed over time.

## 5) What is the `.git/` folder?

- `.git/` stores the internal database of the repository.
- It contains history, references, objects, configuration, and branch information.
- If I delete it, Git no longer knows this folder is a repository.

## 6) Working directory vs staging area vs repository

- Working directory: the files I edit on my computer.
- Staging area: the place where I prepare changes for the next commit.
- Repository: the saved history inside `.git/`.

## Quick summary

- Git tracks file history.
- `git add` prepares changes and `git commit` saves them.
- The staging area helps me build clean commits.

## Key takeaways

- Git is about snapshots, not just file copying.
- Understanding the three areas helps me avoid messy commits.
