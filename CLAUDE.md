# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A static-hosting repo for personal running/training plan pages. There is no build
system, package manager, or source code in this repo (no `package.json`, no
bundler config) — every file here is served as-is. There are no test or lint
commands because there is nothing to build or execute.

- `index.html` — a hand-written directory page listing links to each published plan.
- `plans/<name>.html` — one file per athlete's training plan (e.g. `plans/hakam.html`,
  `plans/pari-plan.html`).

## Architecture of a plan file (`plans/*.html`)

Each file under `plans/` is **not hand-authored HTML**. It is the full static
export of a compiled Svelte 5 single-page app: a self-contained, minified JS
bundle inlined in a `<script type="module" crossorigin>` tag, plus the
plan's data embedded verbatim as JSON in a
`<script type="application/json" id="plan-data">` tag at the end of `<body>`.
The Svelte app's source is *not* part of this repo — only its rendered output is
committed here.

Confirmed from the current files: the entire minified app bundle (the
`<script type="module">` contents) is byte-for-byte identical across
`hakam.html` and `pari-plan.html`. The only thing that differs between plan
files is the JSON payload in the `plan-data` script tag. So when working with
these files, the JSON block is the meaningful "content"; the rest is
boilerplate you should copy from an existing plan file rather than write by hand.

The embedded JSON has roughly this shape (see the `plan-data` block at the
bottom of any `plans/*.html`):
```
{
  "version": "1.0",
  "meta": { "id", "athlete", "event", "eventDate", "planStartDate",
            "planEndDate", "totalWeeks", "generatedBy": "Claude Running Coach" },
  "preferences": { units for swim/bike/run, firstDayOfWeek },
  "assessment": { foundation, currentForm, ... },
  ... weekly training schedule ...
  "hydration": {...},
  "mentalStrategy": { "mantras": [...], "breakpoints": [...] },
  "raceWeekChecklist": [...],
  "taper": {...}
}
```

### Important gotcha (from git history)

An earlier commit (`00d4f59`) mistakenly committed the **raw JSON plan spec**
directly as the contents of `plans/hakam.html` instead of the rendered app.
It had to be fixed in a follow-up commit (`ad4cb7a`, "fix: push rendered
HTML, not raw JSON") by replacing the file with the actual compiled
Svelte bundle + embedded JSON. When adding or updating a plan file, always
commit the full rendered HTML export (bundle + `plan-data` script), never
the bare JSON.

### Reading/editing these files

Plan files are large (~750KB) and contain single lines hundreds of thousands
of characters long (the minified bundle), which will blow past normal
file-read limits. To inspect or edit one:
- Use `grep -oE '<script[^>]*>'` to locate the script tags rather than reading the whole file.
- The JSON payload is the last `<script type="application/json" id="plan-data">…</script>`
  block before `</body>` — target that region specifically (e.g. with `tail`/`sed`)
  instead of reading the file in full.

## Adding a new plan

1. Add `plans/<name>.html` as a full rendered export (bundle + `plan-data` JSON),
   following the existing gotcha above.
2. Add a corresponding link entry to `index.html` — this is a manual step and is
   easy to forget (e.g. `plans/pari-plan.html` was added without an `index.html`
   entry).
