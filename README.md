# Canvas IMSCC Skill for Claude

A Claude skill for reading, analyzing, and building Canvas LMS course files (`.imscc` format). Works with any Canvas course export, regardless of institution or discipline.

## What it does

This skill gives Claude deep knowledge of the IMS Common Cartridge v1.1 format as Canvas implements it, covering two workflows:

**Extraction** — Read an existing `.imscc` export and pull out:
- Module structure (all modules, items, content types, positions)
- Assignment metadata (due dates in any timezone, grading type, submission types, points)
- External URLs, file attachments, Canvas Pages, LTI links
- Course settings, assignment groups, rubric definitions

**Reconstruction** — Build a new `.imscc` file for Canvas import, with correct:
- ZIP structure and compression (Canvas has strict requirements that cause silent failures if wrong)
- Identifier format (`g` + 32-char MD5 hex — human-readable IDs are rejected)
- File ordering, required stub files, namespace declarations
- UTC date encoding with timezone-aware guidance
- Post-import steps the user needs to complete manually in Canvas

## Why this skill exists

Canvas IMSCC files have a number of non-obvious requirements that cause silent "unexpected error" import failures if violated — things like ZIP compression type, entry order, specific XML root element names, and a dual-system structure (`imsmanifest.xml` + `module_meta.xml`) where neither file alone is complete. This skill encodes the knowledge needed to avoid those failure modes without having to rediscover them each time.

## Requirements

- Claude (any interface that supports skills: Claude Code, Cowork, or compatible Claude Desktop configurations)
- Python 3 available in the execution environment (for extraction scripts and IMSCC build scripts)

## Installation

### Claude Code

```bash
claude skills install canvas-imscc.skill
```

### Cowork / Claude Desktop

Open Settings → Skills → Install from file, and select `canvas-imscc.skill`.

## Usage

Once installed, Claude will automatically use this skill when you reference `.imscc` files, Canvas exports, or course cartridges. You can also trigger it directly:

**Extracting from a Canvas export:**
> "I have a Canvas export at ~/Downloads/my_course.imscc. Can you pull out all the assignments with their due dates and grading types?"

**Reading module structure:**
> "What's the full module structure in this imscc file? Show me what's in each week and what type each item is."

**Building a new course file:**
> "I need to build an IMSCC file for Canvas import. The course is called 'BIOL 101'. It has two modules: Week 1 with a slide link and an assignment due January 15 at 11:59 PM EST, and Week 2 with an assignment and a CSV file attachment."

## What's inside

```
canvas-imscc/
├── SKILL.md                    ← Skill instructions and workflow guide
└── references/
    ├── extraction.md           ← Full extraction reference: file structure, XML schemas,
    │                               resource types, field references, gotchas, read order
    └── reconstruction.md       ← Build reference: Canvas ZIP requirements, identifier format,
                                    required stub files, UTC conversion, post-import steps
```

## Scope and limitations

- **Canvas-specific.** The skill targets Canvas LMS's implementation of IMS Common Cartridge v1.1, including Canvas extensions. Other LMS platforms (Blackboard, Moodle, D2L) export valid IMS CC files but use different conventions — the base structure applies, but Canvas-specific guidance (module_meta.xml, identifier format, ZIP requirements) does not.

- **Timezone-aware.** Due dates in IMSCC are always stored as UTC. The skill includes a timezone lookup table and instructs Claude to ask for or infer the correct local timezone rather than assuming any particular one.

- **Post-import manual steps.** Some Canvas settings cannot be encoded in an IMSCC file — rubric attachment to assignments and the "Remove points from rubric" checkbox must be set manually after import. The skill always surfaces these to the user.

## Background

This skill was developed from a practical project at the University of Washington — building and analyzing course cartridges for HCDE 530 (Computational Concepts in HCDE). The knowledge it encodes came from systematic comparison of known-good Canvas exports and repeated import testing to identify exactly which requirements cause silent failures.

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Improvements welcome. If you encounter a Canvas import failure not covered by the current skill, or find that the skill's guidance leads to incorrect output, please open an issue with:
- The Canvas version you're importing into (if known)
- The element or file that caused the problem
- What the correct behavior should be

Pull requests that extend coverage to additional Canvas features (quizzes, LTI tools, peer review, etc.) are particularly welcome.
