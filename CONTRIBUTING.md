# Contributing to PureBrain Skills

We welcome skill contributions from the community!

## Skill Format

Every skill must be a valid Claude Code SKILL.md with YAML frontmatter:

```yaml
---
name: your-skill-name
description: One-line description
version: 1.0.0
category: marketing | operations | content | development
author: Your Name
---
```

## Submission Process

1. Fork this repo
2. Create your skill in the appropriate `skills/{category}/` directory
3. Include a README with usage examples
4. Submit a PR with a description of what the skill does and how you've tested it

## Quality Standards

- Skills must be tested and functional
- Include clear documentation
- No hardcoded credentials or API keys
- Follow the SKILL.md format standard
