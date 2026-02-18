# Testing your skill

Three testing levels, from quickest to most thorough.

## Level 1: Manual testing (fastest)

Open a new session and try your trigger phrases. Verify the skill activates and produces correct output.

### Trigger testing

Test that the skill activates on expected phrases and stays quiet on irrelevant ones.

| Test type | Example | Expected |
|-----------|---------|----------|
| Exact trigger | "/my-skill" | Skill activates |
| Natural language | "help me with X" (where X is in description) | Skill activates |
| Partial match | Related but not exact keyword | Skill may or may not activate |
| Negative | Completely unrelated prompt | Skill does NOT activate |

Write 3-5 trigger test prompts per skill. Include at least one negative test.

### Functional testing

Run the skill's main workflow end-to-end. For each mode:

1. Provide realistic input
2. Check output matches expected format
3. Verify all sections/steps are present
4. Check for hallucinated content (wrong paths, phantom APIs, made-up examples)

## Level 2: Scripted testing (recommended)

Use Claude Code to run a sequence of test prompts and check outputs.

```bash
# Example: test that scan mode produces expected report format
echo "/skeall --scan ~/.claude/skills/my-skill/" | claude --print
# Check output contains: "Score:", "STRUCTURE", "FRONTMATTER", "CONTENT"
```

### What to test

| Area | Test | Pass criteria |
|------|------|---------------|
| Activation | Trigger phrases from description field | Skill loads and responds |
| Core workflow | Main mode end-to-end | Output matches documented format |
| Edge cases | Empty input, missing files, invalid paths | Helpful error message, not crash |
| Cross-platform | Same SKILL.md on different LLM platform | Same general behavior |

## Level 3: Programmatic testing (for distributed skills)

Use the Claude API to run automated test suites.

```python
# Pseudocode for API-based skill testing
test_prompts = [
    {"input": "trigger phrase 1", "expect_contains": "expected output pattern"},
    {"input": "trigger phrase 2", "expect_contains": "expected section"},
    {"input": "unrelated prompt", "expect_not_contains": "skill output"},
]

for test in test_prompts:
    response = claude_api.send(test["input"])
    assert test["expect_contains"] in response
```

## Testing checklist

Use this after creating or improving a skill:

| # | Check | Status |
|---|-------|--------|
| 1 | All trigger phrases from description field activate the skill | |
| 2 | Main workflow produces correct output format | |
| 3 | At least one negative test (unrelated prompt) passes | |
| 4 | Error cases produce helpful messages | |
| 5 | Output contains no hallucinated paths or APIs | |
| 6 | Skill works with the target LLM platform(s) | |

## Common testing pitfalls

| Pitfall | Fix |
|---------|-----|
| Testing only the happy path | Add edge cases: empty input, missing files, wrong format |
| Testing in same session as development | Start fresh session to avoid context bleed |
| Not testing trigger phrases | Users find skills through triggers, not direct invocation |
| Skipping negative tests | A skill that activates on everything is worse than one that activates on nothing |
