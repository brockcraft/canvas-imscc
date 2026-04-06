# IMSCC Extraction Reference — Canvas LMS

Use this document when reading or analyzing an existing Canvas `.imscc` export. Start here before examining any specific file.

---

## Step 1: Unzip

```bash
unzip course_export.imscc -d course_export/
```

The `.imscc` extension is cosmetic — the file is a standard ZIP archive.

---

## Top-Level Directory Structure

```
{course-export}/
├── imsmanifest.xml                     ← Master index: org hierarchy + resource declarations
├── course_settings/                    ← Canvas-specific config (all required for import)
│   ├── canvas_export.txt               ← Marker file (presence = Canvas-native format)
│   ├── course_settings.xml             ← Course metadata (title, dates, home page, etc.)
│   ├── module_meta.xml                 ← KEY: Module structure with items and content types
│   ├── assignment_groups.xml           ← Gradebook assignment groups
│   ├── rubrics.xml                     ← Rubric definitions
│   ├── late_policy.xml                 ← Late submission policy
│   ├── grading_standards.xml           ← Letter grade thresholds (if used)
│   ├── files_meta.xml                  ← Metadata for uploaded file resources
│   ├── events.xml                      ← Calendar events
│   ├── context.xml                     ← External tool configs
│   ├── media_tracks.xml                ← Media caption tracks
│   └── syllabus.html                   ← Canvas Syllabus page body (raw HTML, no wrapper)
├── {hex-id}/                           ← One folder per assignment
│   ├── assignment_settings.xml         ← Assignment metadata: grading, due dates, types
│   └── {slug}.html                     ← Student-facing assignment body HTML
├── {hex-id}.xml                        ← Root-level weblink files (imswl format, one per external URL)
├── wiki_content/                       ← Canvas Pages (HTML with Canvas meta tags)
│   └── {slug}.html
├── web_resources/                      ← Uploaded files, organized in subfolders
│   └── {subfolder}/{filename}
├── lti_resource_links/                 ← LTI tool link definitions
│   └── {hex-id}.xml
└── non_cc_assessments/                 ← Canvas-native quiz files (QTI format)
    └── {hex-id}.xml.qti
```

Not every course will have all directories. `wiki_content/`, `lti_resource_links/`, and `non_cc_assessments/` are present only if the course used those features.

---

## Two Parallel Systems — Read Both

Canvas IMSCC exports contain two systems that describe the course structure. Neither is complete alone.

| File | What it provides |
|------|-----------------|
| `imsmanifest.xml` | IMS CC-compliant org hierarchy; resource declarations for every content object |
| `course_settings/module_meta.xml` | Canvas-specific module structure; content types, URLs for external links, indent levels |

`imsmanifest.xml` tells you what resources exist. `module_meta.xml` tells you how Canvas presents them to students.

---

## imsmanifest.xml — Structure

**Namespace:** `http://www.imsglobal.org/xsd/imsccv1p1/imscp_v1p1`

```xml
<manifest identifier="{course-id}" xmlns="...">
  <metadata>
    <schema>IMS Common Cartridge</schema>
    <schemaversion>1.1.0</schemaversion>
    <lomimscc:lom> ... course title, date, rights ... </lomimscc:lom>
  </metadata>
  <organizations>
    <organization identifier="org_1" structure="rooted-hierarchy">
      <item identifier="LearningModules">           ← Single root for all modules
        <item identifier="{module-id}">             ← One per Canvas module
          <title>Week 1: Introduction</title>
          <item identifier="{item-id}" identifierref="{resource-id}">
            <title>Assignment Title</title>
          </item>
          <item identifier="{item-id}">             ← No identifierref = subheader only
            <title>Section Label</title>
          </item>
        </item>
      </item>
    </organization>
  </organizations>
  <resources>
    ... one <resource> element per content object ...
  </resources>
</manifest>
```

Items without `identifierref` are headers/subheaders with no linked resource. Items with `identifierref` link to a `<resource>` in the `<resources>` section.

---

## Resource Types in imsmanifest.xml

### Assignment
```xml
<resource identifier="{assignment-id}"
          type="associatedcontent/imscc_xmlv1p1/learning-application-resource"
          href="{assignment-id}/{slug}.html">
  <file href="{assignment-id}/{slug}.html"/>
  <file href="{assignment-id}/assignment_settings.xml"/>
</resource>
```

### Canvas Page (wiki page)
```xml
<resource identifier="{page-id}"
          type="webcontent"
          href="wiki_content/{slug}.html">
  <file href="wiki_content/{slug}.html"/>
</resource>
```

### External URL (web link)
```xml
<resource identifier="{link-id}"
          type="imswl_xmlv1p1"
          href="{link-id}.xml">
  <file href="{link-id}.xml"/>
</resource>
```
Root-level `{link-id}.xml` contains:
```xml
<webLink xmlns="http://www.imsglobal.org/xsd/imsccv1p1/imswl_v1p1" ...>
  <title>Link Title</title>
  <url href="https://..."/>
</webLink>
```

### File attachment
```xml
<resource identifier="{file-id}"
          type="associatedcontent/imscc_xmlv1p1/learning-application-resource"
          href="web_resources/{subfolder}/{filename}">
  <file href="web_resources/{subfolder}/{filename}"/>
</resource>
```

### LTI link
```xml
<resource identifier="{lti-id}" type="imsbasiclti_xmlv1p3">
  <file href="lti_resource_links/{lti-id}.xml"/>
</resource>
```

---

## module_meta.xml — Canvas Module Structure

**Location:** `course_settings/module_meta.xml`
**Namespace:** `http://canvas.instructure.com/xsd/cccv1p0`

This is the most important file for understanding what students see in Canvas.

```xml
<modules xmlns="http://canvas.instructure.com/xsd/cccv1p0" ...>
  <module identifier="{module-id}">
    <title>Week 1 — April 1</title>
    <workflow_state>active</workflow_state>    ← "active" = published
    <position>1</position>                     ← 1-based display order

    <items>
      <!-- External URL -->
      <item identifier="{item-id}">
        <content_type>ExternalUrl</content_type>
        <workflow_state>active</workflow_state>
        <title>Slide Deck</title>
        <identifierref>{item-id}</identifierref>  ← Self-referential for ExternalUrl
        <url>https://...</url>                    ← URL lives here, NOT in a weblink XML file
        <position>1</position>
        <new_tab>true</new_tab>
        <indent>0</indent>
      </item>

      <!-- Assignment -->
      <item identifier="{item-id}">
        <content_type>Assignment</content_type>
        <identifierref>{assignment-resource-id}</identifierref>
        <position>2</position>
        <new_tab>false</new_tab>
        <indent>0</indent>
      </item>

      <!-- Text subheader (no resource) -->
      <item identifier="{item-id}">
        <content_type>ContextModuleSubHeader</content_type>
        <title>Files for This Week</title>
        <position>3</position>
      </item>

      <!-- File attachment -->
      <item identifier="{item-id}">
        <content_type>Attachment</content_type>
        <identifierref>{file-resource-id}</identifierref>
        <position>4</position>
      </item>

      <!-- Canvas Page -->
      <item identifier="{item-id}">
        <content_type>WikiPage</content_type>
        <identifierref>{wiki-resource-id}</identifierref>
        <position>5</position>
      </item>

    </items>
  </module>
</modules>
```

### content_type Reference

| `content_type` | When used | Has `identifierref`? | Has `url` field? |
|---------------|-----------|----------------------|-----------------|
| `ExternalUrl` | Slide links, YouTube, external URLs | Self-referential | Yes |
| `Assignment` | All Canvas assignments | Yes → assignment folder | No |
| `Attachment` | Uploaded files (CSV, PDF, .ipynb, .py) | Yes → file resource | No |
| `WikiPage` | Canvas Pages | Yes → wiki resource | No |
| `ContextModuleSubHeader` | Section headers (no resource) | No | No |

**Critical:** For `ExternalUrl`, the item `identifier` and `identifierref` are the same value. The URL is in the `<url>` field — it does not reference a root-level weblink XML file. Root-level weblink XMLs in `imsmanifest.xml` are CC-compliance artifacts only.

---

## assignment_settings.xml — Field Reference

**Location:** `{assignment-id}/assignment_settings.xml`

Key fields:

| Field | Description | Notes |
|-------|-------------|-------|
| `<title>` | Assignment title | |
| `<due_at>` | Due date/time | UTC. PDT = UTC-7, PST = UTC-8 |
| `<all_day_date>` | Human-readable due date | Local date (not UTC shifted) |
| `<points_possible>` | Point value | 0.0 for pass/fail, >0 for scored |
| `<grading_type>` | Grading method | `pass_fail` = Complete/Incomplete in UI; `points` = numeric |
| `<submission_types>` | How students submit | comma-separated; see table below |
| `<workflow_state>` | Published state | `published` or `unpublished` |
| `<omit_from_final_grade>` | Excluded from grade calc | `true` or `false` |
| `<allowed_attempts>` | Attempt limit | `-1` = unlimited |
| `<assignment_group_identifierref>` | Which assignment group | Cross-references `assignment_groups.xml` |

### Submission Type Values

| XML value | Canvas UI label |
|-----------|----------------|
| `none` | No submission |
| `online_text_entry` | Text entry |
| `online_url` | URL submission |
| `online_upload` | File upload |
| `online_text_entry,online_url` | Multiple types |

### Due Date UTC Conversion

`<due_at>` is always stored in UTC, regardless of where the course was created. `<all_day_date>` reflects the local calendar date in the instructor's timezone.

To convert `<due_at>` to local time: subtract the UTC offset for the relevant timezone.

| Timezone | UTC offset | Example: T05:00:00 UTC = |
|----------|-----------|--------------------------|
| Pacific (PDT) | UTC−7 | 10:00 PM previous day |
| Pacific (PST) | UTC−8 | 9:00 PM previous day |
| Mountain (MDT) | UTC−6 | 11:00 PM previous day |
| Central (CDT) | UTC−5 | midnight (same day) |
| Eastern (EDT) | UTC−4 | 1:00 AM same day |
| UK (BST) | UTC+1 | 6:00 AM same day |
| CET (CEST) | UTC+2 | 7:00 AM same day |

When presenting due dates to users, always ask which timezone they're working in, or look at `<time_zone_edited>` in `assignment_settings.xml` — it records the instructor's timezone at the time the assignment was created. Use that as the reference if no other information is available.

---

## course_settings.xml — Key Fields

```xml
<course identifier="{course-id}" xmlns="http://canvas.instructure.com/xsd/cccv1p0" ...>
  <title>Course Full Title</title>
  <course_code>DEPT 100 A</course_code>
  <start_at>...</start_at>           ← UTC
  <conclude_at>...</conclude_at>     ← UTC
  <default_view>modules</default_view>
  <is_public>false</is_public>
  <grading_standard_enabled>false</grading_standard_enabled>
</course>
```

---

## Recommended Extraction Order

1. `course_settings/course_settings.xml` — course title, dates, basic metadata
2. `course_settings/module_meta.xml` — full module and item structure
3. `imsmanifest.xml` — resolve `identifierref` values to file paths
4. `{assignment-id}/assignment_settings.xml` — grading config and due dates per assignment
5. `{assignment-id}/{slug}.html` — student-facing assignment descriptions
6. `course_settings/assignment_groups.xml` — resolve group references if needed
7. `course_settings/rubrics.xml` — rubric definitions if needed
8. `wiki_content/{slug}.html` — Canvas Page content if needed

---

## Common Extraction Gotchas

1. **`ExternalUrl` URLs are in `module_meta.xml`**, not in root-level weblink XML files. If you need a slide deck URL, look in the `<url>` field of the module item.

2. **Smart quotes and HTML entities** may appear in assignment body HTML. Decode (`&amp;`, `&lt;`, etc.) before processing text.

3. **`pass_fail` = Complete/Incomplete**: The XML value is `pass_fail`; Canvas renders this as "Complete/Incomplete" in the gradebook UI.

4. **Module vs. item positions**: Each module has a `<position>` (its order among modules), and each item has its own `<position>` within its module. Both are 1-based.

5. **`workflow_state` at both levels**: A module can be `active` while individual items within it are `unpublished`.

6. **Identifier formats vary by export**: Some Canvas exports use UUIDs, others use `g`-prefixed hex strings, others use human-readable slugs. Treat all identifiers as opaque strings.

7. **Rubric "Remove points" state** is not stored in IMSCC — it must be checked in Canvas directly.

---

*Covers IMS Common Cartridge v1.1 as exported by Canvas LMS.*
