# Standard Skill Template

Copy this template when creating a new skill. Fill each section; delete any that don't apply.

```markdown
---
name: skill-name
description: One-sentence summary of what this skill does. Use when [specific triggers]. Covers [specific use cases].
allowed-tools:              # Optional. Tools the skill needs.
  - Bash(npm run build)
---

# Skill Name

One-liner restating the skill's purpose.

## Behavior

Declare the skill's input -> output transformations. Describe what the skill
does, not how it does it step-by-step.

\```
input: [what the user provides]
output: [what the skill produces]
\```

For skills with multiple functions, declare each as a separate transformation:

- **function_a**: input -> output description
- **function_b**: input -> output description

## Examples

### Expected behavior

Scenario: User asks to [typical request].

Expected output:
\```
[concrete output]
\```

Acceptance criteria:
- Criterion A (measurable)
- Criterion B (testable)

### Edge cases

\```
case input:
  empty input         -> return empty result, no error
  missing field       -> use sensible default
  malformed input     -> return clear error message
  valid input         -> produce expected output
\```

## Error Handling

| Condition | Behavior |
|-----------|----------|
| Missing input | Inform user, no action |
| Invalid format | Clear error with details |
| Partial failure | Complete what's possible, report remainder |

## Bundled Resources

- `scripts/example.py` — What it does and when to run it
- `references/schema.md` — When to consult this file
- `assets/template/` — How to use these files

For [advanced topic], see [references/advanced.md](references/advanced.md).
```

## Section Order

Follow this order. Omit sections that don't apply.

| # | Section | Purpose |
|---|---------|---------|
| 1 | Title + one-liner | Immediate orientation |
| 2 | Behavior | Input -> output transformations |
| 3 | Examples | Acceptance by example |
| 4 | Error Handling | Edge cases and failures |
| 5 | Bundled Resources | What's included and when to use it |

## Guidelines

- Keep SKILL.md under 500 lines. Move detail to `references/`.
- Put all trigger language in frontmatter `description`, not the body.
- Use imperative voice throughout ("Extract text with..." not "You can extract text with...").
- One example is better than three paragraphs of explanation.
- Every referenced file must exist. No dead links.
