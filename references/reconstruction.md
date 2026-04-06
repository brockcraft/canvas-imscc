# IMSCC Reconstruction Reference — Building a Canvas-Compatible File

Use this document when building a new `.imscc` file for import into Canvas. Many of these requirements were discovered through repeated silent import failures — follow them exactly.

---

## Python Setup

Recommended approach: build with Python's `zipfile` module and `xml.etree.ElementTree`.

```bash
pip install markdown   # if converting Markdown assignment bodies to HTML
```

---

## ZIP Requirements (critical)

Canvas has strict ZIP requirements that cause silent "unexpected error" failures if violated:

| Requirement | Correct | Wrong |
|-------------|---------|-------|
| Compression | `ZIP_STORED` (uncompressed) | `ZIP_DEFLATED` — fails silently |
| First entry | `imsmanifest.xml` must be written first | Any other file first — fails silently |
| `canvas_export.txt` | Must be non-empty | Empty file — may fail |

```python
import zipfile

with zipfile.ZipFile("course.imscc", "w", compression=zipfile.ZIP_STORED) as zf:
    zf.writestr("imsmanifest.xml", manifest_xml)   # MUST be first
    zf.writestr("course_settings/canvas_export.txt", "course, version 1")
    # ... rest of files
```

---

## Identifier Format

All identifiers in a Canvas-built IMSCC must follow this format:

```
"g" + 32-char MD5 hex string
```

Human-readable IDs (e.g., `assignment_week1`) are silently rejected. Generate identifiers like this:

```python
import hashlib

def make_id(seed: str) -> str:
    return "g" + hashlib.md5(seed.encode()).hexdigest()
```

Use a stable, unique seed per object (e.g., `"course_settings"`, `"module_week1"`, `"assignment_a1"`). This ensures IDs are deterministic across rebuilds.

---

## imsmanifest.xml Requirements

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest identifier="{course-id}"
          xmlns="http://www.imsglobal.org/xsd/imsccv1p1/imscp_v1p1"
          xmlns:lom="http://ltsc.ieee.org/xsd/imsccv1p1/LOM/resource"
          xmlns:lomimscc="http://ltsc.ieee.org/xsd/imsccv1p1/LOM/manifest"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://www.imsglobal.org/xsd/imsccv1p1/imscp_v1p1
                              https://www.imsglobal.org/profile/cc/ccv1p1/ccv1p1_imscp_v1p2_v1p0.xsd">
```

**Required namespace attributes:**
- `xmlns:lom` and `xmlns:lomimscc` must be present on the root `<manifest>` element
- `xsi:schemaLocation` must cite `ccv1p1_imscp_v1p2_v1p0.xsd` — using `v1p1` in the schema path causes failures

**Required LOM metadata block in `<metadata>`:**
```xml
<metadata>
  <schema>IMS Common Cartridge</schema>
  <schemaversion>1.1.0</schemaversion>
  <lomimscc:lom>
    <lomimscc:general>
      <lomimscc:title><lomimscc:string>{Course Title}</lomimscc:string></lomimscc:title>
    </lomimscc:general>
    <lomimscc:lifeCycle>
      <lomimscc:contribute>
        <lomimscc:date><lomimscc:dateTime>{ISO date}</lomimscc:dateTime></lomimscc:date>
      </lomimscc:contribute>
    </lomimscc:lifeCycle>
    <lomimscc:rights>
      <lomimscc:copyrightAndOtherRestrictions>
        <lomimscc:value>yes</lomimscc:value>
      </lomimscc:copyrightAndOtherRestrictions>
      <lomimscc:description>
        <lomimscc:string>Private (Copyrighted) - http://en.wikipedia.org/wiki/Copyright</lomimscc:string>
      </lomimscc:description>
    </lomimscc:rights>
  </lomimscc:lom>
</metadata>
```

**Resource ordering in `<resources>`:**
- Syllabus resource must appear **before** the course settings bundle
- Course settings bundle: do NOT include `intendeduse="coursesettings"` — that attribute does not exist in the schema

```xml
<resources>
  <!-- Syllabus first -->
  <resource identifier="{course-id}_syllabus"
            type="associatedcontent/imscc_xmlv1p1/learning-application-resource"
            href="course_settings/syllabus.html"
            intendeduse="syllabus">
    <file href="course_settings/syllabus.html"/>
  </resource>

  <!-- Course settings bundle second (NO intendeduse attribute) -->
  <resource identifier="{course-id}"
            type="associatedcontent/imscc_xmlv1p1/learning-application-resource"
            href="course_settings/canvas_export.txt">
    <file href="course_settings/course_settings.xml"/>
    <file href="course_settings/module_meta.xml"/>
    <file href="course_settings/assignment_groups.xml"/>
    <file href="course_settings/rubrics.xml"/>
    <file href="course_settings/files_meta.xml"/>
    <file href="course_settings/events.xml"/>
    <file href="course_settings/late_policy.xml"/>
    <file href="course_settings/context.xml"/>
    <file href="course_settings/media_tracks.xml"/>
    <file href="course_settings/canvas_export.txt"/>
  </resource>

  <!-- Then all other resources (assignments, pages, links, files) -->
</resources>
```

---

## course_settings/ Stub Files

Several stub files are required even if the course doesn't use those features. All stubs must include `xmlns:xsi` and `xsi:schemaLocation`.

### context.xml
Root element must be `<context_info>` — NOT `<context>`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<context_info xmlns="http://canvas.instructure.com/xsd/cccv1p0"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="...">
</context_info>
```

### media_tracks.xml
Root element must be `<media_tracks>` — NOT `<mediaTracks>`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<media_tracks xmlns="http://canvas.instructure.com/xsd/cccv1p0" ...>
</media_tracks>
```

### late_policy.xml
Root element must have an `identifier=` attribute:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<late_policy identifier="{late-policy-id}"
             xmlns="http://canvas.instructure.com/xsd/cccv1p0" ...>
  <missing_submission_deduction_enabled>false</missing_submission_deduction_enabled>
  <missing_submission_deduction>0.0</missing_submission_deduction>
  <late_submission_deduction_enabled>false</late_submission_deduction_enabled>
  <late_submission_deduction>0.0</late_submission_deduction>
  <late_submission_interval>day</late_submission_interval>
  <late_submission_minimum_percent_enabled>false</late_submission_minimum_percent_enabled>
  <late_submission_minimum_percent>0.0</late_submission_minimum_percent>
</late_policy>
```

### files_meta.xml, events.xml, grading_standards.xml
Empty stubs — root element with namespace declarations, no children:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<files xmlns="http://canvas.instructure.com/xsd/cccv1p0"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">
</files>
```

---

## assignment_settings.xml

Do NOT include `<allowed_attempts>` — it is absent from Canvas exports and may cause import issues.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<assignment identifier="{assignment-id}"
            xmlns="http://canvas.instructure.com/xsd/cccv1p0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="...">
  <title>Assignment Title</title>
  <due_at>2026-04-09T05:00:00</due_at>     ← UTC always
  <all_day_date>2026-04-08</all_day_date>  ← Local date
  <points_possible>0.0</points_possible>
  <grading_type>pass_fail</grading_type>   ← or "points"
  <submission_types>online_url</submission_types>
  <workflow_state>published</workflow_state>
  <omit_from_final_grade>true</omit_from_final_grade>
  <assignment_group_identifierref>{group-id}</assignment_group_identifierref>
  <position>1</position>
  <all_day>false</all_day>
  <turnitin_enabled>false</turnitin_enabled>
  <peer_reviews>false</peer_reviews>
  <automatic_peer_reviews>false</automatic_peer_reviews>
  <freeze_on_copy>false</freeze_on_copy>
  <hide_in_gradebook>false</hide_in_gradebook>
  <only_visible_to_overrides>false</only_visible_to_overrides>
  <moderated_grading>false</moderated_grading>
  <grader_count>0</grader_count>
  <anonymous_grading>false</anonymous_grading>
  <post_policy>
    <post_manually>true</post_manually>
  </post_policy>
</assignment>
```

### Due Date UTC Conversion

`<due_at>` must be stored in UTC — Canvas converts to the viewer's local timezone for display. Convert local time to UTC by adding the UTC offset.

**General formula:** `<due_at>` = local time + UTC offset (as a positive shift)

| Timezone | UTC offset | Local 10:00 PM = UTC |
|----------|-----------|----------------------|
| Pacific (PDT) | UTC−7 | next day T05:00:00 |
| Pacific (PST) | UTC−8 | next day T06:00:00 |
| Mountain (MDT) | UTC−6 | next day T04:00:00 |
| Central (CDT) | UTC−5 | next day T03:00:00 |
| Eastern (EDT) | UTC−4 | next day T02:00:00 |
| UK (BST) | UTC+1 | same day T21:00:00 |
| CET (CEST) | UTC+2 | same day T20:00:00 |

`<all_day_date>` is always the **local** calendar date (not the UTC-shifted date). If the user hasn't specified their timezone, look at `<time_zone_edited>` in existing assignment files or ask before converting.

Also set `<time_zone_edited>` to the instructor's timezone name as Canvas recognizes it (e.g., `Pacific Time (US & Canada)`, `Eastern Time (US & Canada)`, `London`, `Amsterdam`). This is informational metadata — it doesn't affect the UTC value Canvas uses.

---

## Assignment Body HTML

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<title>Assignment: {Title}</title>
</head>
<body>
  <!-- Student-facing assignment description -->
  <!-- File references: src="$IMS-CC-FILEBASE$/web_resources/..." -->
</body>
</html>
```

`$IMS-CC-FILEBASE$` is a Canvas macro that resolves to the course Files root at runtime.

---

## Wiki Page (Canvas Page) HTML

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
<title>Page Title</title>
<meta name="identifier" content="{page-id}"/>
<meta name="editing_roles" content="teachers"/>
<meta name="workflow_state" content="active"/>
</head>
<body>
  <!-- Page content -->
</body>
</html>
```

---

## ExternalUrl in module_meta.xml

For external links (slide decks, YouTube, etc.), the item `identifier` and `identifierref` must be the same value (self-referential). The URL goes in a `<url>` field on the item:

```xml
<item identifier="{item-id}">
  <content_type>ExternalUrl</content_type>
  <workflow_state>active</workflow_state>
  <title>Week 1 Slides</title>
  <identifierref>{item-id}</identifierref>   ← same as identifier
  <url>https://docs.google.com/presentation/...</url>
  <position>1</position>
  <new_tab>true</new_tab>
  <indent>0</indent>
  <link_settings_json>null</link_settings_json>
</item>
```

These items also get a corresponding root-level `{item-id}.xml` weblink file (for CC compliance) and a matching `<resource>` entry in `imsmanifest.xml` — but the actual URL Canvas uses comes from `module_meta.xml`, not from that file.

---

## Post-Import Manual Steps (cannot be done via IMSCC)

Always surface these to the user after a successful import:

1. **Attach rubrics to assignments** — Rubrics defined in `rubrics.xml` are created in Canvas's Rubrics bank but are NOT automatically attached to individual assignments. Must be done per assignment in Canvas.

2. **"Remove points from rubric"** — The checkbox that makes a rubric non-scoring cannot be set via IMSCC. Must be checked manually per assignment after import.

3. **Placeholder external URLs** — If slide deck URLs weren't known at build time, update them in Canvas via the module item edit interface.

4. **File link resolution** — If assignment descriptions reference files via `$IMS-CC-FILEBASE$`, verify the links resolve correctly after import by opening an assignment and clicking a file link.

---

## Pre-Packaging Checklist

Before writing the ZIP:

- [ ] `imsmanifest.xml` will be the first entry written
- [ ] All identifiers match the `g` + 32-char MD5 format
- [ ] Syllabus resource appears before the course settings bundle in `<resources>`
- [ ] Course settings bundle has no `intendeduse` attribute
- [ ] `context.xml` root is `<context_info>`
- [ ] `media_tracks.xml` root is `<media_tracks>`
- [ ] `late_policy.xml` root has an `identifier=` attribute
- [ ] All stub XMLs include `xmlns:xsi` and `xsi:schemaLocation`
- [ ] `assignment_settings.xml` does NOT include `<allowed_attempts>`
- [ ] ZIP compression is `ZIP_STORED`
- [ ] Post-import manual steps are documented for the user
