# /update

Process session logs and update MindPalace.

---

## Overview

```
Raw JSONL logs → Chunk → Extract → Merge → Route → Generate outputs
```

Produces:
- **Summaries** (`sessions/summaries/`) — for AI context
- **Chronicles** (`sessions/chronicles/`) — for human enjoyment + evaluation

---

## Step 1: Find New Sessions

### Locate Transcripts

Claude CLI stores transcripts at:
```
~/.claude/projects/{project-path}/*.jsonl
~/.claude/projects/{project-path}/*/subagents/*.jsonl
```
Where `{project-path}` = working directory with `/` → `-` and leading `-`.
Example: `/home/user/MindPalace` → `-home-user-MindPalace`

**Include subagent files.** Subagents are spawned during sessions for exploration, research, and parallel work. Their JSONL logs live in `{session-id}/subagents/agent-*.jsonl`. These may contain low-signal tool output, but they may also contain substantive research, discoveries, or decisions that the parent session didn't surface. **Process them all** — extract what's substantive, skip pure tool noise. Attribute subagent content to its parent session (the UUID directory name).

### Check Cursor

Read cursor from `sessions/README.md`:
```markdown
Last processed: 4ed9573c-31d8-4e6f-ac10-c6e8284de5c8.jsonl @ 2025-01-19 17:48
```

Format: `{filename} @ {timestamp}`

### Get Current Time

```bash
date '+%Y-%m-%d %H:%M'
```

Always use this for timestamps — never rely on env info.

### Find New Files

List JSONL files (including subagents) with mtime **after** the cursor timestamp.
```bash
find ~/.claude/projects/{project-path}/ -name "*.jsonl" -newermt "2025-01-19 17:48"
```
This catches both top-level session files and subagent files in subdirectories.

If no new sessions, report "Nothing to update" and stop.

---

## Step 2: Chunk Sessions

**Important**: The Read tool has a 25K token limit. Large JSONL files must be chunked.

### Chunking Process

For each session JSONL:

1. **Get file stats**:
   ```bash
   wc -l ~/.claude/projects/{project}/{file}.jsonl
   ```

2. **Find natural split points** (compact_boundary markers):
   ```bash
   grep -n "compact_boundary" ~/.claude/projects/{project}/{file}.jsonl
   ```

3. **Read in chunks** using Read tool's `offset` and `limit` parameters:
   - If file < 500 lines: read whole file
   - If file has compact_boundary markers: split at those line numbers
   - Otherwise: split into ~300 line chunks with 10-line overlap

4. **Process each chunk** through Pass 1 (Extract)

---

## Step 3: Pass 1 — Extract (per chunk)

For each chunk, **extract what's relevant**. Structure follows content — don't force categories.

Things you might extract (depending on what's there):
- People mentioned and context
- Projects discussed, progress, decisions
- Topics, themes, concepts
- Insights, realizations, learnings
- Actions, follow-ups, commitments
- Dates and deadlines

**Extract liberally** — don't filter yet. Capture everything that might matter.

The output format is flexible. Example:

```yaml
chunk: session-abc123-chunk-2

John:
  - discussed grant collaboration
  - mentioned Sarah in grants office

chess project:
  - alpha-beta pruning breakthrough
  - decided on iterative deepening

insight: "Chronicles for humans, summaries for AI"

action: "Follow up with John by Jan 25"
```

---

## Step 4: Pass 2 — Merge + Re-evaluate (Map-Reduce)

**Scales to any volume.** Never try to merge everything in one context window.

### Cluster by entity
Group all extractions by entity (person, project, topic):
```
All chunk extractions
    ↓
Group: "John" mentions from chunks 1, 3, 7
Group: "chess project" mentions from chunks 2, 3, 5
Group: "grant deadline" from chunk 4
...
```

### Merge within each cluster
For each cluster (one context window per cluster):
1. **Dedupe**: Same info from overlapping chunks
2. **Aggregate context**: Combine all mentions into coherent summary
3. **Re-score importance**: Something minor in chunk 1 may be significant given chunk 4
4. **Note co-occurrences**: "John appeared with chess project and grants"

### If clusters are too large
Recurse — merge cluster summaries into higher-level summaries. Each step is bounded.

### Output
One clean merged extraction per entity, with:
- Aggregated context
- Importance signal (how often, how central)
- Co-occurrence map (which entities appeared together)

---

## Step 5: Pass 3 — Route + Write + Cascade

**Three sub-phases, each bounded per-file. No step holds everything in context.**

### Phase 3a: Route content (per entity, one at a time)

For each merged entity:

```
Does folder exist? (e.g., people/john/)
    │
    ├── YES → Read its README, append new info, save
    │
    └── NO → Is it significant?
            │
            ├── YES (multiple mentions, explicit naming, has actions)
            │   → Create folder + README with content
            │
            └── NO / UNCERTAIN
                → Add to sessions/unrouted.md
```

- Process ONE entity per context window
- Include timestamps (via `date` bash) and session references
- Track which files were modified: `modified_files[]`

### Phase 3b: Cross-link sweep (after all routing done)

For each pair of entities that co-occurred in sessions:
1. Check if `[[wiki-links]]` exist between their READMEs
2. Add missing links where relevant
3. Each link check reads TWO files — bounded and fast

This is a separate pass because you need all entities routed before you can link between them.

### Phase 3c: Cascade timestamps (bottom-up, lightweight)

```
people/john/README.md        ← modified in 3a
        ↓
people/README.md             ← update timestamp + child summary
        ↓
README.md (root)             ← update navigation map + keyword index
```

**Process:**
1. Collect all `modified_files[]` from 3a + 3b
2. Compute all ancestor folders up to root
3. Sort by depth (deepest first)
4. For each folder (one at a time):
   - Read its README + first lines of children
   - Update timestamp: `> Last updated: YYYY-MM-DD HH:MM`
   - Update child summary lines
   - Add `[[wiki-links]]` to children
5. Continue up to root

Each cascade step is lightweight — one parent README + child headers.

### Root README: Richer Than Normal Parents

Root is the **entry point for all navigation**, so it gets extra detail:

- Keywords bubble up from deep in the tree, not just immediate children
- Cross-cutting keywords that span multiple folders

**Root does standard cascade work, plus:**
1. **Quick Navigation table** — Add keywords that help find content
2. **Current Structure** — Keep the folder tree current
3. **Keyword Index** — Add semantic routing entries

---

## Step 6: Pass 4 — Generate Outputs

### 6a: Structured Summary

Write to `sessions/summaries/YYYY-MM-DD.md`:

- Clear markdown, structure follows content
- Capture what happened, decisions made, things learned
- Include references to folders created/updated
- Keep it parseable for AI context loading

**No rigid template** — let the summary reflect the session.

### 6b: Chronicle (The Newspaper PDF)

**Always generate LaTeX → PDF**. The summary already covers the structured data.

Write to `sessions/chronicles/YYYY-MM-DD.tex`, then compile to PDF.

**Template**: `.claude/templates/chronicle.tex`

**Sections** (populate from extracted data):

| Section | Content |
|---------|---------|
| HEADLINES | Grabby, fun headlines for major updates |
| PEOPLE IN THE NEWS | Profile updates, new contacts |
| PROJECT UPDATES | What changed, decisions made |
| NEW DISCOVERIES | New folders, topics, insights |
| BY THE NUMBERS | Metrics dashboard (see below) |
| MINDPALACE WEATHER | Fun health check (e.g., "Sunny — 0 unrouted items!") |
| QUESTIONS FOR THE READER | From unrouted.md, 5-10 max, auto-archive after 7 days |

**Write like a newspaper: sell AND inform.** The Chronicle is both salesman and informer.

**Structure each topic as a real article:**
- **Headline hooks you in** — grabby, fun, makes you want to read ("Partnership Takes Shape as Revenue Model Crystallizes")
- **The article delivers the full story** — all the nuance, specifics, and context that make it valuable

**Don't skimp on important detail.** Each article should include:
- **Specific names, dates, figures** — concrete details, not vague summaries
- **The narrative** — what happened, in what order, and why each step mattered
- **Decisions and reasoning** — what was decided and the logic behind it
- **Quotes** — direct quotes from sessions where they add colour or capture key moments
- **Strategic context** — why this matters, what it connects to, what it means for the bigger picture
- **Next steps** — what happens next, with dates if known

**The headline → nuance pipeline**: A headline is a promise. The article delivers on that promise with the detail needed to fully inform the reader. Someone reading in 6 months should understand exactly what happened, not just that something happened.

**Detail weighting.** Both effort and novelty matter. Use signals from the data to judge emphasis:
- **Effort**: Session length, message count, multi-session topics, iterative refinement cycles — if someone spent 5 hours building something, the chronicle should make that visible
- **Novelty**: New insights, unexpected connections, first-time events — these deserve prominence even if brief in the logs

How to weight these against each other is an evolving design choice. For now: **err heavily on the side of too much detail and long chronicles.** We can always trim later but can't recover what we didn't capture. Every topic gets a proper article. Better a 10-page chronicle that captures everything than a 2-page one that misses nuance.

**The Chronicle is also an evaluation tool** — it reveals capture quality, routing accuracy, and what the system is uncertain about. More detail = better evaluation.

**Chronicle length target**: Aim for 4-5 pages for a typical daily update. Can go up to ~20 pages for dense multi-day periods. Don't fix rigidly — give each nuance the explanation it needs.

### Metrics Dashboard ("By the Numbers")

Collect these metrics during processing and include in the chronicle's "By the Numbers" section. Add any metric that's easy to gather — we can narrow later.

**Input metrics** (gathered during Steps 1-2):
```bash
# Total JSONL lines processed
wc -l <all processed files> | tail -1

# Total bytes of raw logs
du -sh <all processed files>

# Number of sessions + subagent files
ls <processed files> | wc -l
```

**Processing metrics** (tracked during Steps 3-5):
- Chunks created (how many extraction passes)
- Entities extracted (people, projects, topics)
- Entities routed (to existing folders vs new folders)
- Items added to unrouted.md
- READMEs updated (count)
- Wiki-links added/verified
- New folders created

**Output metrics**:
- Summary file size
- Chronicle page count
- Questions for the reader (count)

Present in the chronicle as a compact dashboard — numbers tell the story of how much work the system did.

### LaTeX Layout Requirements

**CRITICAL**: These rules prevent layout bugs like empty pages or content running off page.

1. **Always load breakable library**:
   ```latex
   \usepackage{tcolorbox}
   \tcbuselibrary{breakable,skins}  % MUST be after usepackage
   ```

2. **All tcolorbox styles MUST include `breakable`**:
   ```latex
   \tcbset{
     featurebox/.style={
       colback=mpblue!5,
       ...
       breakable  % Without this, boxes won't flow across pages
     }
   }
   ```

3. **When creating custom boxes in the .tex file**, always add `breakable` to the options:
   ```latex
   \begin{tcolorbox}[featurebox, breakable, title={Feature Title}]
   ```

4. **Avoid rigid \columnbreak** where content length varies — let LaTeX flow naturally.

5. **No fixed heights** on boxes — they should expand with content.

The template at `.claude/templates/chronicle.tex` includes these settings. When writing custom LaTeX sections, follow the same patterns.

### PDF Compilation

```bash
cd sessions/chronicles && pdflatex -interaction=nonstopmode YYYY-MM-DD.tex
```

Clean up auxiliary files (`.aux`, `.log`) after successful compilation.

**If pdflatex unavailable**, offer to install:
- Ubuntu/Debian: `sudo apt-get install texlive-latex-base texlive-fonts-recommended texlive-latex-extra`
- macOS: `brew install basictex`
- Lightweight: `tectonic`

**If compilation fails**: Keep `.tex` file, warn user, continue.

---

## Step 7: Update Cursor

Update `sessions/README.md` with the newest processed file:
```markdown
Last processed: {filename} @ {YYYY-MM-DD HH:MM}
```

---

## Step 8: PDF Reflection & Validation

**MANDATORY BEFORE COMPLETING.** After PDF compilation, validate the output.

### 8a: Read the PDF

Use the Read tool to inspect the generated PDF:
```
Read sessions/chronicles/YYYY-MM-DD.pdf
```

This renders the PDF visually — check the actual layout, not just that compilation succeeded.

### 8b: Layout Validation Checklist

Verify each of these:

| Check | Issue if Failed | Fix |
|-------|-----------------|-----|
| **No empty pages** | Page 1 has masthead + headlines then mostly blank | Content boxes not breaking — verify `breakable` in all styles |
| **Content flows across pages** | Feature article runs off bottom of page | Box missing `breakable` option, or library not loaded |
| **No orphaned headers** | Section header at bottom with content on next page | Add `\needspace{3\baselineskip}` before header |
| **Readable column balance** | One column full, other mostly empty | Reduce `\columnbreak` usage, let LaTeX flow |
| **All sections present** | Missing expected content (people, projects, etc.) | Check LaTeX generation logic |

### 8c: If Issues Found

1. **Identify the cause** — usually missing `breakable` or improper box nesting
2. **Edit the .tex file** — fix the specific issue
3. **Recompile**: `pdflatex -interaction=nonstopmode YYYY-MM-DD.tex`
4. **Re-read the PDF** to verify fix
5. **Repeat until layout is clean**

### 8d: Report Layout Quality

In your completion message, include:
- Page count
- Whether any layout issues were found and fixed
- Overall layout quality assessment

**The update is not complete until the PDF passes validation.**

---

## Triggering

- **Manual**: User runs `/update`
- **Nightly**: Via SessionEnd hook or cron (if configured)

---

## Permissions

This command runs autonomously. Settings allow Write/Edit broadly:
```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Bash", ...],
    "deny": ["Write(.claude/CLAUDE.md)", "Write(.claude/settings.json)", ...]
  }
}
```

---

## Error Handling

- **No new sessions**: Report "Nothing to update", stop gracefully
- **Chunk too large**: Sub-split further, warn in summary
- **Routing uncertain**: Add to unrouted.md, continue
- **LaTeX fails**: Keep .tex file, skip PDF, warn user
