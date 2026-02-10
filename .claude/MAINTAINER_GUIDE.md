# SkillKit Maintainer Guide

> **INTERNAL USE ONLY** - This file is for maintaining the SkillKit repository itself. Never push to GitHub.

This guide is for Claude and maintainers working on SkillKit itself, not for users of SkillKit.

## Important Distinctions

**This is NOT a typical application project:**
- SkillKit is a skills repository/template system
- It doesn't have a build process, linter, or typical dev dependencies
- The workflows in `skills/workflow/` are for projects that USE SkillKit, not for SkillKit itself
- Don't follow the same rules that apply to user projects

## Adding New Skills

When adding a new skill to SkillKit:

### 1. Create the Skill File

```bash
# Create directory structure
mkdir -p .claude/skills/{category}/{skill-name}

# Create SKILL.md with frontmatter
```

**Skill file structure:**
```markdown
---
name: skill-name
description: Brief description. Trigger words - word1, word2, word3
---

# Skill Name

[Content following existing skill patterns]
```

### 2. Update Documentation

Update **THREE** files (in this order):

#### A. `.claude/CLAUDE.md`
Add entry to the skills table:
```markdown
| When the task involves... | Use this skill |
|---------------------------|----------------|
| Your description | `category/skill-name` |
```

#### B. `README.md` - Skills List
Add to the appropriate category in "Available Skills" section:
```markdown
### Category
| Skill | What It Does |
|-------|--------------|
| `skill-name` | Brief description |
```

#### C. `README.md` - Changelog
- **If adding to existing version:** Add bullet under current version
- **If creating new version:** Create new version section at top

### 3. Update Version Files

**For new version releases:**

Update **TWO** files:

#### A. `README.md` Header
```markdown
**Version:** 1.1.1
**Date:** DD/MM/YYYY  ← Use this format! Not YYYY-MM-DD
```

#### B. `.claude/version.json`
```json
{
  "version": "1.1.1",
  "releaseDate": "YYYY-MM-DD",  ← ISO format here
  "repository": "webclay/skillkit",
  "changelogUrl": "https://github.com/webclay/skillkit/releases"
}
```

**Note the date format difference:**
- README.md uses: `DD/MM/YYYY` (e.g., "10/02/2026")
- version.json uses: `YYYY-MM-DD` (e.g., "2026-02-10")

## Git Workflow for SkillKit

### Creating a Feature Branch

```bash
# Create from main
git checkout main
git pull origin main
git checkout -b feature/descriptive-name
```

### Committing Changes

**DO NOT run linters** - SkillKit doesn't have ultracite configured.

```bash
# Stage changes
git add .

# Commit with conventional format
git commit -m "Feature: Add [skill-name] skill

- Created new skill at .claude/skills/category/skill-name
- Updated CLAUDE.md skills table
- Updated README.md skills list and changelog
- Updated version to X.X.X

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

### Pushing and Creating PR

```bash
# Push to remote
git push -u origin feature/descriptive-name

# Create PR with gh CLI
gh pr create --base main --title "Feature: Add [skill-name] skill" --body "$(cat <<'EOF'
## Summary
Brief description of what was added

## Changes
- Created skill file
- Updated documentation
- Updated version (if applicable)

## Notes
Any relevant context

🤖 Created with Claude Code
EOF
)"
```

### Handling Merge Conflicts

**Common conflict: README.md date format**

When merging from main, the date format conflict occurs because:
- Your branch changed the date to today
- Main branch has a different date

**Resolution:**
1. Keep main's date format: `DD/MM/YYYY`
2. Resolve conflict manually:
```bash
git pull --no-rebase origin main
# Edit README.md to resolve conflict markers
git add README.md
git commit -m "Merge main into feature branch - resolve date conflict"
git push origin feature/branch-name
```

## Common Mistakes to Avoid

### 1. Don't Run Linters
❌ `npx ultracite fix` - This repo doesn't have linting configured
✅ Skip linting step when following push workflows

### 2. Date Format Consistency
❌ `**Date:** 2026-02-10` in README.md
✅ `**Date:** 10/02/2026` in README.md

❌ `"releaseDate": "10/02/2026"` in version.json
✅ `"releaseDate": "2026-02-10"` in version.json

### 3. Changelog Updates
❌ Adding to old version after creating new version
✅ Create new version section, move new entries there

### 4. Version Bumping
When to bump version:
- **Patch (1.1.0 → 1.1.1)**: New skill, minor docs update
- **Minor (1.1.0 → 1.2.0)**: Multiple skills, significant feature
- **Major (1.0.0 → 2.0.0)**: Breaking changes to structure

### 5. Workflow Skills
❌ Using `/push`, `/branch`, `/log` workflows from skills/workflow/
✅ These are for user projects, not for SkillKit maintenance
✅ Follow manual git commands in this guide instead

## Files That Should NOT Be Modified

- `.claude/project/` - This is for SkillKit's own project management
- `.claude/settings.json` - User-specific IDE settings
- `.claude/templates/` - Only update if changing template patterns
- `.claude/commands/` - Only update if changing command definitions

## Files to ALWAYS Update When Adding Skills

1. `.claude/skills/{category}/{skill-name}/SKILL.md` - The skill itself
2. `.claude/CLAUDE.md` - Skills reference table
3. `README.md` - Available Skills section
4. `README.md` - Changelog section
5. `README.md` - Version header (if new version)
6. `.claude/version.json` - Version info (if new version)

## Checklist for Adding a Skill

- [ ] Create skill file with proper frontmatter
- [ ] Follow existing skill structure/format
- [ ] Add to CLAUDE.md skills table
- [ ] Add to README.md Available Skills section
- [ ] Update or create changelog entry
- [ ] Update version numbers if creating new version
- [ ] Use correct date formats (DD/MM/YYYY in README, YYYY-MM-DD in version.json)
- [ ] Commit with conventional message format
- [ ] Create PR with detailed description
- [ ] Don't run linters (not configured for this repo)

## Testing a Skill

Before pushing:
1. Copy `.claude` folder to a test project
2. Test that trigger words activate the skill
3. Verify skill instructions are clear and actionable
4. Check that examples work in practice

## After PR Merge

1. Pull latest main: `git checkout main && git pull origin main`
2. Delete feature branch: `git branch -D feature/branch-name`
3. Delete remote branch: `git push origin --delete feature/branch-name`
4. Tag release if new version: `git tag v1.1.1 && git push --tags`

## Questions?

If uncertain about any step, ask the user before proceeding. It's better to clarify than to make mistakes that require manual fixes.
