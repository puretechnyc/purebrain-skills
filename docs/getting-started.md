# Getting Started with PureBrain Skills

## Prerequisites

- [Claude Code](https://claude.ai/code) installed and configured
- Git

## Installation

### Option 1: Clone the full repo

```bash
git clone https://github.com/puretechnyc/purebrain-skills.git
```

### Option 2: Copy individual skills

```bash
# Clone, then copy just the skill you need
git clone https://github.com/puretechnyc/purebrain-skills.git
cp -r purebrain-skills/skills/marketing/cold-email-builder ~/.claude/skills/
```

## Using a Skill

Once a skill is in your `~/.claude/skills/` directory, Claude Code loads it automatically.

Reference it in conversation:

```
/skill cold-email-builder
```

Or let Claude Code detect it based on your task context.

## Skill Structure

Each skill follows a standard format:

```
skill-name/
├── SKILL.md          # Main skill definition (required)
├── README.md         # Usage docs and examples
└── examples/         # Sample inputs/outputs (optional)
```

The `SKILL.md` file contains YAML frontmatter with metadata and the skill instructions that Claude Code reads.

## Categories

| Directory | Focus |
|-----------|-------|
| `skills/marketing/` | Email, ads, SEO, social media automation |
| `skills/operations/` | CRM, workflows, reporting, task management |
| `skills/content/` | Blog posts, newsletters, social carousels |
| `skills/development/` | Deployment, APIs, infrastructure patterns |

## Next Steps

- Browse the [skills directory](../skills/) to find what you need
- Read [CONTRIBUTING.md](../CONTRIBUTING.md) if you want to add your own
- Visit [purebrain.ai](https://purebrain.ai) for the full managed platform
