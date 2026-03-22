# How The Library Works — A Concrete Guide

This doc explains the library system using real examples from this setup.

## The Mental Model

There are **3 layers** to understand:

```
1. SOURCE REPOS        Where skill code actually lives (GitHub)
2. THE CATALOG         library.yaml — just pointers to source repos
3. INSTALLED COPIES    Local copies pulled from source repos via /library use
```

That's it. The catalog points to sources. You install from sources. You edit installed copies. You push changes back to sources.

## Your Current Setup

### Layer 1: Source Repos (GitHub)

These repos contain the actual skill code:

| Repo | What's in it |
|---|---|
| `t6tv8e/t6tv8e-agentic-engineering` | Your own skills (easytranscriber) |
| `vercel-labs/agent-skills` | Vercel's skills (6 skills) |
| `mattpocock/skills` | Matt Pocock's skills (7 skills) |
| `anthropics/skills` | Anthropic's skills (skill-creator) |

### Layer 2: The Catalog (`~/.claude/skills/library/`)

This is the `t6tv8e/the-library` repo. It contains:
- `library.yaml` — pointers to the 15 skills across the source repos above
- `SKILL.md` — the library skill itself (the brain)
- `cookbook/` — step-by-step instructions for each command

**The catalog stores zero skill code.** It only stores URLs.

### Layer 3: Installed Copies

When you run `/library use <name>`, the skill gets cloned from its source repo
and placed locally:

| Install type | Directory | When to use |
|---|---|---|
| Project-local (default) | `.claude/skills/<name>/` | Skill only needed in one project |
| Global | `~/.claude/skills/<name>/` | Skill needed everywhere on this machine |

Right now you have one installed skill:
- `~/.claude/skills/easytranscriber/` (installed globally)

The other 14 skills are cataloged but not installed. They exist only as
pointers in library.yaml until you `/library use` them.

## The Workflow

### Day-to-day: Using skills across projects

```
/library use deploy-to-vercel           Install into current project
/library use deploy-to-vercel globally  Install for all projects on this machine
```

### After editing an installed skill

```
# You edited ~/.claude/skills/easytranscriber/SKILL.md
/library push easytranscriber           Pushes back to t6tv8e-agentic-engineering on GitHub
```

### Getting the latest version

```
/library use easytranscriber            Re-pulls from GitHub, overwrites local copy
/library sync                           Re-pulls ALL installed skills from GitHub
```

### Adding a new skill to the catalog

```
/library add my-new-skill from https://github.com/me/repo/blob/main/skills/my-new-skill/SKILL.md
```

This only adds a pointer to library.yaml. Nothing is installed yet.

## Common Confusion Points

### "Where do I edit skills?"

Edit the **installed copy** (in `.claude/skills/` or `~/.claude/skills/`),
then `/library push` to send changes back to the source repo.

Do NOT edit the source repo directly. The library manages that for you.

### "What if I built a skill inside a project?"

This is normal — you author skills inside the project they serve. But once
you want to reuse it elsewhere:

1. Create a source repo (or use an existing one like `t6tv8e-agentic-engineering`)
2. Push the skill there: `/library add` to catalog it, then manually push the
   code to the source repo (first time only)
3. Delete the original copy from the project
4. `/library use <name>` to reinstall from the source repo
5. From now on, the library manages it

### "What does `/library push` actually do?"

1. Finds the installed copy on your machine
2. Clones the source repo into a temp directory
3. Replaces the skill directory in the clone with your local version
4. Commits and pushes to GitHub
5. Cleans up the temp directory

It looks for installed copies in the default and global directories only.
It will NOT find skills in random project paths.

### "What if someone else updates the source repo?"

Your installed copy is now stale. Run:
```
/library use <name>      # refresh one skill
/library sync            # refresh all installed skills
```

### "Is this like npm/pip?"

Similar idea, simpler execution:
- **npm** has a registry, versions, lock files, dependency resolution
- **The library** has GitHub URLs, always-latest, no versions, typed dependencies

Think of it as `package.json` without the complexity.

## Visual Summary

```
  YOU (editing)
    |
    v
  INSTALLED COPY              /library push
  ~/.claude/skills/foo/  ──────────────────>  SOURCE REPO
                                              github.com/you/repo
  /library use                                     |
  <────────────────────────────────────────────────-+
    |
    v
  INSTALLED COPY
  (refreshed)


  CATALOG (library.yaml)
  Just stores the URL:
  source: https://github.com/you/repo/blob/main/skills/foo/SKILL.md
```
