# Why the palace keeps reasons, not versions

> Date: 2026-11-26 · Tier 2 · Originator: `[A+J]`

## The change

The template gained two optional mechanisms: evolution crystals (this folder, and one in any area that wants one) and a crystal library (`crystal-library/`). Both are examples, to keep, change or delete.

## The reasoning

A README-centric palace retrieves by navigation: the model reads an index and opens what it needs. That reads whole, chosen pages instead of fragments found by similarity, but it depends on curation, and curation is where it fails. Indexes drift, old copies accumulate, and a later session cannot tell why a page says what it says.

Version control answers "what changed?" for code, because every snapshot can be tested. Understanding cannot be tested snapshot by snapshot, so the record that matters is the reason for each change. The folder keeps the latest version; this folder keeps the reasons.

## Alternatives considered

- **Keep every version in the working folder.** Rejected: each extra copy makes every reader work out which one is live.
- **Rely on version control alone.** Rejected as insufficient: commit messages are short, written for code, and rarely opened while the model works.

## The trail

The first rule was to write few crystals, only for major shifts. It was inverted to write everything and read selectively, with two tiers so that recording every change never means an essay per typo. Provenance tags were added later, after a suggestion from the agent was read back as the owner's own idea.

## Open questions

How much to capture live and how much to leave to the backfill at the next `/update`; when a changelog line deserves promotion to a full crystal.
