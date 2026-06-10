# agent-magic

A collection of Claude Code skills for professional software development workflows.

## Skills

| Skill | Description |
|-------|-------------|
| [fix-issue](skills/fix-issue/SKILL.md) | Structured bug-fix workflow: branch → investigate → fix → verify → document → PR |

---

## Installing a Skill

Skills live in `~/.claude/skills/<name>/SKILL.md`. To install:

```bash
# Clone this repo
git clone https://github.com/abhiiitr/agent-magic.git
cd agent-magic

# Copy the skill you want
cp -r skills/fix-issue ~/.claude/skills/fix-issue
```

That's it. Claude Code picks up skills automatically on the next session start.

---

## fix-issue

Guides Claude through every bug fix end-to-end:

1. **Branch** — creates a scoped `fix/<name>` branch from main
2. **Investigate** — traces the full data path (frontend → API → DB) before touching code; includes read-only DB access patterns
3. **Fix** — lint, type-check, and test after every change; adds missing test coverage
4. **Verify** — confirms the fix at the UI/endpoint surface, not just in tests
5. **Document** — writes a root-cause analysis doc in `docs/issues/`
6. **PR** — creates a PR and merges only when CI is clean

### Usage

The skill activates automatically when you ask Claude to fix a bug or unexpected behaviour. You can also invoke it explicitly:

```
Fix the issue where user avatars show "Unknown" on the profile page
```

Claude will follow the full workflow rather than jumping straight to a code change.

### Requirements

- Git repository with a `main` branch
- GitHub CLI (`gh`) for PR creation (optional — steps 1–5 work without it)
- `docs/issues/` directory for RCA docs (created on first use if absent)
