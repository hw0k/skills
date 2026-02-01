# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository is a skills workspace for Claude Code. Skills are modular packages that extend Claude's capabilities by providing specialized knowledge, workflows, and tool integrations.

## Architecture

### Directory Structure

```
.
├── .agents/skills/skill-creator/    # The skill-creator skill (meta-skill for building skills)
│   ├── SKILL.md                     # Skill definition and creation guide
│   ├── scripts/                     # Skill management utilities
│   │   ├── init_skill.py           # Initialize new skills from template
│   │   ├── package_skill.py        # Validate and package skills for distribution
│   │   └── quick_validate.py       # Quick skill validation
│   └── references/                  # Design pattern documentation
│       ├── workflows.md            # Sequential and conditional workflow patterns
│       └── output-patterns.md      # Template and example patterns
├── .claude/skills/                  # Symlinks to active skills
│   └── skill-creator -> ../../.agents/skills/skill-creator
└── skills/                          # User-created skills directory
    └── test-skill/                 # Example skill demonstrating structure
```

### Skill Anatomy

Every skill consists of:
- **SKILL.md** (required): YAML frontmatter metadata + Markdown instructions
- **Bundled Resources** (optional):
  - `scripts/` - Executable code for deterministic tasks
  - `references/` - Documentation loaded into context as needed
  - `assets/` - Files used in output (templates, boilerplate, etc.)

### Progressive Disclosure

Skills use a three-level loading system:
1. **Metadata** (name + description) - Always in context (~100 words)
2. **SKILL.md body** - Loaded when skill triggers (<5k words, prefer <500 lines)
3. **Bundled resources** - Loaded as needed by Claude

## Common Development Tasks

### Creating a New Skill

```bash
# Initialize from template
.agents/skills/skill-creator/scripts/init_skill.py <skill-name> --path skills/

# The script creates:
# - skills/<skill-name>/SKILL.md with frontmatter template
# - Example directories: scripts/, references/, assets/
```

### Validating a Skill

```bash
# Quick validation
.agents/skills/skill-creator/scripts/quick_validate.py skills/<skill-name>

# Checks:
# - YAML frontmatter format and required fields (name, description)
# - Skill naming conventions
# - Description completeness
# - File organization
```

### Packaging a Skill

```bash
# Package skill into distributable .skill file
.agents/skills/skill-creator/scripts/package_skill.py skills/<skill-name>

# Optional: specify output directory
.agents/skills/skill-creator/scripts/package_skill.py skills/<skill-name> ./dist

# This:
# 1. Validates the skill automatically
# 2. Creates a .skill file (zip with .skill extension) if validation passes
```

## Key Principles

### Conciseness
- Skills share the context window with everything else
- Only add context Claude doesn't already have
- Challenge each piece of information for token cost justification
- Prefer concise examples over verbose explanations
- Keep SKILL.md body under 500 lines

### Skill Metadata (Frontmatter)

Required fields in SKILL.md YAML frontmatter:
- `name`: The skill name
- `description`: **Primary triggering mechanism** - Include what the skill does AND when to use it. All "when to use" information must be here (not in body) since the body is only loaded after triggering.

### Content Organization

When SKILL.md approaches 500 lines, split content into separate files:

**Pattern 1: High-level guide with references**
- Keep core workflow in SKILL.md
- Move detailed info to references/ files
- Reference them clearly from SKILL.md

**Pattern 2: Domain-specific organization**
- For multi-domain skills, organize references by domain
- Claude only loads relevant domain files when needed

**Pattern 3: Framework/variant-specific**
- Keep selection guidance in SKILL.md
- Move variant-specific details to separate reference files

### File Organization Rules

- Avoid deeply nested references (keep one level deep from SKILL.md)
- For files >100 lines, include table of contents at top
- Delete unused example files created by init_skill.py
- Do NOT create auxiliary documentation (README.md, CHANGELOG.md, etc.) - skills should only contain what Claude needs

## Workflow

1. **Understand** - Gather concrete examples of skill usage
2. **Plan** - Identify reusable resources (scripts, references, assets)
3. **Initialize** - Run init_skill.py to create template
4. **Edit** - Implement resources, update SKILL.md
5. **Package** - Run package_skill.py to validate and create .skill file
6. **Iterate** - Test on real tasks, improve based on usage
