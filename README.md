# Gasskill

The OpenClaw skill library — reusable skills any bot can install and invoke.

## What is Gasskill?

Gasskill is a collection of self-contained skills for OpenClaw bots. Each skill provides a specific capability (git workflows, testing, monitoring, etc.) that any bot can install and use.

## Skill Structure

Each skill lives in its own directory with:

```
skill-name/
├── SKILL.md      # Frontmatter + documentation
└── scripts/      # Executable scripts
    ├── script1.py
    └── script2.sh
```

### SKILL.md Frontmatter

```yaml
---
name: my-skill
description: What it does in one line
metadata:
  openclaw:
    emoji: "🔧"
    os:
      - linux
    requires:
      bins:
        - git
parameters:
  action:
    type: string
    description: What to do
    required: true
---

# My Skill

Description and usage examples.
```

## Available Skills

| Skill | Description |
|-------|-------------|
| skills/* | Individual skill directories |

See individual skill directories for details.

## Using Skills

Skills are installed to `~/.openclaw/skills/` and invoked via their scripts:

```bash
python3 ~/.openclaw/skills/<skill-name>/scripts/<script>.py [args]
```

## Contributing

1. Create your skill in a new directory
2. Add `SKILL.md` with proper frontmatter
3. Add `scripts/` with executable files
4. **Include ≥2 test cases before submitting PR**
5. Follow conventional commit format

### Test Requirements

All new skills must include at least 2 test cases before merge. Tests can be:
- Unit tests for scripts
- Integration tests
- Manual verification scripts

## Repository Structure

```
gasskill/
├── README.md
├── docs/              # Documentation
├── skills/            # Individual skills
├── openclaw-gateway/  # Gateway configuration
├── tmux-manager/      # TMUX session management
└── tmux-test/         # TMUX testing utilities
```

## Resources

- [OpenClaw Docs](https://docs.openclaw.ai)
- [Gasskill Issues](https://github.com/gastown-publish/gasskill/issues)
- [OpenClaw Discord](https://discord.com/invite/clawd)