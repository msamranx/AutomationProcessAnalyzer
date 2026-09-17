# process-analyzer

A Claude Skill that turns a free-text description of a business process into two Word documents, produced in strict sequence:

1. **Process Design Document (PDD)** — the process as it works today (as-is): actors, systems, a numbered narrative, a text-based flowchart, and a per-step automation-opportunity rating.
2. **Solution Design Document (SDD)** — the process as it should work after automation (to-be): a strict **Pure RPA** vs **RPA+AI** decision (never a third option), its own flowchart, explicit limitations, and a module breakdown so a development team can divide the build across people.

No diagramming tool is required — flowcharts are rendered as bordered text boxes and arrows directly inside the `.docx` files.

## Repository structure

```
process-analyzer/
├── SKILL.md                              # Skill definition Claude reads to trigger and run this
└── references/
    ├── pdd_structure.md                  # Section-by-section outline for the PDD
    ├── sdd_structure.md                  # Section-by-section outline for the SDD
    ├── rpa_decision_framework.md         # Per-step scoring + the binary Pure RPA / RPA+AI decision rule
    └── flowchart_conventions.md          # The text-based box/arrow flowchart convention
```

## Installing

**Claude.ai / Claude apps (packaged skill):**
1. Download `process-analyzer.skill` from this repo's releases (or package it yourself — see below).
2. In Claude, go to Settings → Capabilities → Skills, and upload the `.skill` file.

**Claude Code / self-hosted:**
Copy the `process-analyzer/` folder into your skills directory (e.g. `~/.claude/skills/` or your project's `.claude/skills/`), preserving the folder structure above.

**Packaging it yourself** (requires the [skill-creator](https://github.com/anthropics/skills) tooling):
```bash
python -m scripts.package_skill path/to/process-analyzer path/to/output-dir
# → path/to/output-dir/process-analyzer.skill
```

## Usage

Invoke it with `/process-analyzer` followed by a free-text description of the process, or just describe a process and ask for a PDD/SDD/automation assessment — the skill triggers on either.

### Example

**Input:**
```
/process-analyzer RPA Dispatcher/Performer automation for RM report reconciliation.
A robot logs into a portal, navigates, and downloads the RM Report. A lightweight
Dispatcher filters rows where Status = New and enqueues each row (columns A–N +
Req ID) into an Orchestrator Queue. The Performer dequeues each item and checks
if the Req ID exists in either tracker. If yes, it reads the Productivity Tracker
status, syncs the Master Tracker to match, and skips. If no, both trackers are
written with Status = Pending. The robot then calls the Oracle B2C API and
compares columns A–N field-by-field. If all match, status is set to Completed
and both trackers updated. If any mismatch, status is set to a dedicated
Mismatch status and the specific mismatched fields are logged into a dedicated
Mismatch Column in both the Productivity Tracker and Master Tracker.
```

**What it produces:**
- `RM_Report_Reconciliation_Process_Design_Document.docx` — as-is narrative and flowchart (portal login → filter/enqueue → Req ID lookup → Oracle B2C comparison → status write-back), plus a per-step table rating every step as a rules-based automation candidate.
- `RM_Report_Reconciliation_Solution_Design_Document.docx` — recommends **Pure RPA**, naming the specific steps that ruled out an AI component (exact-match lookups, structured field comparison, no free text anywhere in the flow); a to-be flowchart marking every step `[BOT]`; a limitations section (portal fragility, schema drift risk, mismatch resolution staying manual); and a 7-module breakdown (Portal Login, Dispatcher, Tracker Lookup/Sync, Oracle B2C Integration, Comparison Engine, Tracker Write-Back, Exception Handling) for parallel development.

### Other example prompts that trigger this skill

```
Here's how our invoice approval works: AP receives an invoice by email, checks
if the vendor exists in SAP, routes anything over $10,000 to a manager for
sign-off, and otherwise posts it directly. Can you document this and tell me
if it's a good candidate for automation?
```

```
Create a PDD and SDD for our employee onboarding process — HR gets a new-hire
form, creates accounts in AD and Workday, and emails IT to provision hardware.
```

## What makes the SDD's decision "strict"

The Solution Design Document always commits to exactly one of two labels — **Pure RPA** or **RPA+AI** — and never a hedge or an intermediate category (no "RPA with a future AI roadmap", no "IPA"). The rule, defined in [`references/rpa_decision_framework.md`](references/rpa_decision_framework.md):

- **Pure RPA** — every automatable step is rules-based, works on structured data, and needs no judgment, classification, or generation.
- **RPA+AI** — at least one automatable step requires interpreting unstructured input (free text, scanned documents, images), judgment-based decisioning, prediction, or generative drafting.

The decision is always stated with the specific PDD step number(s) that drove it.

## License

Add your preferred license here before publishing.
