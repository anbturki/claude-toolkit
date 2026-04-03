---
name: create-pr
description: Create a GitHub pull request following project conventions. Use when ready to submit changes.
disable-model-invocation: true
allowed-tools: Bash(git *, gh *)
---

# Create Pull Request

Create a PR following project conventions.

## Steps

1. Check current state:
   ```bash
   git status
   git log origin/main..HEAD --oneline
   git diff origin/main...HEAD --stat
   ```

2. Push branch if needed:
   ```bash
   git push -u origin $(git branch --show-current)
   ```

3. Create PR with HEREDOC:
   ```bash
   gh pr create --title "<title>" --body "$(cat <<'EOF'
   ## Summary
   - <bullet point 1>
   - <bullet point 2>

   ## Changes
   ### <Section>
   - <specific change>
   EOF
   )"
   ```

## PR Title Format

- Keep under 70 characters
- Use imperative mood
- Be specific about the change

Examples:
- `Add contact import from CSV`
- `Fix duplicate phone number validation`
- `Refactor campaign service to modules package`

## PR Body Format

```markdown
## Summary
- Brief bullet points describing the changes
- Focus on what and why

## Changes
### Backend
- Specific backend changes

### Frontend
- Specific frontend changes

### Database
- Schema or migration changes
```

## Rules

1. **No test plan section** - Don't include test checklists
2. **No time estimates** - Don't mention how long things took
3. **No AI references** - Don't mention Claude or AI assistance
4. **Focused PRs** - One feature/fix per PR
5. **Descriptive title** - Clear what the PR does

## Do NOT Include

- Test plan sections with checkboxes
- Time estimates
- AI/Claude references
- "Generated with" footers
