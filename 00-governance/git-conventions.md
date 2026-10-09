# Git conventions

> Adapted to the challenge README. Confirm any workflow not specified there with the repository owner.

## 1. Repository
- Work in the team's fork of the challenge repository.
- Before pushing, use git remote -v and confirm origin points to the fork, not upstream code-sena.
- Keep this challenge separate from the main application repository.

## 2. Stage and commit
Stage only intended files and review git status before committing.

Example:
git add 04-requirements/non-functional.md
git commit -m "docs(requirements): add non-functional requirements"

The challenge provides docs(scope): description-style examples. Broader branch and pull-request policy: [TODO].

## 3. Push and verify
Push the commit to origin main. Check git status and git log --oneline -5. Confirm the commit appears on the fork before the deadline.

If a push is rejected, inspect the repository state and follow the challenge README's rebase guidance.

## 4. Branches and pull requests
The challenge README does not require a branch or pull request workflow. Team policy: [TODO].
