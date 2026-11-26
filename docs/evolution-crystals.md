# Evolution crystals

Version control keeps snapshots of code because every snapshot can be tested: this version worked, the next one works, and the diff between two working states tells the whole story. Understanding does not work that way. A state of understanding is diffuse, it shifts from one version to the next, and no test certifies any of them. A pile of old drafts tells a later reader what was believed, never why it changed.

An evolution crystal keeps the reason instead: a short record of **what changed and why**, kept in the area it belongs to. The folder holds the current version of each document; the history lives in the crystals.

This is a pattern that has worked for me, not a rule. The tiers, formats and folder names below are suggestions: adopt them, adapt them or skip them.

## Two tiers

**Tier 1, a changelog line, for most changes.** One line appended to the area's `evolution-crystals/changelog.md`, for example:

```
- YYYY-MM-DD HH:MM | what changed | why: one clause | ref: file or commit (optional) [tag]
```

**Tier 2, a full crystal, for genuine evolution.** A dated file, such as `evolution-crystals/YYYY-MM-DD-<slug>.md`, written when ideas actually moved: a concept coined or abandoned, a reframe, a decision with alternatives rejected, a phrasing that emerged over several drafts. I give each an `Originator:` field (see [provenance.md](provenance.md)) and these sections: the change; the reasoning; alternatives considered; the trail (how it got there); open questions.

A full crystal also gets a changelog line pointing to it, so the changelog stays the complete chronological spine. One thread per file keeps them findable: two separate evolutions on the same day are two crystals.

## What has worked

- **Write a lot, read selectively.** Storage is cheap; lost reasoning is not. Crystals are not loaded by default, so they cost nothing on ordinary requests. They are opened when someone asks why something is the way it is.
- **Keep them per area.** Any folder that accumulates decisions can have its own: a project, a person's page, the structure of the palace itself.
- **Edit in place.** Updating READMEs and documents in place, instead of appending versions or keeping old copies beside the new one, spares every reader the work of finding the live copy.
- **Tag provenance**, if you use tags (see [provenance.md](provenance.md)).
- **Capture live, backfill on update.** Write the line when the change happens; the [update](update.md) can catch what was missed and promote notable changes to full crystals.
- **Generative as well as corrective.** A new structure, approach or decision counts, not only a fix.

## Reading them back

To reconstruct why an area looks the way it does, open its changelog first (the chronological spine), then the dated crystal for the full story.

## Example

The template's root folder has an example: [mindpalace/evolution-crystals/](../mindpalace/evolution-crystals/README.md), with a changelog and a full crystal explaining why the palace keeps reasons rather than versions. Keep it, rename it or delete it.

## Why this suits language models

A later session, or a different model, reconstructs what happened from what earlier sessions wrote down. It is in the position of a historian reading footnotes, or a scientist reading a lab notebook that kept its crossed-out lines: it can trust the record because the reasoning, and the corrections, are on the page. Evolution crystals borrow more from the lab notebook than from the git log.
