---
name: skill-creator
description: Guide for creating effective skills. This skill should be used when users want to create a new skill (or update an existing skill) that extends Claude's capabilities with specialized knowledge, workflows, or tool integrations.
---

# Skill Creator

Create and iterate on skills — modular packages that extend Claude with specialized knowledge, workflows, and tools.

## Skill Structure

```
skill-name/
├── SKILL.md              # Required. Frontmatter + instructions.
├── scripts/              # Executable code for deterministic/repeated tasks
├── references/           # Documentation loaded into context on demand
└── assets/               # Files used in output (templates, icons, fonts)
```

### Frontmatter

```yaml
---
name: skill-name                # Required. Skill identifier.
description: What this skill does. Use when [triggers]. Covers [use cases].  # Required. Primary trigger mechanism.
allowed-tools:                  # Optional. Tools the skill needs permission to use.
  - Bash(npm run build)
  - WebFetch(docs.example.com/*)
---
```

The `description` is the primary trigger mechanism. Include all "when to use" language here — the body only loads after triggering.

`allowed-tools` grants tool permissions when the skill is active. Each entry is a tool name with an optional argument pattern. Use when the skill requires specific tool access (shell commands, web fetches, MCP tools) to function.

Example for a `docx` skill:
```yaml
description: Document creation, editing, and analysis with tracked changes,
  comments, and formatting. Use when working with .docx files for creating,
  modifying, or reviewing professional documents.
allowed-tools:
  - Bash(python scripts/convert_docx.py *)
```

### Bundled Resources

| Directory | Purpose | Loads into context? |
|-----------|---------|---------------------|
| `scripts/` | Deterministic code, repeatedly needed | No (executed directly) |
| `references/` | Documentation Claude consults while working | Yes, on demand |
| `assets/` | Templates, images, boilerplate for output | No (used in output) |

Test all added scripts by running them. Delete example files and directories not needed by the skill.

## Standard Template

Follow the section order in [references/template.md](references/template.md) when writing SKILL.md:

1. **Title + one-liner** — Immediate orientation
2. **Behavior** — Input -> output transformations
3. **Examples** — Acceptance by example
4. **Error Handling** — Edge cases and failures
5. **Bundled Resources** — What's included and when to use it

Think of skills as functional constructs: declare the transformations (what goes in, what comes out), not procedural steps. For skills with multiple functions, declare each transformation separately.

Omit sections that don't apply. Keep SKILL.md under 500 lines.

## Writing Style

The context window is a public good. Every line in a skill competes with system prompt, conversation history, and the user's actual request.

**Rules:**
- Write in imperative voice ("Extract text with..." not "You can extract text with...")
- Cut "why this works" paragraphs — Claude doesn't need motivation
- Prefer one example over three paragraphs of explanation
- Only add context Claude doesn't already have
- Challenge each paragraph: does it justify its token cost?

### Use Declarative instead of Procedural

**Pattern** (declarative):
```markdown
## Changelog Generation

input: git commit range (defaults to last tag..HEAD)
output: changelog entry prepended to CHANGELOG.md

Format: [Keep a Changelog](https://keepachangelog.com). Group commits by
conventional commit type. Use today's date and the next semantic version.
```

**Anti-pattern** (procedural):
```markdown
## Generating a Changelog

1. First, run `git log` to get the commits since the last tag
2. Then, group the commits by type (feat, fix, chore)
3. Next, format each group as a markdown section
4. Finally, prepend the new entry to CHANGELOG.md with today's date
```

## Design Patterns

Brief summaries below. For full examples, see [references/patterns.md](references/patterns.md).

### Progressive Disclosure

Load context in three levels:

| Level | What | Budget |
|-------|------|--------|
| Metadata | `name` + `description` | ~100 words, always loaded |
| SKILL.md body | Instructions, workflow | <5k words, on trigger |
| Bundled resources | Scripts, references, assets | Unlimited, on demand |

Keep core behavior in SKILL.md. Move variant-specific details to reference files. Reference them clearly so Claude knows they exist and when to read them.

### Degrees of Freedom

Match specificity to fragility:

| Freedom | When | Format |
|---------|------|--------|
| High | Multiple valid approaches | Text instructions |
| Medium | Preferred pattern, some variation | Pseudocode or parameterized scripts |
| Low | Fragile operations, consistency critical | Exact scripts, few parameters |

### Acceptance by Example

Define success through concrete examples with testable acceptance criteria. Structure: **Scenario** (input) -> **Expected output** -> **Acceptance criteria** (pass/fail conditions).

**Example — Changelog skill:**

Scenario: User asks to generate a changelog entry for a bug fix.

Expected output:
```
## [1.2.1] - 2025-03-15
### Fixed
- Resolved timeout error when uploading files >50MB (#423)
```

Acceptance criteria:
- Follows [Keep a Changelog](https://keepachangelog.com) format
- Version uses semantic versioning
- Change categorized correctly (Added, Fixed, Changed, etc.)
- References issue/PR number when available
- Date in ISO 8601 format (YYYY-MM-DD)

### Case Arrows

Declare edge cases and failure modes with arrow (`->`) format. Edge cases first, expected behavior last:

```
case input:
  empty input     -> return empty result, no error
  missing field   -> use sensible default
  malformed input -> return clear error message
  valid input     -> produce expected output
```

Use arrows for branching logic, bullets for output quality within each branch.

### Skill Stacking

Each skill owns a specific domain. When a domain grows too large, split it into focused skills that compose together:

```
# Too broad — one skill doing everything
document-processor/    # handles PDF, DOCX, XLSX, images, OCR...

# Better — stacked skills with clear ownership
pdf-editor/            # PDF-specific transforms
docx-editor/           # DOCX-specific transforms
document-converter/    # converts between formats, invokes pdf-editor and docx-editor
```

Skills reference other skills for cross-domain tasks. A high-level skill orchestrates; leaf skills own their domain. Split when a skill accumulates unrelated functions or its SKILL.md approaches the 500-line limit.

## Process

### 1. Understand

Gather concrete examples of how the skill will be used. Ask:
- What functionality should this skill support?
- What would a user say that should trigger it?
- Can you show example inputs and expected outputs?

Start with the most important questions. Follow up as needed.

### 2. Plan

Analyze each example to identify reusable resources:

| If you find yourself... | Create a... |
|------------------------|-------------|
| Rewriting the same code | `scripts/` file |
| Re-discovering schemas or docs | `references/` file |
| Copying the same boilerplate | `assets/` directory |

### 3. Build

Create the skill directory and SKILL.md. Follow the [standard template](references/template.md) and [writing style](#writing-style) rules.

Write the frontmatter first — `description` is the trigger, `allowed-tools` grants permissions. Then declare the behavior: input -> output transformations, examples, error handling, resource references.

Request user input when the skill needs their assets, documentation, or domain knowledge.

### 4. Iterate

1. Use the skill on real tasks
2. Notice struggles or inefficiencies
3. Update SKILL.md or bundled resources
4. Test again

Use case arrows to build a test plan. Verify edge cases first, expected behavior last.

## Anti-patterns

| Anti-pattern | Why it's bad | Fix |
|--------------|-------------|-----|
| README, CHANGELOG, or other auxiliary docs | Clutter — skills are for agents, not humans | Delete them |
| "When to use" in the body | Body loads after triggering — too late | Move to frontmatter `description` |
| Nested references (ref -> ref -> ref) | Claude loses the thread | Keep references one level deep |
| Duplicated content across SKILL.md and references | Wastes context, risks drift | Single source of truth |
| Dead links to non-existent files | Confuses Claude, wastes a tool call | Verify all referenced files exist |
| Verbose explanations over examples | Higher token cost, lower signal | One example beats three paragraphs |
| Reference files >100 lines without TOC | Claude can't preview scope | Add table of contents at top |
