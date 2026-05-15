# Git Commands Reference

This file is my living Git cheat sheet. I will keep adding commands as I learn them.

## Setup & Config

- `git --version` - Shows the installed Git version.
  - Example: `git --version`
- `git config --global user.name "Your Name"` - Sets the name Git uses in commits.
  - Example: `git config --global user.name "Krish Patel"`
- `git config --global user.email "you@example.com"` - Sets the email Git uses in commits.
  - Example: `git config --global user.email "krish@example.com"`
- `git config --list` - Shows the current Git configuration.
  - Example: `git config --list`

## Basic Workflow

- `git init` - Creates a new Git repository in the current folder.
  - Example: `git init`
- `git status` - Shows what changed, what is staged, and what is not tracked yet.
  - Example: `git status`
- `git add <file>` - Moves changes into the staging area.
  - Example: `git add git-commands.md`
- `git commit -m "message"` - Saves staged changes as a snapshot.
  - Example: `git commit -m "Add first Git notes"`

## Viewing Changes

- `git diff` - Shows unstaged changes line by line.
  - Example: `git diff`
- `git diff --staged` - Shows changes that are already staged.
  - Example: `git diff --staged`
- `git log --oneline` - Shows commit history in a compact format.
  - Example: `git log --oneline`

## Branching & Undo

- `git branch` - Lists branches or creates a new one when given a name.
  - Example: `git branch feature-notes`
- `git switch <branch>` - Moves to another branch.
  - Example: `git switch feature-notes`
- `git checkout -- <file>` - Restores a file from the last commit.
  - Example: `git checkout -- git-commands.md`
- `git restore <file>` - Modern way to discard changes in a file.
  - Example: `git restore git-commands.md`
