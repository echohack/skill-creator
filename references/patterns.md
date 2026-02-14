# Design Patterns for Skills

Detailed pattern examples. SKILL.md links here; read when designing a new skill.

## Progressive Disclosure

Skills use a three-level loading system:

| Level | What loads | Budget |
|-------|-----------|--------|
| 1. Metadata | `name` + `description` | ~100 words, always in context |
| 2. SKILL.md body | Instructions, workflow | <5k words, on trigger |
| 3. Bundled resources | Scripts, references, assets | Unlimited, on demand |

### Pattern: High-level guide with references

```markdown
# PDF Processing

## Quick start
Extract text with pdfplumber: [code example]

## Advanced features
- **Form filling**: See [references/forms.md](references/forms.md)
- **API reference**: See [references/api.md](references/api.md)
```

Claude loads forms.md or api.md only when needed.

### Pattern: Domain-specific organization

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── references/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    └── product.md (API usage, features)
```

User asks about sales metrics — Claude reads only sales.md.

### Pattern: Framework variants

```
cloud-deploy/
├── SKILL.md (workflow + provider selection)
└── references/
    ├── aws.md
    ├── gcp.md
    └── azure.md
```

User picks AWS — Claude reads only aws.md.

### Pattern: Conditional details

```markdown
# DOCX Processing

## Creating documents
Use docx-js for new documents. See [references/docx-js.md](references/docx-js.md).

## Editing documents
For simple edits, modify the XML directly.
- **Tracked changes**: See [references/redlining.md](references/redlining.md)
- **OOXML details**: See [references/ooxml.md](references/ooxml.md)
```

## Degrees of Freedom

Match specificity to the task's fragility and variability.

### High freedom — text instructions

Use when multiple approaches are valid and decisions depend on context.

```markdown
## Image optimization
Choose a compression strategy based on the image type and target
file size. Prefer lossless for icons; lossy for photos.
```

### Medium freedom — pseudocode or parameterized scripts

Use when a preferred pattern exists but some variation is acceptable.

```markdown
## Database migration
1. Generate migration: `npx prisma migrate dev --name <description>`
2. Review the generated SQL in `prisma/migrations/`
3. Apply: `npx prisma migrate deploy`

Adjust the schema path if non-standard.
```

### Low freedom — exact scripts, few parameters

Use when operations are fragile, consistency is critical, or a specific sequence must be followed.

```markdown
## PDF rotation
Run the rotation script. Do not rewrite — edge cases are handled.
\```bash
python scripts/rotate_pdf.py <input.pdf> <degrees> <output.pdf>
\```
```

**Rule of thumb:** Narrow bridge with cliffs = low freedom (specific guardrails). Open field = high freedom (many valid routes).

## Acceptance by Example

Define success through concrete examples with acceptance criteria instead of verbose explanations.

### Structure

1. **Scenario** — A realistic input or user request
2. **Expected output** — What the skill should produce
3. **Acceptance criteria** — Specific, testable conditions

### Example: Changelog skill

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

### Writing guidelines

- Focus on outcomes, not process
- Keep criteria testable — clear pass/fail, no "should look nice"
- Make criteria independent of each other
- Two or three examples covering distinct scenarios beat ten similar ones
- Place acceptance examples in `references/` when there are more than three

## Case Arrows

Use case arrow (`->`) format to declare edge cases and failure modes alongside the expected behavior. Borrowed from Elixir's `case` pattern matching: edge cases first, expected behavior last.

### When to use arrows vs bullets

| Format | Use for |
|--------|---------|
| Bulleted criteria | Defining what correct output looks like |
| Arrow clauses | Declaring branches: what happens for each input type |

The two work together: arrows declare the branches, bullets define quality within each branch.

### Example: PDF editor

```
case input:
  no file provided          -> inform user, no action taken
  file is not a PDF         -> return error identifying the file type
  PDF is password-protected -> inform user, suggest removing protection
  PDF has zero pages        -> inform user the document is empty
  corrupt or unreadable PDF -> return error with filename, do not crash
  single-page PDF           -> apply operation, output valid PDF
  multi-page PDF            -> apply to correct pages, preserve others
```

### Example: Changelog (complements the acceptance example above)

```
case changelog input:
  no changes provided     -> output nothing, inform user
  missing version number  -> infer next patch from previous entry
  missing issue/PR number -> omit reference, do not fabricate
  change type ambiguous   -> ask user to clarify before categorizing
  valid bug fix           -> generate entry per acceptance criteria
```

**Key principle:** Edge cases first, expected behavior last. New cases slot in naturally — the format gives them a place to go.

## Skill Stacking

Each skill owns a specific domain. When a domain outgrows a single skill, split into focused skills that compose together — like functions calling functions.

### Signals to split

- SKILL.md approaching the 500-line limit
- Skill accumulates unrelated functions (PDF rotation + spreadsheet parsing)
- Two users would describe the skill's purpose differently
- Bundled resources serve distinct, independent tasks

### Composition pattern

```
# Leaf skills — own a specific domain
pdf-editor/
├── SKILL.md          # PDF transforms: rotate, merge, split, extract
└── scripts/
    └── rotate_pdf.py

docx-editor/
├── SKILL.md          # DOCX transforms: create, edit, track changes
└── references/
    └── ooxml.md

# Orchestrator skill — composes leaf skills for cross-domain tasks
document-converter/
├── SKILL.md          # Converts between formats
└── references/
    └── format-matrix.md
```

The orchestrator's SKILL.md references the leaf skills:

```markdown
## Behavior

Convert documents between formats.

- **PDF -> DOCX**: Extract text with `pdf-editor`, create document with `docx-editor`
- **DOCX -> PDF**: Export with `docx-editor`, post-process with `pdf-editor`

For PDF-specific operations, defer to the `pdf-editor` skill.
For DOCX-specific operations, defer to the `docx-editor` skill.
```

### Ownership rules

- **One domain, one skill.** Avoid two skills that handle the same file type or API.
- **Leaf skills are self-contained.** They must work independently, not only through an orchestrator.
- **Orchestrators are thin.** They declare transformations and delegate — minimal domain logic of their own.
- **Reference, don't duplicate.** If a leaf skill already handles an edge case, the orchestrator defers to it rather than reimplementing the logic.
