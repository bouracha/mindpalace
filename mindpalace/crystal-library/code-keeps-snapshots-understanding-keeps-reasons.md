# Code keeps snapshots, understanding keeps reasons

> Type: WHAT / WHY · Originator: `[A+J]` (the idea from the author, the wording from the agent) · Created: 2026-11-26

## The crystal

Code is versioned by snapshot because every snapshot can be tested: this version worked, the next one works, and the diff between two working states is the whole story. Understanding cannot be versioned that way. It is diffuse, it shifts between versions, and no test certifies any one of them, so a pile of snapshots tells a later reader what was believed and never why it changed. Understanding needs the motivation for each change, kept beside the current state.

## Why it matters

- It is why a knowledge base needs more than version control: the folder keeps the latest version, and the evolution crystals keep the reasons.
- It shows the working, not just the result. A reconstruction that only converges on its best version hides its uncertainty. One that records each correction, and why it was made, can be trusted by someone who was not there.

## Where it applies

Deciding what to record when a document, plan or design changes; answering "why not just use git?"; teaching people to show how they iterated, not only what they arrived at.
