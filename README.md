# obsidian-llm-wiki

A prompt-based system for maintaining a living knowledge wiki inside an [Obsidian](https://obsidian.md) vault, using the [Karpathy LLM wiki method](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

## What this is

Instead of asking an LLM the same questions repeatedly, you maintain a persistent `wiki/` folder inside your vault. The LLM reads raw sources (meeting transcripts, journal entries, clippings), synthesizes them into structured markdown pages, and keeps the wiki current over time. Knowledge compounds — each new source enriches dozens of pages, and valuable query answers get filed back in rather than disappearing into chat history.

## Getting started

See **[DEPLOY.md](DEPLOY.md)** for the full setup guide, including:
- Prerequisites (Claude with Obsidian MCP)
- How to customize the prompt for your vault
- Step-by-step first-time initialization
- Troubleshooting common failures
- Ongoing maintenance cadence

**Quick version:**
1. Confirm the Obsidian MCP integration is active in your Claude session.
2. Edit the PHASE 1 folder list in `INIT_PROMPT.md` to match your vault layout.
3. Paste `INIT_PROMPT.md` into a fresh Claude conversation.
4. Wait for the Phase 4 report — then start a new session and use the resume prompt below.

## How it works

The system has three components:

| File | Purpose |
|------|---------|
| `INIT_PROMPT.md` | The initialization prompt. Run once to bootstrap the `wiki/` folder in a fresh vault. |
| `DEPLOY.md` | Step-by-step setup guide, customization instructions, and troubleshooting. |
| `SCHEMA_TEMPLATE.md` | Reference template for `wiki/SCHEMA.md` (the LLM writes the actual file during init). |
| `wiki/SCHEMA.md` | Written by the LLM during init. Governs all future sessions — folder map, conventions, and the three skills. |
| `wiki/index.md` | Content catalog. One row per wiki page. The LLM reads this first on every query. |
| `wiki/log.md` | Append-only operation log with grep-parseable headings. |

## The three skills

Once initialized, you drive the wiki with three commands:

**`INGEST [source path]`** — Process a new source into the wiki. The LLM reads it, discusses key takeaways with you, updates 10–15 wiki pages, and logs the operation. Run `INGEST BATCH [folder]` for unattended catch-up on a backlog.

**`QUERY [question] AS [format]`** — Answer a question from the wiki. Supported formats: `TABLE`, `MARP` (slides), `TIMELINE`, `CHART` (matplotlib), `CANVAS` (Obsidian), `PAGE`. Valuable answers can be filed back into the wiki with `FILE IT`.

**`LINT`** — Health-check the wiki. Finds contradictions, stale pages, orphan pages, missing cross-references, and unprocessed raw sources. Returns a severity-tiered report (🔴 high / 🟡 medium / 🟢 low) with recommended next actions.

## Resuming a session

After initialization, start every new session with this prompt:

```
Read wiki/SCHEMA.md, wiki/index.md, and the last 10 entries in wiki/log.md
to orient yourself. Then tell me what was done recently and what you recommend next.
```

The three files together contain everything the LLM needs — no memory of prior
sessions required. The LLM will read them, summarize recent activity from the log,
and suggest what to do next.

## Vault structure assumed

This prompt was built for a vault with the following layout, but is easily adapted:

```
Areas/          Granola/        People/         Templates/
Claude Analysis/ Inbox/          Projects/       assets/
Clippings/      Limitless Lifelogs/ Resources/  daily briefing/
End of Day Reviews/ Meetings/   Supernote/      end of day briefing/
Excalidraw/     News Briefings/ journals/
```

Key integrations in use: Granola (meeting transcripts), Todoist, Limitless (lifelogs), Supernote, Dataview.

## Log parseability

The `wiki/log.md` file uses a consistent heading format for unix tooling:

```bash
grep "^## \[" wiki/log.md              # all entries
grep "^## \[" wiki/log.md | tail -10   # last 10 entries
grep "INGEST" wiki/log.md              # all ingests
grep "2026-04" wiki/log.md             # all April 2026 activity
```

## Reference

- [Karpathy's original gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Obsidian](https://obsidian.md)
