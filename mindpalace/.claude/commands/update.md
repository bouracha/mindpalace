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

## Step 0: Ask Update Scope

Before processing, ask the user:

> **What should I update from?**
> 1. **Everything since last update** — Process all sessions since the cursor timestamp
> 2. **Just today** — Only process sessions from today

Use the AskUserQuestion tool with these options.

Based on their choice:
- **Everything since last update**: Use the cursor from `sessions/README.md` as normal
- **Just today**: Override the cursor to today's date at 00:00 (ignoring the stored cursor)

---

## Scope Discipline (CRITICAL — read before Step 1)

**The cursor is authoritative at the message level, not the file level.**

Long-running session files keep being appended to across multiple /update runs. A file with an mtime after the cursor doesn't mean the *whole file* is new — only the lines with timestamps after the cursor are new. Treating the whole file as new is how scope-bleed happens (May 2026 incident: pre-cursor content from sessions started in late April got re-ingested into the May 7–13 chronicle because their files were still being written to).

**Two-layer defense:**

1. **Filter by per-message timestamp during extraction** (primary). Every JSONL line has a `timestamp` field — skip any line older than the cursor.
2. **Cross-reference the previous summary** (secondary). Before writing the new summary, read the most recent `sessions/summaries/*.md` and check whether you're about to re-describe things that are already there. If yes, that's a scope-bleed signal — diagnose before continuing.

Naming this failure mode explicitly so it doesn't happen quietly: **session files outlive cursors**. Always slice content by message timestamp, never by file boundary.

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
Last processed: 2026-05-13T09:00:00Z
```

Format: a single ISO 8601 UTC timestamp. No filename — filename is ambiguous when many files were touched, and the cursor is about *content*, not files.

If the existing cursor is in the old `{filename} @ {YYYY-MM-DD HH:MM}` format, convert it to UTC ISO 8601 on this run and store the new form going forward. Interpret the old local-time stamp as the user's local timezone (run `date '+%Z'` to confirm).

### Get Current Time

```bash
date -u '+%Y-%m-%dT%H:%M:%SZ'
```

UTC ISO 8601. Always — JSONL message timestamps are UTC, and the cursor needs to compare apples to apples. Never use env info.

### Find Candidate Files

List JSONL files (including subagents) with mtime **after** the cursor — this is a *candidate list*, not the final scope.

```bash
find ~/.claude/projects/{project-path}/ -name "*.jsonl" -newermt "$CURSOR_TS"
```

This catches both top-level session files and subagent files in subdirectories.

If no candidate files, report "Nothing to update" and stop.

### Slice Candidate Files by Message Timestamp

**For each candidate file, extract only the messages with `timestamp > cursor`.** Do not pass the whole file to the extract pass. Quick Python sketch:

```python
import json
cutoff = "2026-05-13T09:00:00Z"  # cursor
for line in open(path):
    try:
        obj = json.loads(line)
    except json.JSONDecodeError:
        continue
    ts = obj.get('timestamp')
    if ts and ts > cutoff:
        # in-scope — feed to extract pass
        ...
```

Lines without timestamps (rare — usually structural records like session start markers) can be kept or dropped at your discretion; default to keeping them only if surrounded by in-scope messages.

**Sanity check before extraction**: count in-scope messages per file. If a file contributes 0 in-scope messages, drop it entirely from this run even if its mtime is newer than the cursor.

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

Write to `sessions/summaries/YYYY-MM-DD.md` (or `YYYY-MM-DD-to-YYYY-MM-DD.md` for multi-day windows):

- Clear markdown, structure follows content
- Capture what happened, decisions made, things learned
- Include references to folders created/updated
- Keep it parseable for AI context loading
- **State the window explicitly** in a header line: "Strict window: post-cursor (YYYY-MM-DDTHH:MM:SSZ). Earlier session content is in `sessions/summaries/<prev>.md`."

**Scope sanity check before writing.** Read the most recent prior summary in `sessions/summaries/`. For each major item you're about to include, ask: "Was this already covered there?" If yes, drop it from this summary. Reference the prior summary instead. This is the second layer of defense against scope-bleed; the first (message-timestamp filtering in Step 1) should catch most of it, but a final check before committing the summary file is cheap and high-value.

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

Update `sessions/README.md` with a UTC ISO 8601 timestamp. Use the *current time* at update completion — not the newest in-scope message timestamp, because subsequent /update runs should pick up anything written while this one was processing.

```markdown
Last processed: 2026-05-13T10:00:00Z
```

No filename. The cursor is about *content* (messages with timestamps), not about files (which keep getting appended to).

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
