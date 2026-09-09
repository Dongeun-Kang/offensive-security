# Offensive Security

Personal offensive security notes, lab writeups, references, and reusable templates.

## Structure

- [writeups/](writeups/) - Hands-on labs grouped by domain and topic.
- [reports/](reports/) - Formal assessment-style reports for completed labs.
- [checklists/](checklists/) - Repeatable review and enumeration checklists.
- [references/](references/) - Ports, tools, glossary, and quick reference material.
- [templates/](templates/) - Reusable formats for future writeups and reports.
- [assets/](assets/) - Images, diagrams, payload notes, and supporting files.

## Naming Rules

- Use lowercase `kebab-case` for files and folders.
- Put lab writeups under `writeups/<domain>/<topic>/`.
- Keep screenshots and large supporting files under `assets/`.
- Prefer one focused Markdown file per lab.
- Avoid characters that break Windows paths, such as `:`, `*`, `?`, `<`, `>`, and `|`.

## Add a New Writeup

1. Copy `templates/writeup-template.md`.
2. Save it under the closest matching category in `writeups/`.
3. Add screenshots or supporting files under `assets/`.
4. Link the new file from the category `README.md`.

## Add a New Report

1. Copy `templates/report-template.md`.
2. Save it under the closest matching folder in `reports/`.
3. Link the report from `reports/README.md`.
4. Keep report language concise, evidence-based, and remediation focused.
