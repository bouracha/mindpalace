# Sessions

> Last updated: —

Session logs, summaries, and chronicles.

## Cursor

```
Last processed: (none)
```

Format: `{filename} @ {timestamp}` or `(none)` for fresh start.

## Structure

```
sessions/
├── README.md       ← you are here (cursor tracking)
├── summaries/      — structured summaries (for AI context)
├── chronicles/     — fun newspapers (for humans, PDF)
└── unrouted.md     — items pending routing decision
```

## Context Reconstruction

To get context for topic Y:
1. Check Y's README → find timestamp
2. Load `summaries/` for that date
3. Find Y section → full context
