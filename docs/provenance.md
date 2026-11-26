# Provenance

When a human and an agent think together, ideas from both end up in the same records. A month later, a proposal the agent made in passing can be read back as something the human decided. Provenance tags prevent that: a captured idea says whose it is.

This is the scheme I use. The letters and the number of tags are yours to choose; what matters is that the record can tell a vetted idea from an unvetted one.

## Why it matters

A vetted idea, an unvetted proposal and a joint one deserve different trust. Collapsing them hides whether a claim was ever checked by the person whose palace it is. The records are what later sessions, and later models, reason from.

## The tags

- `[A]`: the author, the human who owns the palace, originated it.
- `[J]`: the agent proposed it, and it is **unvetted**.
- `[J→A]`: the agent proposed it and the human adopted it. It is the human's by endorsement now.
- `[A+J]`: built together, each building on the other.
- `[J?]`: the agent proposed it **and raised it with the human, who did not reply**.

## How I use them

- **Default to `[J]`.** An idea that first appears in an agent's turn stays `[J]` until the human explicitly adopts it.
- **Adoption is an explicit event.** Silence is not adoption, and repetition is not consent: raising something twice counts the raising, not the agreement. That is what `[J?]` is for. The first four tags record who originated an idea; `[J?]` separates something the human never saw from something put to them that drew no answer. The second is the risky one, because a record showing it was raised, perhaps more than once, later reads as acquiescence.
- **Take the tag from the transcript**, by who said it first, rather than guessing.
- **Keep the tag in the telling.** A summary should not phrase a `[J]` proposal as "you decided", "you realised" or "you wanted".

## Where the tags go

- **Evolution crystals** can carry an `Originator:` field, and changelog lines that record an idea, rather than an event, can carry a tag. See [evolution-crystals.md](evolution-crystals.md).
- **Crystals** in the library can carry a tag beside their type. See [crystallisation.md](crystallisation.md).
- **The update** (`/update`) can carry the tags from the session transcripts into its summaries and the Chronicle. See [update.md](update.md).
- **Read-backs** can state whose idea it was: "you proposed X; I proposed Y, and you have not adopted it."

Optional kernel wording for all of this is in [kernel-additions.md](kernel-additions.md).
