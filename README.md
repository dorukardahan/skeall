# Skeall

Build, improve, and audit [Agent Skills](https://agentskills.io) for cross-platform LLM coding agents.

Skeall encodes real-world lessons from building and restructuring production skills across Claude Code, OpenAI Codex, Cursor, Gemini CLI, OpenClaw, and 20+ other platforms that support the Agent Skills open standard.

## What it does

- **Create** new skills from scratch with guided interviews and scaffolding
- **Improve** existing skills with specific before/after edit proposals
- **Scan** skills for spec compliance, LLM-friendliness, and cross-platform issues
- **Batch scan** your entire skill collection at once

## Installation

```bash
# Claude Code
git clone https://github.com/dorukardahan/skeall ~/.claude/skills/skeall

# OpenAI Codex
git clone https://github.com/dorukardahan/skeall ~/.agents/skills/skeall

# OpenClaw
git clone https://github.com/dorukardahan/skeall ~/.openclaw/skills/skeall
```

## Usage

```
/skeall --create              # Interview + scaffold new skill
/skeall --improve <path>      # Analyze and improve existing skill
/skeall --scan <path>         # Audit only, severity-tagged report
/skeall --scan-all            # Batch scan all skills
```

## What it checks

27 checklist items across 4 categories:

| Category | Checks | Examples |
|----------|--------|---------|
| Structure | 7 | SKILL.md exists, references/ dir, no junk files |
| Frontmatter | 5 | Description length, trigger phrases, naming |
| Content | 8 | Token budget, redundancy, framing style |
| LLM-friendliness | 8 | Tables vs bullets, emoji, heading style |
| Cross-platform | 4 | No baseDir, relative paths, standard links |

Plus 20 documented anti-patterns with before/after examples.

## Scoring

Base score: 10. Deductions: HIGH (-1.5), MEDIUM (-0.5, capped at -3), LOW (-0.2, capped at -1).

| Score | Rating |
|-------|--------|
| 9-10 | Excellent |
| 7-8 | Good (PASS) |
| 5-6 | Needs work |
| 0-4 | Major rewrite needed |

## Structure

```
skeall/
├── SKILL.md                    # Core skill (always loaded)
├── references/
│   ├── anti-patterns.md        # 20 anti-patterns with before/after
│   ├── template.md             # Copy-paste SKILL.md template
│   └── scoring.md              # Scoring methodology details
└── README.md                   # This file
```

## Built on real experience

Skeall was born from restructuring [0gfoundation/0g-compute-skills](https://github.com/0gfoundation/0g-compute-skills):
- Audited 19 documentation issues (phantom APIs, wrong model names, parameter order bugs)
- Restructured from flat layout to Agent Skills standard (references/ directory)
- Reduced SKILL.md from 373 lines (~4,800 tokens) to 177 lines (~2,500 tokens)
- Achieved 62% token savings through progressive disclosure

Research sources: [agentskills.io spec](https://agentskills.io), [Anthropic skills guide](https://claude.com/blog/complete-guide-to-building-skills-for-claude), [OpenAI Codex docs](https://developers.openai.com/codex/skills/), [OpenClaw docs](https://docs.openclaw.ai/tools/skills).

## License

MIT
