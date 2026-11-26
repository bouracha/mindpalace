# Give the model latitude, fix the mechanism

> Type: WHY / HOW · Originator: `[A]` · Created: 2026-11-26

## The crystal

Do not over-tune the instructions a model is given. Models differ in how they reason, and they keep differing as they improve, so a prompt fitted closely to one model's habits is fitted to something that will change. Give the model as much latitude as possible in how it does the work. Spend the engineering on mechanisms: the parts that must hold whatever model sits inside them, and whose correctness can be checked from outside the model.

## Why it matters

- The model is the part most likely to be replaced. Each new generation reasons differently, and a system tuned to the last one degrades quietly.
- Mechanisms do not depend on how the model reasons: a file layout, an index that is kept current, a fixed reply format that is validated, a provenance tag, a regular consolidation pass, a record of reasons.
- Human institutions work the same way. Peer review, double-entry bookkeeping, minutes and a newspaper's corrections column work whoever fills the role, because they constrain outcomes, not thought. A system of language models has the properties those institutions evolved to survive: members who forget, who reason unevenly and differently, who are confidently wrong at times, and who are replaced.

## Where it applies

Designing any agent system that will outlive its current model; deciding whether a failure needs a better prompt or a better mechanism (usually the second); reviewing a long instructions file for rules that only one model ever needed.
