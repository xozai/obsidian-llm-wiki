# Deployment Guide

How to deploy the LLM wiki to an Obsidian vault for the first time.

---

## Prerequisites

### 1. Obsidian vault

You need an existing Obsidian vault with some content. The init prompt will
explore your vault structure and adapt to what it finds — it does not require
any specific folder layout. The more content in your vault, the richer the
initial wiki will be.

### 2. Claude with Obsidian MCP access

The LLM must be able to read and write files in your vault. Two supported setups:

**Option A — Claude.ai with MCP (recommended)**

1. Open [claude.ai](https://claude.ai) and go to **Settings → Integrations**.
2. Add the **Obsidian** MCP integration and point it at your vault folder.
3. Verify the integration is active — you should see Obsidian listed under
   connected integrations in a new conversation.

**Option B — Claude Code CLI with Obsidian MCP**

1. Install Claude Code: `npm install -g @anthropic-ai/claude-code`
2. Configure the Obsidian MCP server in your Claude Code settings.
   The server must expose at minimum these tools:
   - `obsidian_list_files_in_vault`
   - `obsidian_list_files_in_dir`
   - `obsidian_get_file_contents`
   - `obsidian_simple_search`
   - `obsidian_append_content`
   - `obsidian_patch_content`
3. Start a session in your vault directory: `claude` (from the vault root or anywhere)

> **Important:** The Obsidian MCP server must have **read and write** access to
> your vault. Read-only access will allow Phase 1 (exploration) but fail at
> Phase 3 (writing wiki/ files).

---

## Customizing the prompt for your vault

`INIT_PROMPT.md` contains a hardcoded vault structure under **PHASE 1**. Before
running it, update that section to match your actual vault layout.

Open `INIT_PROMPT.md` and replace the folder list and descriptions under
**PHASE 1 — EXPLORE** with your own folders. For each folder, describe:
- What kind of content it holds
- Whether it is a raw source, a template, or something to skip

The rest of the prompt (PHASE 2, 3, 4 and the SKILLS) is vault-agnostic and
does not need editing.

**Minimal vault example** (if you have a simple vault):
```
  Notes/          — Freeform notes and ideas
  Journal/        — Daily journal entries
  Projects/       — Project notes
  Resources/      — Saved articles and references
```

**If you don't know your vault structure** — skip editing PHASE 1 entirely
and instead prepend this line to the prompt before pasting:

```
Before starting Phase 1, call obsidian_list_files_in_vault to discover the
vault structure, then proceed with the exploration using what you find.
```

---

## First-time initialization (step by step)

### Step 1 — Open a new conversation

Start a fresh conversation in Claude.ai or a new Claude Code session.
Do not use an existing conversation — the initialization is long and you want
a clean context.

Confirm the Obsidian MCP integration is active. In Claude.ai you can verify
by asking: *"List the files in my Obsidian vault root"* before pasting the
prompt. You should see your vault folders listed.

### Step 2 — Paste the initialization prompt

Copy the full contents of `INIT_PROMPT.md` and paste it as your first message.

The LLM will begin **Phase 1** immediately — it will start listing and reading
files across your vault. This is the longest phase. Depending on vault size,
expect 10–30 tool calls before it finishes exploring.

**Do not interrupt Phase 1.** Let it complete the full survey before
responding. If it asks you a question mid-exploration, answer briefly and
tell it to continue.

### Step 3 — Review the Phase 2 synthesis

After exploring, the LLM will pause and present its understanding of your
vault before writing anything. Read this carefully:

- Does it have the right picture of who you are and what this vault is for?
- Are the active projects correct?
- Are the key people right?

If something is wrong, correct it now. Say: *"Actually, [correction]. Continue
with Phase 3."*

This is the most important checkpoint — the wiki's initial shape comes from
this synthesis.

### Step 4 — Let Phase 3 run

The LLM will now create the `wiki/` folder and all infrastructure files:

```
wiki/
├── SCHEMA.md           ← governing document for all future sessions
├── index.md            ← content catalog
├── log.md              ← operation log (first entry: BOOTSTRAP)
├── work/
│   ├── overview.md
│   ├── projects.md
│   └── areas.md
├── people/
│   └── work-contacts.md
├── meetings/
│   └── themes.md
└── personal/
    └── overview.md
```

This phase involves many file writes. It will take a few minutes.
You will see individual file creation/update calls as it works.

### Step 5 — Read the Phase 4 report

The LLM finishes with a report covering:
- Vault snapshot (counts, date ranges, projects found)
- Every wiki file created
- Key findings about your vault
- Gaps identified
- Recommended first INGEST, QUERY, and LINT

Read this report. It tells you what the LLM understands about your vault
and what to do next. The **Recommended First INGEST** is your immediate
next action.

### Step 6 — Verify the wiki was created

In Obsidian, navigate to your vault root. You should see a new `wiki/` folder.
Open `wiki/index.md` — it should have a populated table of pages.
Open `wiki/log.md` — it should have a single `BOOTSTRAP` entry.

If `wiki/` does not appear in Obsidian, check:
- That the MCP server has write access to your vault path
- That Obsidian has refreshed (close and reopen the vault, or use
  the "Reload vault" command)

---

## First working session (after initialization)

End the initialization conversation. Start a **new conversation**.

Paste this as your opening message every time you start a session:

```
Read wiki/SCHEMA.md, wiki/index.md, and the last 10 entries in wiki/log.md
to orient yourself. Then tell me what was done recently and what you
recommend doing next.
```

The LLM will read those three files and give you a status update. From there,
use the three skills:

```
INGEST Granola/2026-04-23
QUERY What are my open blockers across all active projects? AS TABLE
LINT
```

---

## Troubleshooting

**"I can't find the wiki/ folder in Obsidian"**
The Obsidian app sometimes caches the file tree. Press `Ctrl/Cmd+R` to reload
the vault, or close and reopen Obsidian.

**"The LLM stopped mid-initialization"**
Context limits can cut off very long initialization runs on large vaults.
If this happens, start a new conversation and paste this recovery prompt:

```
The wiki initialization for my Obsidian vault was interrupted. The wiki/
folder may be partially created. Please:
1. Read wiki/SCHEMA.md if it exists.
2. Read wiki/index.md if it exists.
3. Read wiki/log.md if it exists.
4. List the contents of wiki/ to see what was completed.
5. Resume from where it stopped, completing any missing files.
   Required files: wiki/SCHEMA.md, wiki/index.md, wiki/log.md,
   and the initial pages listed in INIT_PROMPT.md Phase 3.
```

**"Phase 1 is reading too many files and running out of context"**
Add this constraint before pasting the prompt:
```
In Phase 1, sample a maximum of 2 files per folder (not 3). For journals/,
read only 2 recent and 2 older entries. Keep Phase 1 exploration concise.
```

**"The wiki folder map in SCHEMA.md doesn't match my vault"**
Edit `wiki/SCHEMA.md` directly in Obsidian to correct the folder map.
The SCHEMA.md is a living document — you can edit it any time to update
conventions, add folders, or adjust skill behavior.

**"I want to start over"**
Delete the `wiki/` folder from your vault and run the initialization again
from a fresh conversation.

---

## Ongoing maintenance

| How often | What to do |
|-----------|------------|
| Daily | `INGEST` yesterday's journal entry or any new meeting transcripts |
| Weekly | `INGEST BATCH` the week's Granola folder |
| Monthly | `LINT` to health-check the wiki |
| As needed | `QUERY` anything you want to know or analyze |
| When the wiki grows large | Add subfolders to `wiki/work/`, `wiki/people/` etc. and update `wiki/SCHEMA.md` |

The wiki compounds. After 30 ingests it starts to feel like a real knowledge
base. After 100 it starts surfacing connections you wouldn't have noticed.
