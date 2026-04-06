---
name: imscc
description: >
  Canvas LMS IMSCC handler: extraction, analysis, and reconstruction of IMS Common Cartridge (.imscc) course files exported from Canvas. Use this skill whenever the user mentions .imscc, Canvas export, course cartridge, Canvas import, or asks to read/analyze/build/rebuild a Canvas course file. Also trigger when the user wants to inspect or replicate a Canvas course structure, even if they don't use the word "imscc". Covers both directions: reading an existing export to understand its contents, and building a new .imscc file from course data.
---

# IMSCC Skill — Canvas Course Cartridge Handler

This skill covers two related workflows for Canvas `.imscc` files:

1. **Extraction** — reading and analyzing an existing Canvas export
2. **Reconstruction** — building a valid `.imscc` file from course content

Read the appropriate reference file before starting work:

- Extraction task → read `references/extraction.md`
- Reconstruction task → read `references/reconstruction.md`
- Both directions involved → read both

---

## What is an IMSCC file

A `.imscc` file is a ZIP archive following the IMS Common Cartridge v1.1 standard, with Canvas-specific extensions layered on top. Renaming it to `.zip` and unzipping it is always step one for any extraction task.

```bash
unzip course_export.imscc -d course_export/
```

---

## Deciding which workflow applies

**Extraction** applies when the user has an existing `.imscc` file and wants to:
- Understand its structure or content
- Pull out assignments, modules, due dates, or other data
- Compare two exports
- Use the content as source material for something else

**Reconstruction** applies when the user wants to:
- Build a new `.imscc` for Canvas import
- Update an existing course and re-export it programmatically
- Replicate a course structure from one term to another

Both apply when the user wants to extract content from one course and build a modified version.

---

## Quick orientation

Canvas IMSCC exports have two parallel systems that must be read together — neither is complete alone:

| File | What it provides |
|------|-----------------|
| `imsmanifest.xml` | IMS CC-compliant org hierarchy; resource declarations |
| `course_settings/module_meta.xml` | Canvas module structure, content types, external URLs |

The recommended read order for extraction is in `references/extraction.md`. The Canvas-specific build requirements (identifier format, ZIP settings, element order) are in `references/reconstruction.md`.

---

## Bundled references

- `references/extraction.md` — Full extraction guide: file structure, resource types, XML schemas, field references, gotchas, recommended read order
- `references/reconstruction.md` — Canvas build requirements: ZIP settings, identifier format, required elements, common silent-failure causes, post-import manual steps

---

## General principles

**Always handle timezones explicitly.** `<due_at>` in IMSCC is always UTC. When extracting, convert to the user's local timezone (check `<time_zone_edited>` in assignment files, or ask). When reconstructing, convert from local time to UTC. Never assume Pacific Time — the skill is used by instructors in any timezone.

**Be precise about identifiers.** Canvas uses `g` + 32-char MD5 hex for identifiers in reconstructed files. Human-readable IDs cause silent import failures. When extracting, treat all identifiers as opaque strings regardless of format.

**Prefer Python for parsing.** When extracting data programmatically, use `xml.etree.ElementTree` or `lxml` with explicit namespace handling. XML namespaces in Canvas exports are non-trivial and must be declared correctly.

**Verify structure before writing.** When reconstructing, validate the ZIP entry order and all required stub files before packaging. `imsmanifest.xml` must be the first entry in the ZIP.

**Surface the post-import manual steps.** Some Canvas settings cannot be encoded in IMSCC (rubric attachment, "Remove points" checkbox). Always flag these to the user at the end of a reconstruction task.
