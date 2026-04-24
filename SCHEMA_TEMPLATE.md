# Wiki Schema Template

> This file is a reference template. The LLM writes an actual `wiki/SCHEMA.md`
> during initialization, populated with vault-specific content in sections 1 & 2.
> Sections 3–7 are copied verbatim from the init prompt and should not be modified
> unless you are intentionally changing the wiki's operating rules.

---

## 1. VAULT OVERVIEW

<!-- Written by the LLM during BOOTSTRAP. Describes the owner, their context,
     and what this vault tracks. Example:

     This vault belongs to [Name], a [role] at [company]. It tracks work across
     [teams/projects], personal life, and ongoing learning. Primary sources are
     daily journals (journals/), meeting transcripts (Granola/), and project
     hub notes (Projects/).
-->

## 2. WIKI FOLDER MAP

<!-- Written by the LLM during BOOTSTRAP. One row per subfolder. -->

| Folder | Contents | Primary Sources | Update Frequency |
|--------|----------|-----------------|------------------|
| wiki/work/ | Projects, areas, decisions, blockers | Projects/, Areas/, Granola/, journals/ | Every ingest |
| wiki/people/ | Person profiles | People/, Granola/, journals/ | When new person appears |
| wiki/meetings/ | Themes, decisions, action items | Granola/, Meetings/, Areas/ | Every meeting ingest |
| wiki/personal/ | Health, family, goals | journals/, Areas/Personal/ | Weekly |
| wiki/knowledge/ | Concepts, learnings, references | Resources/, Clippings/, News Briefings/ | On article ingest |
| wiki/queries/ | Filed query answers and analyses | LLM-generated | On QUERY FILE IT |

## 3. NAVIGATION FILES

Two special files form the LLM's navigation layer. Every session must read
both before doing any other work. Their formats are defined here and must
be maintained exactly — they are machine-readable as well as human-readable.

### index.md

**PURPOSE:** Content catalog. The LLM reads this first on every QUERY to find
relevant pages, then drills into them. This replaces the need for
embedding-based RAG at moderate scale (up to ~hundreds of pages).

**FORMAT:**
```
# Wiki Index
> Last updated: YYYY-MM-DD | Pages: N | Sources ingested: N

## Work
| Page | Summary | Last Updated | Sources |
|------|---------|--------------|---------|
| [[wiki/work/overview]] | Owner role, team, company, current focus | YYYY-MM-DD | N |

## People
...

## Meetings
...

## Personal
...

## Knowledge
...

## Queries Filed
| Page | Question | Date |
|------|----------|------|
```

**MAINTENANCE RULES:**
- Add a row whenever a new wiki page is created.
- Update the summary and last_updated date whenever a page is significantly changed.
- Update the header counts (Pages, Sources ingested) on every ingest.
- Never remove rows. If a page is deprecated, mark it: ~~[[link]]~~ *(deprecated YYYY-MM-DD)*
- Keep summaries to one line (~80 chars). They must be scan-readable at a glance.
- The index is the only file where the LLM is allowed to overwrite existing lines
  (to update date and source count). All other wiki files are append/amend only.

### log.md

**PURPOSE:** Chronological, append-only record of all operations — ingests,
queries, lint passes, and bootstrap. Gives the LLM a timeline of what has
been done and what was learned recently. Also parseable with standard unix tools.

**FORMAT:**
Each entry uses this exact heading format:
```
## [YYYY-MM-DD] OPERATION | Label
```

Where OPERATION is one of: `BOOTSTRAP`, `INGEST`, `INGEST-BATCH`, `QUERY`, `LINT`

And Label is a short human-readable identifier:
- INGEST: the source file or folder name
- INGEST-BATCH: the scope (e.g. "Granola 2026-01 to 2026-03")
- QUERY: truncated question (≤50 chars)
- LINT: "full" or scope if partial
- BOOTSTRAP: "initial setup"

**Examples:**
```
## [2026-04-02] INGEST | Granola/2026-04-02
## [2026-04-10] QUERY | What are the open blockers on SAP Commissions?
## [2026-04-15] LINT | full
## [2026-04-23] BOOTSTRAP | initial setup
```

**PARSEABILITY** — The `## [YYYY-MM-DD]` prefix enables unix queries:
```bash
grep "^## \[" wiki/log.md              # all entries
grep "^## \[" wiki/log.md | tail -10   # last 10 entries
grep "INGEST" wiki/log.md              # all ingests
grep "2026-04" wiki/log.md             # all April 2026 activity
```

Never use `##` headings inside a log entry body — only at the entry level.
Use plain text, bullet points, or bold labels inside entry bodies.

## 4. NOTE CONVENTIONS

- Every wiki page requires this frontmatter:
  ```yaml
  ---
  title: [Page Title]
  type: wiki
  sources: [list of vault paths this was derived from]
  last_updated: YYYY-MM-DD
  ---
  ```
- Internal wiki links: `[[wiki/path/to/page]]`
- Source citations: `[filename](vault/path/filename.md)`
- Date format: ISO 8601 (YYYY-MM-DD) everywhere
- Every page must appear in `wiki/index.md`
- Every operation must be logged in `wiki/log.md`

## 5. OPERATING PRINCIPLES

- `wiki/` is the LLM's layer. Raw vault folders are read-only sources.
- Never modify files outside `wiki/` without explicit owner instruction.
- Never delete or overwrite raw source files.
- Every fact must be traceable to a source file.
- When in doubt about sensitive content: summarize themes, not details.
- Always read `wiki/index.md` before re-reading raw sources for a query.
- Always read the last 10 entries of `wiki/log.md` at session start to
  understand what has been done recently.
- After any significant operation, append to `wiki/log.md`.

## 6. SESSION START PROTOCOL

At the start of every new session (before responding to any owner request):
1. Read `wiki/SCHEMA.md` (this file) — orient to structure and skills.
2. Read `wiki/index.md` — know what pages exist and their summaries.
3. Read the last 10 log entries in `wiki/log.md` — know what was done recently.
   (`grep "^## \[" wiki/log.md | tail -10`, then read those entries)

These three reads are the complete context needed to resume work from any state.

## 7. SKILLS

The owner invokes skills by saying: `INGEST [source]`, `QUERY [question]`,
or `LINT`. Execute the full procedure for the named skill. Do not abbreviate.

---

### SKILL: INGEST

**Invocation:** `INGEST [path or description of source]`  
**Purpose:** Process a new source into the wiki. A single source typically
touches 10–15 wiki pages. Stay interactive — present findings before writing.

**PROCEDURE:**

**Step 1 — READ**  
Read the source file(s) in full using `obsidian_get_file_contents`.
If the source is a folder (e.g. a Granola date folder), read all files in it.

**Step 2 — EXTRACT & DISCUSS** *(interactive checkpoint)*  
Before writing anything, present to the owner:

a) **SUMMARY** (3–5 sentences): What is this source about?

b) **KEY TAKEAWAYS** (bulleted): The most important facts, decisions, or ideas.
   - Meeting transcripts: decisions made, action items, blockers raised.
   - Journal entries: themes, mood, events, intentions.
   - Articles/clippings: core argument, evidence, implications.

c) **WIKI PAGES TO UPDATE** (list with rationale):
   For each page you plan to touch: page path + what you will add.

d) **NEW PAGES TO CREATE** (if any):
   Path and what it will contain.

Wait for owner feedback before writing. The owner may say "skip [page]",
"emphasize [topic]", or "go ahead".

**Step 3 — WRITE**  
a) Update each existing wiki page. Append only — never remove existing content.  
b) Create any new pages with standard frontmatter.  
c) Update `wiki/index.md`: add rows, update dates, increment source count.

**Step 4 — LOG**
```
## [YYYY-MM-DD] INGEST | [source filename or folder]
**Source:** [full vault path]
**Pages updated:** [comma-separated list]
**Pages created:** [comma-separated list, or "none"]
**Key facts added:** [2–3 sentence summary]
```

**Step 5 — CONFIRM**  
"Ingest complete. Updated N pages, created M pages. [One interesting observation.]"

**BATCH INGEST VARIANT:** `INGEST BATCH [folder or date range]`  
Runs Steps 1–3 autonomously without checkpoints. Presents consolidated summary
at end. Logs each source individually, then adds one summary entry:
```
## [YYYY-MM-DD] INGEST-BATCH | [scope]
**Sources processed:** N
**Date range:** YYYY-MM-DD to YYYY-MM-DD
**Total pages updated:** N | **Total pages created:** N
**Top themes:** [3–5 bullet points]
```

---

### SKILL: QUERY

**Invocation:** `QUERY [question]` or `QUERY [question] AS [format]`  
**Purpose:** Answer a question from the wiki. File valuable answers back in.

**PROCEDURE:**

**Step 1 — SEARCH THE WIKI**  
Read `wiki/index.md`. Scan summaries to find relevant pages. Read them in full.
If wiki coverage is insufficient, supplement from raw vault sources.
Note any gaps for the LINT log.

**Step 2 — SYNTHESIZE THE ANSWER**  
Default format is structured markdown. Supported formats with `AS [format]`:

| Format | Output |
|--------|--------|
| `AS TABLE` | Comparison table in markdown |
| `AS MARP` | Slide deck (Marp: `---` breaks, `#` title, `##` content) |
| `AS TIMELINE` | Chronological markdown list with dates |
| `AS CHART` | Python/matplotlib code block with all data inline |
| `AS CANVAS` | Obsidian canvas JSON for concept maps |
| `AS PAGE` | Full wiki-style markdown page ready to save |

Always end with:
```
**Sources:** [[wiki/page]], [raw file](vault/path)
```

If raw sources were needed: "Note: This answer drew from [sources] not yet
in the wiki. Consider `INGEST [path]` to preserve this knowledge."

**Step 3 — FILE VALUABLE ANSWERS** *(interactive checkpoint)*  
Ask: "Should I file this into the wiki?"

If yes (or owner said `QUERY ... FILE IT` upfront):
a) Determine location: existing page to append to, or new page under `wiki/queries/`.  
b) Write or append with standard frontmatter.  
c) Update `wiki/index.md`.  
d) Log:
```
## [YYYY-MM-DD] QUERY | [truncated question ≤50 chars]
**Question:** [full question]
**Format:** [format used]
**Filed:** [path, or "not filed"]
**Sources consulted:** [wiki pages and raw files read]
**Gap noted:** [missing coverage, or "none"]
```

**QUERY CHAINING:** Follow-ups ("drill into [X]", "compare [X] to [Y]") are
treated as continuation. Append to the same filed page. Log as one entry.

---

### SKILL: LINT

**Invocation:** `LINT` or `LINT [scope]`  
**Purpose:** Health-check the wiki. Find problems, suggest fixes, surface gaps.

**PROCEDURE:**

**Step 1 — READ THE FULL WIKI**  
Read `wiki/index.md` and all pages listed in it.  
Read all `wiki/log.md` entries (`grep "^## \["` first to get the list).

**Step 2 — CHECK FOR STRUCTURAL PROBLEMS**

| Check | Description |
|-------|-------------|
| **Stale pages** | `last_updated` > 30 days, likely changed. Flag stale-and-risky vs. stale-but-stable. |
| **Orphan pages** | No inbound links from other wiki pages. Suggest which pages should link to them. |
| **Missing pages** | Concepts/people/projects mentioned 3+ times with no dedicated page. |
| **Contradictions** | Conflicting facts between pages. Cite both; recommend resolution. |
| **Superseded claims** | Facts likely outdated per newer log entries. |
| **Missing cross-references** | Page A discusses topic X; Page B is canonical for X; A doesn't link B. |
| **Raw source gaps** | Granola/ date folders or journal ranges with no log ingest entry. |

**Step 3 — SUGGEST INVESTIGATIONS**

- **New queries:** "The wiki mentions [topic] frequently. Suggested: `QUERY [question]`"
- **New ingests:** List unprocessed raw vault folders, sorted by value.
- **Web search candidates:** Facts that external context could enrich or verify.

**Step 4 — PRESENT FINDINGS**

Organized by severity:
- 🔴 **HIGH** — Contradictions, superseded active decisions, missing pages for current projects/people.
- 🟡 **MEDIUM** — Stale active pages, orphan pages, unprocessed recent sources.
- 🟢 **LOW** — Missing cross-references, old source gaps, style issues.

End with recommended action list (top 5).

**Step 5 — LOG**
```
## [YYYY-MM-DD] LINT | full
**Issues found:** N high | M medium | K low
**Stale pages:** [count and names]
**Orphan pages:** [count and names]
**Missing pages:** [top candidates]
**Unprocessed source ranges:** [list]
**Top 3 recommended actions:**
  1. [action]
  2. [action]
  3. [action]
**Suggested next INGEST:** [source]
**Suggested next QUERY:** [question]
```
