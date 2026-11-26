# Conversation on uncertainty

The agent does the running; the human reviews. After each update there are two outputs: a structured summary for the model and the Chronicle for you, a newspaper of what changed. If you run the update nightly, that is a newspaper each morning.

## The loop

1. **The update decides what mattered.** [`/update`](update.md) reads the sessions since the last run and writes the news: what was captured, where it was filed, what changed.
2. **What it cannot place becomes a question.** An item it cannot route, or a fact it is not sure of, goes to `sessions/unrouted.md` and appears in the Chronicle under "Questions for the Reader". It is asked, not guessed.
3. **You answer, it routes.** Reply in a later session and the answer is filed where it belongs. A question left unanswered is archived after a week rather than asked again and again.
4. **Reading it is the check.** The Chronicle shows what was captured, where it went and what the system is unsure about, all in one place. Something filed in the wrong place, or missed, is visible at the next update, while it is still cheap to correct.

## Why it works

Newsrooms solved this long ago. An overnight desk decides what mattered from everything that came in, and a subeditor who is unsure of a fact sends a query to the editor instead of printing a guess. The same arrangement suits an agent: it handles the volume, the human keeps the judgement, and the record shows which is which (see [provenance.md](provenance.md)).

## Make it yours

The Chronicle's sections, length, tone and format are yours to change; its template is `.claude/templates/chronicle.tex`, which you can edit by hand. What has worked for me:

- **Few, specific questions.** Five to ten, each answerable in a line, so that answering takes a minute.
- **Worth reading.** Headlines, specifics, the occasional joke. A report nobody reads is not a check.
- **A missing Chronicle is an alarm.** If the update runs nightly and there is no new Chronicle in the morning, the night did not run.
