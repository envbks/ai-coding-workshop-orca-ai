---
name: aura-git-standards
description: Use when user requests git operations following Aura standards. Triggers include "commit", "push", "create PR", "open PR", "make a PR". Handles JIRA ticket extraction, conventional commits, PR title format, and branch naming conventions.
---

# Aura Git Standards

Commit, push, and create PRs following Aura's internal standards for commit messages, PR titles, and branch naming.

## Workflow

1. **Check branch** - Exit if on protected branch (main, master, develop, pre-prod):
   ```bash
   git branch --show-current
   ```
   If protected, say "On protected branch, skipping." and stop.

2. **Check for changes** - Exit if nothing to commit:
   ```bash
   git status --porcelain
   ```

3. **Analyze changes for commit splitting** - Determine if changes should be split into multiple atomic commits:
   ```bash
   git diff --stat
   git status --porcelain
   ```

   **Group changes by:**
   - Feature area (directory/module)
   - Change type (new files, modifications, deletions)
   - Logical relationship (related functionality)

   **Split criteria - create separate commits when:**
   - Different directories with unrelated functionality
   - Mix of feature work and refactoring
   - Test files that could be committed separately
   - Config/dependency changes separate from code changes

   **If splitting is needed:**
   1. Present the proposed commit breakdown to user for approval
   2. For each logical group:
      - Stage only those files: `git add <specific-files>`
      - Commit with appropriate message following the format below
   3. Continue to push step after all commits are made

   **If changes are cohesive:** Proceed with single commit in step 5.

4. **Extract JIRA ticket from branch** - Branch format: `prefix/JIRA-123-description`
   ```bash
   git branch --show-current | grep -oE '[A-Z]{2,7}-[0-9]+'
   ```

5. **Stage and commit** - Use conventional commit format:
   ```bash
   git add -A
   git commit -m "feat: JIRA-123/ description of changes"
   ```
   Note: If step 3 determined changes should be split, this step was already handled with multiple targeted commits.

6. **Push**:
   ```bash
   git push -u origin HEAD
   ```

7. **Check for implementation plan** - Look for a plan file to use as PR description:
   - **Do NOT stage or commit plan files** unless the user explicitly asks to include them
   - Search for plan files in this order:
     1. Check staged/unstaged `.md` files (`git status --porcelain`) — look for files whose content contains plan indicators (e.g., "Implementation Plan", "## Tasks", "## Goal", numbered task lists with steps)
     2. Check `docs/plans/` directory if it exists
     3. Search the repo root for any `*plan*.md` or `*implementation*.md` files
   - If a plan file is found among staged changes, unstage it: `git reset HEAD <plan-file>`
   - Read the plan file content and use it **verbatim** as the PR body/description — do not summarize, rewrite, or replace it with a custom description
   - If multiple plans are found, ask the user which to use
   - **If no plan is found anywhere:** Inform the user:
     > "No implementation plan found. Plans make great PR descriptions and help reviewers understand your changes."
   - Then fall back to generating a summary from the diff as the PR body

8. **Create PR** - Title must match: `prefix: JIRA-ID/ description`
   - If a plan was found in step 7, pass its content **verbatim** as the PR body using `cat`:
     ```bash
     gh pr create --title "feat: JIRA-123/ description" --body "$(cat <path-to-plan-file>)"
     ```
     Do NOT rewrite or summarize the plan — the file content is the PR description.
   - If you need to update an existing PR's body (e.g. to fix a wrong description), use `gh api` directly instead of `gh pr edit` — `gh pr edit` silently fails due to a GitHub Projects classic deprecation issue:
     ```bash
     gh api --method PATCH repos/<owner>/<repo>/pulls/<number> --field body="$(cat <path-to-plan-file>)"
     ```
   - Otherwise, generate a summary from the changes:
     ```bash
     gh pr create --title "feat: JIRA-123/ description" --body "## Summary\n- Changes\n\n## Test plan\n- [ ] Tested"
     ```

## PR Title Format

Pattern: `^(([a-zA-Z])+(:){1}(\s){1}(((AE|ACDO|APROD|AU([A-Z]){2,})-\d*))\/([a-zA-Z\s]).*)`

Examples:
- `fix: AUPA-6616/ remove all feature flags`
- `feat: ACDO-1234/ add new feature`
- `chore: AU-999/ cleanup`

## Important

Do NOT include `Co-Authored-By` in commit messages.
