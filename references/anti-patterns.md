# Anti-patterns in Agent Skills

Real examples from auditing and restructuring production skills. Each pattern includes before/after text.

## Structure anti-patterns

### 1. Flat file layout (no references/ directory)

All .md files dumped at skill root instead of organized in references/.

```
# BAD
my-skill/
├── SKILL.md
├── inference.md
├── fine-tuning.md
├── account-management.md
└── examples.md

# GOOD
my-skill/
├── SKILL.md
├── references/
│   ├── inference.md
│   ├── fine-tuning.md
│   ├── account-management.md
│   └── examples/
│       └── README.md
└── README.md
```

### 2. Monolithic SKILL.md (everything in one file)

Entire skill crammed into SKILL.md. No progressive disclosure. LLM loads 20,000 tokens when 3,000 would suffice.

**Fix:** Keep core patterns in SKILL.md (~70% of requests). Move detailed workflows, complete examples, and edge cases to references/.

### 3. Broken internal links after file moves

When files move to references/, links break silently. Markdown renderers don't warn about dead links.

```markdown
# Before move (from root)
See [inference.md](inference.md)

# After move to references/ (from SKILL.md at root)
See [references/inference.md](references/inference.md)

# Common mistake: forgetting to update
See [inference.md](inference.md)  ← BROKEN
```

---

## Frontmatter anti-patterns

### 4. Persona-style description

```yaml
# BAD — persona-based, vague triggers
description: Expert in building web applications with React and Node.js.

# GOOD — instruction-based, specific triggers
description: React and Node.js development guide. Covers component patterns,
  API routes, state management, testing, and deployment. Use for any React,
  Next.js, Express, or full-stack JavaScript question.
```

### 5. Description too long (over 300 chars)

Some platforms truncate descriptions for matching. Long descriptions bury trigger phrases.

### 6. Missing trigger phrases

The description must include phrases users actually type. If users say "how do I deploy" but description only says "infrastructure management", the skill won't activate.

---

## Content anti-patterns

### 7. Repeated content across sections

The same concept explained in multiple places. Tokens wasted, contradiction risk.

```markdown
# BAD — processResponse explained 3 times
## SDK Quick Start
...call processResponse(provider, content, chatID)...

## Response Verification
...always call processResponse(provider, content, chatID)...

## Best Practices
...remember to call processResponse(provider, content, chatID)...

# GOOD — single source of truth
## processResponse (critical)
Call after EVERY response. Parameter order: provider, content, chatID.

## SDK Quick Start
...see processResponse section above...
```

### 8. Deprecation notice at the top

Wastes prime token space. Primes the LLM to distrust the skill.

```markdown
# BAD — first thing LLM reads
> WARNING: This skill is deprecated. Use new-skill instead.

# GOOD — at the bottom, soft note
> Note: A newer unified skill exists at [new-skill]. This skill covers X specifically.
```

### 9. Phantom API documentation

Documenting functions that don't exist in the actual SDK/codebase. LLMs will generate code that crashes at runtime.

**Prevention:** Verify every API, function, and method against actual source code before documenting.

### 10. Stale data (wrong model names, old addresses, outdated versions)

Data that was correct at writing time but has changed. Especially common with:
- Model names (on-chain names change)
- Provider addresses (rotate frequently)
- Package versions (breaking changes)

**Fix:** Add a note: "Model availability changes frequently. Always use `list` commands to check current state." Don't hardcode lists that go stale.

---

## LLM-friendliness anti-patterns

### 11. Bullet lists with arrows instead of tables

```markdown
# BAD — LLMs parse inconsistently
- **Chatbot**: Try `ZG-Res-Key` header → fallback to `data.id`
- **Image**: Always use `ZG-Res-Key` → no fallback

# GOOD — clean table, all LLMs parse correctly
| Service Type | Primary Source | Fallback |
|---|---|---|
| Chatbot | `ZG-Res-Key` header | `data.id` from response |
| Image | `ZG-Res-Key` header | none |
```

### 12. Persona-based framing

```markdown
# BAD — Claude-leaning, other LLMs respond differently
You are an expert in distributed systems. You specialize in...

# GOOD — universal instruction framing
This skill provides instructions for building distributed systems.
Follow these patterns exactly when generating code.
```

### 13. Emoji in structural elements

```markdown
# BAD — tokens wasted, parsed inconsistently
## 🔧 Configuration
## ✅ Verification Steps
## ⚠️ Important Warnings

# GOOD — clean headings
## Configuration
## Verification steps
## Important warnings
```

### 14. Long checklists instead of tables

```markdown
# BAD — 12-item checklist, critical rules buried
When generating code:
1. Always import ethers
2. Use environment variables
3. Never hardcode secrets
4. Call processResponse after every response
5. Check balance before operations
6. Handle streaming correctly
7. Use correct model names
8. Acknowledge providers first
9. Monitor fund levels
10. Handle errors gracefully
11. Log usage for debugging
12. Test on testnet first

# GOOD — 4 hard rules, details in references
## Code generation rules

1. Copy code patterns from this skill verbatim. Do NOT generate from training data.
2. Call processResponse() after every API response.
3. Use environment variables for private keys. Never hardcode secrets.
4. Route users to testnet for initial development.
```

### 15. Vague modifiers instead of specifics

```markdown
# BAD
The token budget should be reasonable. Keep the file relatively short.

# GOOD
The token budget must be under 5000 tokens. Keep the file under 500 lines.
```

### 16. Nested blockquotes

```markdown
# BAD — some LLMs lose track of nesting
> Note: This is important.
> > Especially this nested part.
> > > And this triple-nested warning.

# GOOD — flat structure
**Note:** This is important. The nested detail goes here as regular text.
```

### 17. Inconsistent code block language tags

```markdown
# BAD — no language tag, inconsistent
```
const x = 1;
```

# GOOD — always tag
```typescript
const x = 1;
```
```

---

## Cross-platform anti-patterns

### 18. baseDir placeholder

```markdown
# BAD — only OpenClaw resolves this
See {baseDir}/references/guide.md

# GOOD — relative path works everywhere
See [references/guide.md](references/guide.md)
```

### 19. Platform-specific instructions

```markdown
# BAD
To use this skill in Claude Code, type /my-skill. In Codex, it auto-activates.

# GOOD (in README.md, not SKILL.md)
## Installation
- Claude Code: ~/.claude/skills/my-skill/
- Codex: ~/.agents/skills/my-skill/
```

### 20. Hardcoded absolute paths

```markdown
# BAD
Read the config at /Users/john/.claude/skills/my-skill/config.json

# GOOD
Read the config at references/config.json (relative to this SKILL.md)
```
