---
name: fix-pr-comments
description: Use when addressing PR feedback or review comments. Triggers include "fix PR comments", "address review", "implement PR feedback", "resolve review comments". Fetches comments, brainstorms fixes collaboratively, waits for approval, then implements changes.
---

# Fix PR Comments Workflow

## Overview

Systematically address PR review comments by fetching feedback, brainstorming solutions, and getting approval before making changes.

This skill helps turn PR feedback into actionable fixes through collaborative dialogue, ensuring alignment before implementation.

## The Process

**Fetching PR comments:**
- Use `gh pr view <pr-number> --comments` to fetch all review comments
- If no PR number provided, check current branch: `gh pr view --json number,comments`
- Parse and organize comments by:
  - File and line number
  - Reviewer
  - Comment type (change request, suggestion, question)
  - Status (unresolved vs resolved)
- Present comments in digestible groups

**Understanding the feedback:**
- For each comment or related group of comments, clarify intent:
  - What is the reviewer asking for?
  - Is this a blocking concern or suggestion?
  - What context is needed to address it?
- Ask questions one at a time to understand constraints
- Check relevant code sections to understand current implementation
- Identify dependencies between comments

**Brainstorming fixes:**
- For each comment (or group), propose 2-3 approaches:
  - Present trade-offs clearly
  - Include effort estimates (small/medium/large)
  - Note any risks or side effects
- Lead with your recommended approach and explain why
- Consider:
  - Minimal change vs comprehensive fix
  - Impact on other parts of the codebase
  - Test coverage implications
  - Performance and maintainability

**Presenting the fix plan:**
- Once approaches are selected, present a consolidated fix plan:
  - Break into logical sections (200-300 words each)
  - Group related changes together
  - Show before/after for code changes
  - List files that will be modified
  - Identify new tests needed
- Ask after each section: "Does this approach look right?"
- Be ready to revise based on feedback

**Getting approval:**
- Present complete fix plan with:
  - Summary of all changes
  - Files affected
  - Testing strategy
  - Any risks or considerations
- **Wait for explicit approval before implementing**
- Ask: "Ready to proceed with these fixes?"

## After Approval

**Implementation:**
- Implement fixes in the order agreed upon
- Create atomic commits per logical fix when possible
- Reference PR comments in commit messages
- Run tests after each significant change

**Verification:**
- Run full test suite
- Verify each comment is addressed
- Use `gh pr comment <pr-number> --body "<message>"` to respond to comments
- Mark conversations as resolved where appropriate

**Documentation:**
- Update PR description if scope changed significantly
- Add comments to code where reviewer questions revealed unclear logic
- Document any design decisions made during fix process

## Key Principles

- **One comment group at a time** - Don't overwhelm by addressing everything at once
- **Understand before proposing** - Read code context before suggesting fixes
- **Multiple approaches** - Always show trade-offs between solutions
- **Explicit approval required** - NEVER implement without user sign-off
- **Incremental validation** - Present plan in sections, validate each
- **Test-driven fixes** - Identify test gaps revealed by comments
- **Clear communication** - Explain reasoning, not just solutions

## Example Questions

**Understanding feedback:**
- "The reviewer mentioned 'this could be simplified' - should we prioritize readability or performance here?"
- "There are 3 comments about error handling - should we address these with a unified approach or case-by-case?"
- "This comment suggests caching - are we optimizing for speed or memory usage?"

**Proposing approaches:**
- "For the naming concern, we could: (1) use the reviewer's suggestion 'processUserData', (2) keep current name with better docs, or (3) rename to 'handleUserDataProcessing' for clarity. I recommend (1) as it's concise and clear."
- "To address the async concern: (1) add Promise.all for parallel execution (fast but complex), (2) sequential await (simple but slower), or (3) use async queue (balanced). I suggest (3) for maintainability."

## Integration with Other Skills

**After fixes are approved:**
- Use `git-standards` skill for committing with proper conventions
- Use `verification-before-completion` before pushing
- Use `test-driven-development` if adding new functionality

**If fixes require design changes:**
- Use `brainstorming` skill to explore new design
- Return to this skill once design is validated

## Edge Cases

**No PR comments:**
- Verify with user: "No comments found on this PR. Should I check a different PR?"

**Comments already resolved:**
- Show resolved comments separately
- Ask: "Do you want to review resolved comments or only open ones?"

**Blocking vs non-blocking:**
- Prioritize blocking comments (change requests)
- Group optional suggestions separately
- Ask: "Should we address suggestions in this round or defer?"

**Cross-cutting concerns:**
- If multiple comments point to systemic issues (e.g., "error handling inconsistent across files")
- Propose: "These 5 comments suggest a broader refactor. Should we: (1) fix locally as requested, (2) create a follow-up issue for systemic fix, or (3) address systemically now?"
