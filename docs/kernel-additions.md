# Kernel additions

The template's kernel (`mindpalace/.claude/CLAUDE.md`) is protected: the model is told never to edit it, and you can make it read-only on disk. If you want the kernel and the `/update` command to know about the mechanisms in this repository, the four optional blocks below are the wording I use. Paste in the ones that suit you, and change the wording, tags and formats to fit how you work. Each is short on purpose, because the kernel is read on every request.

Background: [update.md](update.md) · [crystallisation.md](crystallisation.md) · [evolution-crystals.md](evolution-crystals.md) · [provenance.md](provenance.md)

## 1. `.claude/CLAUDE.md`: after the section "Feedback Loop: Ask, Don't Guess"

```markdown
## Evolution Crystals: Record the Reasons

A meaningful change, in any area, gets a record of **what changed and why**.

- **Tier 1, most changes:** one line appended to the area's `evolution-crystals/changelog.md`, in the form `- YYYY-MM-DD HH:MM | what changed | why: one clause | ref: optional [tag]`
- **Tier 2, genuine evolution** (a concept coined or abandoned, a reframe, a decision with alternatives rejected): a dated file `evolution-crystals/YYYY-MM-DD-<slug>.md` with an `Originator:` field, covering the change, the reasoning, alternatives considered, the trail and open questions. It also gets a Tier-1 line. One thread per file.
- **Do not load them by default.** Open an area's changelog when asked why something is the way it is.
- **Capture live**, as changes happen. `/update` backfills what was missed.

---

## Provenance Tags

Tag each captured idea with whose it is: `[A]` the author (the human) originated it · `[J]` the agent proposed it, unvetted · `[J→A]` the agent proposed it and the human adopted it · `[A+J]` built together · `[J?]` the agent raised it with the human, who did not reply.

- An idea that first appears in an agent turn stays `[J]` until the human explicitly adopts it. Silence is not adoption, and repetition is not consent.
- Take the tag from the transcript (who said it first) rather than guessing it.
- Evolution crystals carry an `Originator:` field; crystals and changelog lines that record an idea carry a tag; summaries and the Chronicle carry the tags through.
- When reading back an earlier idea, say whose it was. Do not phrase a `[J]` proposal as something the human decided.

---

## Edit in Place

Update READMEs and documents in place rather than appending dated versions to a file or keeping superseded copies beside the live one. The history goes in the evolution crystals, and every extra copy makes each reader work out which one is current.

---

## The Always-Read Index

The root README is read on every request, so its size is a cost paid every time. Keep each index row to keywords, a path and one sentence. Detail goes in the target folder's README, which is read only when its topic comes up.

---

## The Crystal Library

When a lesson generalises beyond the case that produced it, offer to write it as a crystal in `crystal-library/`: the title is the claim; then the type (WHAT, WHY or HOW), a provenance tag, the crystal, why it matters and where it applies. Add one row to the library's index. Extend an existing crystal with a new instance rather than creating a near-duplicate.
```

## 2. `.claude/CLAUDE.md`: at the end of "Summary: Operating Principles"

```markdown
10. **Record the reasons**: evolution crystals per area, not loaded by default
11. **Tag provenance**: silence is not adoption
12. **Keep the root index short**: keywords, a path, one sentence
```

## 3. `.claude/commands/update.md`: at the end of Step 5, before Step 6

```markdown
### Phase 3d: Evolution crystals and crystallisation

Using the same extractions:

1. **Carry provenance.** Take each idea's tag from the transcript (who said it first) into the changelog, the summary and the Chronicle.
2. **Backfill changelog lines.** For each change made in the processed sessions that has no line in its area's `evolution-crystals/changelog.md`, append one (create the folder on first use). Tag provenance where the line records an idea.
3. **Promote genuine evolution.** Where the thinking itself moved (a concept coined or abandoned, a reframe, a decision with alternatives rejected), write a full crystal `evolution-crystals/YYYY-MM-DD-<slug>.md`, one thread per file, and point a changelog line at it.
4. **Propose crystals.** For each lesson that generalises beyond its case, check the crystal library first. Extend an existing crystal with the new instance; write a new crystal when it is portable, new and stated as a claim. List the candidates turned down, with a reason, in the summary.

Report the counts in the Chronicle's "By the Numbers".

### Phase 3e: Index size check

The root README is read on every request.

1. Measure it: `wc -c README.md`.
2. If it grew since the last run, or passes a limit you have set, move detail from the longest index rows down to their target READMEs until each row is keywords, a path and one sentence.
3. Report the size before and after in the Chronicle.
```

## 4. `.claude/commands/update.md`: under "Triggering", replacing the "Nightly" line

```markdown
- **Nightly (optional)**: `/update` works well as a nightly job. To run it that way, schedule `claude -p "/update"` with cron, a systemd timer or launchd, run from the palace folder (see `docs/update.md` in the MindPalace repository), and check the next morning that it ran: a new cursor, a new summary, a Chronicle.
```
