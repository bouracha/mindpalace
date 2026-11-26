# MindPalace

> Last updated: —

Your knowledge operating system. Talk naturally; I organise as patterns emerge.

## How It Works

Everything lives in folders with READMEs. I create structure from our conversations.

- **You talk**: I listen and engage
- **Patterns emerge**: I create folders (people, projects, topics)
- **Updates run**: when you run `/update`, I fold every session since the last run into the READMEs and write a Chronicle
- **You return**: I reconstruct context by reading the indexes
- **Reasons are kept** (optional): a meaningful change can get a line in an `evolution-crystals/` changelog
- **Lessons are kept** (optional): portable lessons can go to the crystal library

No setup required. Just start talking.

## Structure

```
MindPalace/
├── README.md           ← you are here (read on every request)
├── .claude/            : the OS kernel (protected)
├── sessions/           : logs, summaries, chronicles
├── crystal-library/    : portable lessons, one claim per file (example)
├── evolution-crystals/ : why the palace itself changed (example)
├── me/                 : your profile (created on first run)
└── [emergent...]       : people/, projects/, topics/ appear as needed
```

## Commands

- `/update`: process the sessions since the last run, generate summary + chronicle (works well as a nightly job, if you want one)

## Quick Navigation

| Keyword | Location | Description |
|---------|----------|-------------|
| me, profile | [[me/]] | Your profile |
| sessions, logs | [[sessions/]] | Session logs and summaries |
| crystals, lessons, principles | [[crystal-library/]] | Portable lessons, one claim per file |
| why, history, changelog | [[evolution-crystals/]] | Why the palace itself changed |

<!-- A suggestion that has worked well: one row per topic (keywords, a path, one sentence), with detail in the target README, which is read only when the topic comes up. This file is read on every request, so its size is a cost paid every time. -->

## Current Structure

- [[me/]]: your profile (created on first run)
- [[sessions/]]: logs, summaries, chronicles
- [[crystal-library/]]: portable lessons
- [[evolution-crystals/]]: the palace's own changelog and crystals

<!-- Emergent folders appear here as they're created -->
<!-- - [[people/]]: contacts and collaborations -->
<!-- - [[projects/]]: active work -->
<!-- - [[topics/]]: reference material -->

## Keyword Index

<!-- Semantic routing: add keywords as structure grows -->
```
me: me/
profile: me/
sessions: sessions/
logs: sessions/
crystals: crystal-library/
lessons: crystal-library/
why: evolution-crystals/
changelog: evolution-crystals/
```

---

## Navigation

Every folder has a README. Follow the READMEs.
