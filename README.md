# Specification Compliance Statement – an AI Skill for Claude and ChatGPT

An agent skill (open `SKILL.md` format) that turns a construction project specification section into a ready-to-fill **Compliance Statement in Excel** for material / technical submittals. It works in **Claude** (app and Claude Code) and **ChatGPT** (Skills, Codex), and in any other assistant that supports `SKILL.md` skills with a Python sandbox.

Attach a specification section (text PDF, Word, or scanned PDF) and ask the assistant to *"prepare the compliance statement"*. You get back an `.xlsx` with every Part, Article and sub-clause in its own row, ready for your team to fill in.

## What it produces

| Sl. No | Specification Requirement | Compliance Status | Remarks |
|---|---|---|---|
| | **SECTION 08 71 00** | | |
| | **DOOR HARDWARE** | | |
| **PART 1** | **GENERAL** | | |
| **1.01** | **SUMMARY** | | |
| | A. Section includes ... | | |
| | 1. ... | | |
| | a. ... | | |

- **All Parts** (General, Products, Execution), one row per sub-clause (A. / 1. / a. / 1) / a))
- **Compliance Status and Remarks left blank** for the reviewer
- **Project Title and Submittal Title left blank**; the date cell uses `=TODAY()`
- Article numbers are stored as text, so `3.10` never turns into `3.1`
- Justified Arial 10 text, with row heights sized so nothing is cut off
- **Print-ready:** A4 landscape, 1 page wide, header rows repeated on every page, "Page x of y" footer
- Repeating page headers and footers from the spec are removed automatically
- Fixes only obvious word-break / spacing errors from PDF extraction (`e quipment` → `equipment`). Abbreviations, standard references, numbers and units are never changed
- Reports suspected specification errors (conflicting values, missing brackets, leftover unit conversions) so you can raise an RFI

Works with MasterFormat-style specifications (`SECTION xx xx xx` → `PART 1/2/3` → `1.01` articles).

## Install

Download **`spec-compliance-statement.zip`** from this repository or from the [Releases](../../releases) page. The same package works for every platform below.

### Claude app (claude.ai, desktop, mobile)

1. Make sure code execution is on: **Settings → Capabilities** ("Create and edit files").
2. Go to **Customize → Skills**, click **+**, choose **Create skill → Upload a skill**, and upload the ZIP.
3. Switch the skill on.

Available on Free, Pro, Max, Team and Enterprise. On Team / Enterprise an owner may need to allow user-created skills.

### Claude Code

```bash
git clone https://github.com/ljworld/Specification-Compliance-Statement.git
mkdir -p ~/.claude/skills
cp -r Specification-Compliance-Statement/spec-compliance-statement ~/.claude/skills/
```

To share it with one project's team instead, copy it to `.claude/skills/` inside that project.

### ChatGPT (web / desktop)

1. In the sidebar, open **Plugins** and select the **Skills** tab.
2. Click **+** (Create), then choose **Upload from your computer**.
3. Select the ZIP. If your version asks for a folder instead, unzip it first and select the `spec-compliance-statement` folder.
4. Wait for ChatGPT's security scan. Most skills become available straight away; some are marked **Needs Review** until a workspace admin approves them.

**Plans:** ChatGPT Skills are currently offered on **Business, Enterprise, Healthcare and Edu** workspaces, not on Free, Plus or Pro. If you don't see a Skills tab, ask your workspace admin to turn Skills on (**Admin settings → Permissions & Roles**). Availability changes over time, so check your own sidebar.

**Use:** type `@` in the message box and pick **spec-compliance-statement**, or simply ask for a compliance statement and ChatGPT will pick the skill from its description.

### OpenAI Codex (CLI / IDE extension)

```bash
git clone https://github.com/ljworld/Specification-Compliance-Statement.git
mkdir -p ~/.agents/skills
cp -r Specification-Compliance-Statement/spec-compliance-statement ~/.agents/skills/
```

To share it with one repository's team, copy it to `.agents/skills/` in that repository. Codex detects new skills automatically; restart it if the skill doesn't appear.

### Any other assistant

The skill folder is plain `SKILL.md` + `scripts/compliance.py`. With assistants that don't support skills, you can paste the contents of `SKILL.md` as instructions and attach your spec. The script is also embedded at the end of `SKILL.md`, so the assistant can recreate it.

## Use

Attach the specification file and write, for example:

> Prepare the compliance statement for the attached specification.

> Prepare compliance statements for these three sections – Part 2 and Part 3 only.

## Requirements

The assistant's sandbox handles these. Tools that are missing are worked around where possible:

| Need | Used | Fallback if missing |
|---|---|---|
| Excel output | Python 3 + `openpyxl` | none (required) |
| Text PDFs | `pdftotext` (poppler) | `pypdf` or `pdfplumber` |
| Scanned PDFs | `tesseract` OCR | the assistant reads the page images visually, or asks you for a text PDF |
| Word files | LibreOffice (`soffice`) | asks you to save the Word file as PDF |
| Print-layout check | LibreOffice (`soffice`) | skipped; check the print preview in Excel |

Claude's environment includes all of these. In ChatGPT, text PDFs work out of the box. If a Word or scanned spec is refused, save or export it as a text PDF.

## Repository layout

```
Specification-Compliance-Statement/   <- this repository
spec-compliance-statement/        <- the skill (upload this folder, zipped)
  SKILL.md                        <- instructions the assistant follows
  scripts/compliance.py           <- extraction + Excel builder
spec-compliance-statement.zip     <- ready-to-upload package
```

You can also run the script on its own:

```bash
pip install openpyxl pypdf
python3 compliance.py extract spec.pdf -o spec.json
python3 compliance.py build spec.json -o Compliance_Statement.xlsx [--fixes fixes.json]
```

## Limitations

- Built for MasterFormat-style numbering. Other numbering schemes may need manual correction of the extracted rows.
- OCR of poor-quality scans can misread characters. The skill tells the assistant to proofread scanned sources against the page images.
- The header font is Arial Rounded MT Bold. Computers without it fall back to Arial.
- The paper size is A4. Change `PAPERSIZE_A4` in `compliance.py` if you print on Letter.
- AI output can contain mistakes. Always check the generated statement against the original specification before submitting it.

## License

[MIT](LICENSE)
