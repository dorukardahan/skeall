---
name: skeall
description: |
  Agent Skills (SKILL.md) builder, auditor, and improver for cross-platform LLM agents.
  Use for "skeall", "build a skill", "create skill", "improve skill", "audit skill",
  "skill review", or any SKILL.md question. Follows agentskills.io standard.
---

# Skeall

Create, improve, and audit Agent Skills following the [Agent Skills open standard](https://agentskills.io). This skill encodes lessons from real-world skill development and cross-platform compatibility testing.

## Quick start

```text
/skeall --create              # Interview, then scaffold new skill
/skeall --improve <path>      # Analyze and improve existing skill
/skeall --scan <path>         # Audit only, no changes (report)
/skeall --scan .              # Audit skill in current directory
/skeall --scan-all            # Batch scan all skills in ~/.claude/skills/
/skeall --scan-all <dir>      # Batch scan all skills in custom directory
```

---

## Mode 1: Create (scaffold a new skill)

### Process

1. Interview the user (ask questions 1-3 first, then 4-5 if needed):
   - What does this skill do? (one sentence)
   - What triggers should activate it? (keywords users would type)
   - How complex is it? (single file vs references/ needed)
   - Target audience: which LLM platforms?
   - Any existing docs/code to incorporate?

2. Generate the skill structure:

```text
{skill-name}/
├── SKILL.md                    # Core instructions (always loaded)
├── references/                 # On-demand detail files
│   ├── {topic-1}.md
│   └── {topic-2}.md
└── README.md                   # GitHub-facing (optional)
```

3. Write SKILL.md following these rules:
   - YAML frontmatter with `name` and `description` (see Frontmatter section)
   - Body under 500 lines, under 5000 tokens
   - Instruction-based framing, not persona-based
   - Progressive disclosure: core in SKILL.md, details in references/

4. Show the generated SKILL.md to user for review.

5. Run `--scan` on the generated skill. If any HIGH issues found, fix them before delivering.

**Next step:** "Optimize with reprompter?" (optional, see Reprompter section). Then suggest installing the skill.

---

## Mode 2: Improve (refactor existing skill)

### Process

1. Read SKILL.md first. Read reference files only if scan identifies issues requiring them (broken links, routing table mismatches).
2. Run the scan checklist (see Mode 3).
3. For each issue found, propose a specific before/after edit.
4. Group edits by priority: HIGH first, then MEDIUM, then LOW.
5. Ask user: "Fix all? Review one by one? Or just the HIGHs?" (recommended: fix all HIGHs automatically, review MEDIUMs)
6. Apply approved edits.
7. Re-scan once. If new issues appear, report them but do not enter an infinite fix loop.

**Next step:** "Run `--scan` to verify?" or "Commit changes?"

### Common improvements

| Problem | Fix |
|---------|-----|
| Body over 5000 tokens | Move detail sections to references/ |
| Redundant content | Single source of truth, reference elsewhere |
| Persona-based framing | Switch to instruction-based framing |
| Missing trigger phrases | Add keywords to description field |
| Platform-specific patterns | Replace with universal formatting |
| No progressive disclosure | Add routing table to reference files |

---

## Mode 3: Scan (audit and report)

### Process

1. Read the skill's SKILL.md and directory structure.
2. Check every item in the checklist below.
3. Output a severity-tagged report.

### Report format

```text
## Skill Audit: {skill-name}

Score: X/10

STRUCTURE
  [PASS] S1 -- SKILL.md exists at root
  [FAIL] S3 HIGH -- name does not match directory name
  [WARN] S5 MEDIUM -- No references/ directory

FRONTMATTER
  [PASS] F2 -- Trigger phrases present
  [FAIL] F1 HIGH -- description over 1024 characters

CONTENT
  [WARN] C5 MEDIUM -- Persona-based framing ("You are an expert")
  [FAIL] C3 HIGH -- Same content repeated 3 times (lines 45, 120, 280)

LLM-FRIENDLINESS
  [WARN] L4 MEDIUM -- Unicode arrows instead of markdown tables
  [PASS] L3 -- No emoji markers in headings

CROSS-PLATFORM
  [PASS] X1 -- No {baseDir} placeholders
  [WARN] X4 LOW -- No multi-platform install instructions in README

Total: 3 HIGH | 4 MEDIUM | 1 LOW
```

**Next step after scan:** "Want me to fix these? Run `/skeall --improve <path>`"

### Error handling

| Input | Response |
|-------|----------|
| No SKILL.md found at path | "No skill found at {path}. Did you mean `--create`?" |
| Empty directory for `--scan-all` | "No skills found in {dir}. Skills must have a SKILL.md file." |
| Invalid YAML frontmatter | Report the parse error, suggest fixing frontmatter first |
| `--improve` on non-skill file | "Not a valid skill (no YAML frontmatter). Try `--create` instead." |

---

## Agent Skills spec reference

### Frontmatter (required)

```yaml
---
name: my-skill-name
description: What this skill does and when to use it. Include trigger phrases.
---
```

**name rules:**
- Must match the parent directory name
- Lowercase alphanumeric with hyphens only (unicode lowercase allowed)
- 1-64 characters, no leading/trailing/consecutive hyphens
- No spaces, no special characters

**description rules:**
- Explain WHAT it does AND WHEN to use it
- Include trigger phrases users would actually type
- Put the most important keyword first (platforms weight first words)
- Spec limit: 1024 characters. Recommended: under 300 for best matching
- Use noun-phrase style ("Guide for X"), not persona style ("Expert in X")

### Optional frontmatter fields

These are silently ignored by platforms that do not support them:

```yaml
license: MIT                   # For distributed skills
compatibility: "Node.js 18+"  # Environment requirements (max 500 chars)
metadata:                      # Arbitrary key-value (author, version)
  author: your-name
  version: 1.0.0
allowed-tools: "Bash Read"    # Experimental: space-delimited tool list
```

### Directory structure

```text
skill-name/
├── SKILL.md           # REQUIRED -- core instructions
├── references/        # OPTIONAL -- on-demand detail files
├── scripts/           # OPTIONAL -- executable scripts
├── assets/            # OPTIONAL -- static assets (images, etc.)
└── README.md          # OPTIONAL -- GitHub-facing docs
```

### Token budget

| Level | Content | Budget |
|-------|---------|--------|
| Metadata (YAML frontmatter) | name + description | ~100 tokens |
| Instructions (SKILL.md body) | Always loaded by LLM | < 5000 tokens |
| References (each file) | Loaded on demand | ~2000-3000 tokens each |

**Estimation:** ~1.5 tokens per word for mixed code+prose markdown.

**Progressive disclosure:** SKILL.md body should handle ~70% of user requests. Reference files handle the remaining 30% (detailed workflows, complete examples, edge cases).

### Line limits

| Guideline | Limit |
|-----------|-------|
| SKILL.md body | Under 500 lines (under 300 for complex skills with many references) |
| Reference files | No hard limit, but keep each under 700 lines |

---

## Scan checklist

### Structure checks

| ID | Severity | Check |
|----|----------|-------|
| S1 | HIGH | SKILL.md exists at skill root |
| S2 | HIGH | YAML frontmatter present with `---` delimiters |
| S3 | HIGH | `name` field present and valid (lowercase, hyphens, 1-64 chars, no consecutive hyphens) |
| S4 | HIGH | `description` field present |
| S5 | MEDIUM | References in `references/` not loose at root |
| S6 | LOW | README.md present for GitHub-hosted skills |
| S7 | LOW | No unnecessary files (node_modules, .DS_Store, etc.) |
| S8 | HIGH | `name` field matches parent directory name |

### Frontmatter checks

| ID | Severity | Check |
|----|----------|-------|
| F1 | HIGH | Description under 1024 characters (spec limit) |
| F1b | LOW | Description under 300 characters (recommended for matching) |
| F2 | HIGH | Description includes trigger phrases |
| F3 | MEDIUM | Description starts with noun phrase, not "Expert in" |
| F4 | MEDIUM | Name 1-64 characters, no leading/trailing/consecutive hyphens |
| F5 | LOW | No platform-specific fields (keeps universal compatibility) |

### Content checks

| ID | Severity | Check |
|----|----------|-------|
| C1 | HIGH | Body under 500 lines |
| C2 | HIGH | Estimated tokens under 5000 |
| C3 | HIGH | No content repeated in SKILL.md body (controlled repetition across reference files is acceptable) |
| C4 | HIGH | Code examples use correct, verified patterns |
| C5 | MEDIUM | Instruction-based framing (not "You are an expert") |
| C6 | MEDIUM | Has routing table to reference files (if references/ exists) |
| C7 | MEDIUM | Troubleshooting section present (for technical skills) |
| C8 | LOW | No deprecated content at the top (wastes prime token space) |

### LLM-friendliness checks

| ID | Severity | Check |
|----|----------|-------|
| L1 | HIGH | Tables for structured data (not bullet lists with arrows) |
| L2 | HIGH | Imperative instructions ("Do X", not "You should consider X") |
| L3 | MEDIUM | No emoji in headings or structural markers |
| L4 | MEDIUM | No Unicode arrows or special characters for data flow |
| L5 | MEDIUM | Consistent heading hierarchy (no skipped levels) |
| L6 | MEDIUM | Code blocks have language tags |
| L7 | LOW | Sentence case headings (not Title Case) |
| L8 | LOW | No nested blockquotes (some LLMs parse poorly) |

### Cross-platform checks

| ID | Severity | Check |
|----|----------|-------|
| X1 | HIGH | No `{baseDir}` placeholders (breaks non-OpenClaw platforms) |
| X2 | MEDIUM | Relative paths from SKILL.md to references/ |
| X3 | MEDIUM | Internal links use standard markdown `[text](path)` |
| X4 | LOW | README has multi-platform install paths |

---

## LLM-friendliness patterns

These patterns come from real cross-platform testing. Apply them when creating or improving skills.

### Do

- **Tables over prose** for structured data (parameters, options, comparisons)
- **Single source of truth** for any concept explained more than once
- **Instruction-based framing**: "This skill provides instructions for X. Follow these patterns exactly."
- **Imperative verbs**: "Call X after Y", "Use Z for W"
- **Compact routing table** at the top pointing to reference files
- **Parameter comments inline** in code blocks: `providerAddress, // 1st: wallet address`

### Do not

- **Persona-based framing**: "You are an expert in..." (Claude-leaning, other LLMs respond better to instructions)
- **Emoji markers** in headings or structural elements (token-expensive, parsed inconsistently)
- **Unicode arrows** (→, ←) for data flow — use tables or plain prose
- **Blockquote warnings at top** of SKILL.md (wastes prime token space, primes distrust)
- **"When Users Ask" checklists** with 10+ items (bury critical rules, use tables instead)
- **Synonym cycling** for the same concept (confuses LLMs about whether it's the same thing)
- **Repeated content** (wastes tokens, risks contradictions if copies drift)

### Description field optimization

Good description pattern:
```text
{Product/Tool name} guide for {primary use case}. Covers {feature list}.
Use this skill for {trigger phrases separated by commas}.
```

Example:
```yaml
description: 0G Compute Network guide for decentralized AI inference and fine-tuning.
  Covers chatbots, image generation, speech-to-text, SDK integration, CLI commands.
  Use this skill for any 0G compute, 0G AI, or decentralized GPU question.
```

---

## Cross-platform compatibility

### Universal format (works everywhere)

Only `name` and `description` in frontmatter. Standard markdown body. Relative paths. No platform-specific syntax.

### Platform discovery paths

| Platform | User-wide | Project |
|----------|-----------|---------|
| Claude Code | `~/.claude/skills/{name}/` | `.claude/skills/{name}/` |
| OpenAI Codex | `~/.agents/skills/{name}/` | `.agents/skills/{name}/` |
| OpenClaw | `~/.openclaw/skills/{name}/` | `.openclaw/skills/{name}/` |
| Cursor | Standard SKILL.md discovery | Project skills dir |
| Gemini CLI | Standard SKILL.md discovery | Project skills dir |

### Safe optional additions

These fields are silently ignored by platforms that don't support them:

```yaml
user-invocable: true          # OpenClaw: enables manual invocation
```

### Things that break cross-platform

| Pattern | Problem | Fix |
|---------|---------|-----|
| `{baseDir}` placeholder | Only OpenClaw resolves it | Use relative paths |
| Platform-specific instructions | Confuse other LLMs | Keep instructions generic |
| Hardcoded paths | Break on other OS/platforms | Use relative from SKILL.md |

---

## Token estimation

Quick method to estimate tokens for a markdown file:

```bash
# Word count × 1.5 = approximate tokens
wc -w SKILL.md
# Example: 1,200 words × 1.5 = ~1,800 tokens
```

For code-heavy files, use 1.7x multiplier (code has more tokens per word).

### Budget allocation guide

| Skill complexity | SKILL.md target | References needed? |
|-----------------|-----------------|-------------------|
| Simple (one topic, few commands) | 100-200 lines / ~1500 tokens | No |
| Medium (multiple features, some code) | 200-350 lines / ~3000 tokens | 1-2 files |
| Complex (multi-domain, many patterns) | 300-450 lines / ~4500 tokens | 3-5 files |

---

## Severity reference

| Severity | Meaning | Action |
|----------|---------|--------|
| HIGH | Breaks spec compliance or causes LLM confusion | Must fix |
| MEDIUM | Reduces quality or cross-platform compatibility | Should fix |
| LOW | Minor improvement opportunity | Fix if time permits |

---

## Mode 4: Batch scan (scan-all)

Scan every skill in a directory at once. Useful for auditing your entire skill collection.

### Process

1. List all subdirectories containing SKILL.md in the target path (default: `~/.claude/skills/`).
2. Run Mode 3 (scan) on each skill. Output each skill's score as you complete it.
3. Output a summary table sorted by score ascending (worst first).

### Report format

```text
## Batch Skill Audit

| Skill | Score | HIGH | MEDIUM | LOW | Status |
|-------|-------|------|--------|-----|--------|
| seo-optimizer | 5/10 | 3 | 2 | 1 | NEEDS WORK |
| reprompter | 6/10 | 2 | 3 | 0 | NEEDS WORK |
| blogger | 7/10 | 1 | 1 | 2 | NEEDS WORK |
| humanizer-enhanced | 8/10 | 0 | 2 | 1 | PASS |

Total: 4 skills scanned
PASS: 1 | NEEDS WORK: 3

Top issues across all skills:
1. [HIGH] C2 reprompter: Body exceeds 5000 tokens (est. 8,200)
2. [HIGH] C3 seo-optimizer: Content repeated 4 times
3. [HIGH] C5 reprompter: Persona-based framing
```

**PASS threshold:** Score 7+ with zero HIGH issues.

**Next step:** "Start with the lowest-scoring skill. Run `/skeall --improve <path>` on it."

---

## Scoring methodology

**Formula:** `Score = max(0, 10 - (HIGHs x 1.5) - min(MEDIUMs x 0.5, 3) - min(LOWs x 0.2, 1))`

**PASS threshold:** Score 7+ AND zero HIGH issues. For detailed examples, see [references/scoring.md](references/scoring.md).

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Token estimate seems wrong | Use `wc -w` and multiply by 1.5 (prose) or 1.7 (code-heavy) |
| Scan reports FAIL but skill works fine | HIGHs indicate spec/LLM issues, not runtime bugs. Fix them anyway. |
| Batch scan misses a skill | Skill directory must contain SKILL.md at root |
| Two fixes contradict each other | Flag the conflict, ask user to choose (e.g., "shorten file" vs "add section") |
| Score 7+ but still NEEDS WORK | Check for HIGH issues. Any HIGH = NEEDS WORK regardless of score |

---

## Testing your skill

After creating or improving a skill, test three things:

1. **Trigger testing:** Does the skill activate on expected phrases? Try 3-5 variations of trigger keywords.
2. **Functional testing:** Does the skill produce correct output? Run the main workflow end-to-end.
3. **Negative testing:** Does the skill stay quiet on irrelevant queries? Try unrelated prompts.

For detailed testing patterns and example test prompts, see [references/testing.md](references/testing.md).

---

## Reprompter integration (optional)

When creating a skill (`--create` mode), optionally use reprompter to optimize:

1. **Description field**: After the interview, generate 3 description variants using reprompter's quality scoring. Pick the highest-scoring one.
2. **Core rules**: Draft the rules, then score them for clarity and specificity. Target 8+/10.
3. **Code examples**: If the skill has code patterns, reprompter validates they are specific and imperative.

This is optional. If reprompter skill is not installed, skeall works standalone.

To enable: after the create interview, say "reprompter optimize" or answer "yes" when skeall asks "Optimize with reprompter?"

---

## References

For detailed checklists and examples, see:
- [Anti-patterns with before/after examples](references/anti-patterns.md)
- [Complete SKILL.md template](references/template.md)
- [Scoring methodology details](references/scoring.md)
- [Testing patterns and examples](references/testing.md)
