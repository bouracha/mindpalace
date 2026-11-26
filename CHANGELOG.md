# Changelog

Each entry says what changed and why.

## 2026-11-26: the mechanisms from the LLMday talk

Companion to the talk *A Year Living in MindPalace* (LLMday London, 26 November 2026).

- **Use it your way.**
  What: every mechanism below is presented as a pattern to adopt, adapt or skip, with the examples marked as examples.
  Why: everyone uses MindPalace differently and its structure grows from each person's use, so names, formats, tags and schedules are theirs to choose.
- **README: retrieval without the vector database.**
  What: the README now explains how the model navigates an index (a README in every folder, links across the tree) and lists the five mechanisms.
  Why: navigation by index is what sets MindPalace apart from the usual RAG, and its cost, curation, is what the mechanisms exist to pay.
- **The update (`docs/update.md`).**
  What: what `/update` does with every session since the last run, two optional extra steps, and an optional section on running it nightly with a check that it ran.
  Why: a curated index drifts unless something folds the sessions back into it; you trigger `/update` yourself, and running it nightly is a suggestion that works well.
- **Crystallisation (`docs/crystallisation.md`, `mindpalace/crystal-library/`).**
  What: the method, plus a library with three example crystals about the system itself.
  Why: a lesson that will matter again is easier to find, and to reuse, as a short named claim listed one line per crystal. Its social ancestor is the slogan: short, portable and full of meaning, but with the backing kept beside it.
- **Evolution crystals (`docs/evolution-crystals.md`, `mindpalace/evolution-crystals/`).**
  What: a changelog line for most changes and a full crystal when the thinking moves, kept per area, with an example folder.
  Why: code keeps snapshots because each one is testable; understanding is not, so a later reader needs the reasons rather than old copies.
- **Provenance (`docs/provenance.md`).**
  What: five tags recording whose idea a captured idea is.
  Why: a vetted idea, an unvetted proposal and a joint one deserve different trust, and silence is not adoption.
- **Conversation on uncertainty (`docs/review-loop.md`).**
  What: the Chronicle as the human's check, with questions instead of guesses.
  Why: the agent handles the volume and the human keeps the judgement, as an overnight desk sends a query to the editor instead of printing a guess.
- **Kernel additions (`docs/kernel-additions.md`).**
  What: optional, paste-ready text for `.claude/CLAUDE.md` and `.claude/commands/update.md`.
  Why: the model may not edit the protected kernel, so the owner chooses what to adopt and pastes it in.
- **Template root README (`mindpalace/README.md`).**
  What: the two example areas in the index, and a suggestion to keep each index row to keywords, a path and one sentence.
  Why: the root index is read on every request, so its size is a cost paid every time.
- **Install instructions.**
  What: the clone URL placeholder is replaced with this repository.
  Why: the placeholder did not work as written.
- **Unchanged:** the licence, and every file under `.claude/`.
