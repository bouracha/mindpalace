# /update

Process session logs and update MindPalace.

---

## Overview

```
Raw JSONL logs → Chunk → Extract → Merge → Route → Generate outputs
```

Produces:
- **Summaries** (`sessions/summaries/`) — for AI context
- **Chronicles** (`sessions/chronicles/`) — for human enjoyment

---

## Step 1: Find New Sessions

### Locate Transcripts

Claude CLI stores transcripts at:
```
~/.claude/projects/{project-path}/*.jsonl
```
Where `{project-path}` = working directory with `/` → `-` and leading `-`.
Example: `/home/user/MindPalace` → `-home-user-MindPalace`

### Check Cursor

Read cursor from `sessions/README.md`:
```markdown
Last processed: 4ed9573c-31d8-4e6f-ac10-c6e8284de5c8.jsonl @ 2025-01-19 17:48
```

Format: `{filename} @ {timestamp}`

### Find New Files

List JSONL files with mtime **after** the cursor timestamp.
```bash
find ~/.claude/projects/{project-path}/ -name "*.jsonl" -newermt "2025-01-19 17:48"
```

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

### Example
```
File: 1200 lines
compact_boundary at lines: 400, 850

Chunk 1: lines 1-410 (overlap)
Chunk 2: lines 400-860 (overlap)
Chunk 3: lines 850-1200
```

> **Future optimization**: This chunking logic should be a deterministic script, not LLM inference.

---

## Step 3: Pass 1 — Extract (per chunk)

For each chunk, **extract what's relevant**. Structure follows content — don't force categories.

Things you might extract (depending on what's there):
- People mentioned and context
- Projects discussed, progress, decisions
- Topics, themes, concepts
- Insights, realizations, learnings
- Actions, follow-ups, commitments

**Extract liberally** — don't filter yet. Capture everything that might matter.

The output format is flexible. Example of what an extraction might look like:

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

Another chunk might look completely different if the conversation was different. That's fine.

---

## Step 4: Pass 2 — Merge + Re-evaluate

Combine extractions across all chunks:

1. **Dedupe**: John in chunk 1 = John in chunk 3
2. **Aggregate context**: Merge all mentions of same entity
3. **Re-score importance**: Something minor in chunk 1 may be significant given chunk 4
4. **Flag cross-cutting**: Items that span multiple projects/topics

Output: Unified extraction list with aggregated context.

---

## Step 5: Pass 3 — Route + Write + Cascade

For each extracted entity:

### Routing Decision
```
Does folder exist? (e.g., people/john/)
    │
    ├── YES → Append to folder, update README
    │
    └── NO → Is it significant?
            │
            ├── YES (multiple mentions, explicit naming, has actions)
            │   → Create folder + README
            │
            └── NO / UNCERTAIN
                → Add to sessions/unrouted.md
```

### Writing
- Append new information to relevant files
- Include timestamps and session references
- Use `[[wiki-links]]` for cross-references

### Cascading READMEs (Bottom-Up)

Updates flow from leaves to root — each README summarizes its children:

```
projects/chess/experiments/README.md   ← update first (leaf)
        ↓
projects/chess/README.md               ← summarizes experiments/
        ↓
projects/README.md                     ← summarizes chess/
        ↓
README.md (root)                       ← summarizes projects/, plus navigation map
```

**Process:**
1. Collect all modified folders
2. Compute all ancestor folders up to root
3. Sort by depth (deepest first)
4. For each folder, update its README:
   - Update timestamp: `> Last updated: YYYY-MM-DD HH:MM`
   - Summarize what's in child folders/files
   - Add `[[wiki-links]]` to children
5. Continue up to root

### Root README: The Entry Point (Richer Than Normal Parents)

Normal parents summarize children abstractly. Root is different — it's the **entry point for all navigation**, so it should be the **richest** layer:

- Keywords bubble up from deep in the tree, not just immediate children
- More detail than a normal parent would have
- Cross-cutting keywords that span multiple folders

Think of root as the routing table / search index, not just another summary.

**Root does standard cascade work, plus:**

1. **Quick Navigation table** — Add keywords that help find content:
   ```markdown
   | chess, engine, benchmark | [[projects/chess/]] | Chess engine research |
   ```

2. **Current Structure** — Keep the folder tree current:
   ```markdown
   - [[projects/chess/]] — Chess engine research
   ```

3. **Keyword Index** — Add semantic routing entries:
   ```
   chess: projects/chess/
   benchmark: projects/chess/
   ```

**Example: Keywords bubbling up**

```
projects/chess/experiments/alpha-beta.md  — contains "pruning optimization"

Normal parent (projects/chess/README.md):
  - [[experiments/]] — Algorithm experiments

Root README (richer):
  | chess, alpha-beta, pruning | [[projects/chess/]] | Chess engine research |

  Keyword Index:
  pruning: projects/chess/experiments/
  alpha-beta: projects/chess/experiments/
```

The root captures "pruning" and "alpha-beta" even though they live 3 levels deep. A user searching for "pruning" hits the root keyword index immediately.

**Use judgment**: Not every detail needs a root keyword. Focus on things users will actually search for — project names, key concepts, people, recurring themes.

**Techniques are suggestions, not requirements**: If you discover better ways to make content findable (tags, aliases, cross-links, whatever), use them. The goal is navigation — the technique is yours to choose.

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

**Always generate LaTeX → PDF**. No markdown chronicle (the summary already covers that).

Write to `sessions/chronicles/YYYY-MM-DD.tex`, then compile to PDF.

**Template**: `.claude/templates/chronicle.tex`

**Sections** (populate from extracted data):

| Section | Content |
|---------|---------|
| HEADLINES | Grabby, fun headlines for major updates |
| PEOPLE IN THE NEWS | Profile updates, new contacts |
| PROJECT UPDATES | What changed, decisions made |
| NEW DISCOVERIES | New folders, topics, insights |
| BY THE NUMBERS | Sessions processed, files updated, folders created |
| MINDPALACE WEATHER | Fun health check (e.g., "Sunny — 0 unrouted items!") |
| QUESTIONS FOR THE READER | From unrouted.md, 5-10 max, auto-archive after 7 days |

**Make it fun**: Headlines, wordplay, personality. This is a newspaper, not a log.

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

Example:
```markdown
Last processed: 4ed9573c-31d8-4e6f-ac10-c6e8284de5c8.jsonl @ 2025-01-19 20:30
```

---

## Triggering

- **Manual**: User runs `/update`
- **Nightly**: Via SessionEnd hook or cron (if configured)

---

## Permissions

This command runs autonomously. Settings allow Write/Edit broadly, deny only `.claude/**`:
```json
{
  "permissions": {
    "allow": ["Read", "Write", "Edit", "Bash", ...],
    "deny": ["Write(.claude/**)", "Edit(.claude/**)", ...]
  }
}
```

---

## Error Handling

- **No new sessions**: Report "Nothing to update", stop gracefully
- **Chunk too large**: Sub-split further, warn in summary
- **Routing uncertain**: Add to unrouted.md, continue
- **LaTeX fails**: Keep markdown chronicle, skip PDF, warn user
