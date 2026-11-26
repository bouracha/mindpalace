# The update (`/update`)

A palace is only as good as its indexes, and indexes drift. `/update` is the consolidation pass: you run it when you choose, it reads every session since the last run, folds what happened into the READMEs and reports what it did. Run it daily, weekly or after a long session; it picks up from wherever it left off.

## What it does

The command lives in `mindpalace/.claude/commands/update.md`. In order:

1. **Finds the sessions since the last run.** Claude Code keeps every session as a JSONL transcript. The command reads the ones newer than a cursor kept in `sessions/README.md`, however long ago that was, subagent transcripts included.
2. **Extracts, then merges.** Long transcripts are read in chunks. What each chunk says about a person, project or topic is extracted, then merged per entity, so no step needs every session in one context window.
3. **Routes and cascades.** Each entity's news goes into its folder's README, or creates a folder when a topic has earned one. Timestamps and summaries then cascade from the bottom up to the root index.
4. **Asks rather than guesses.** Anything it cannot place goes to `sessions/unrouted.md` and comes back to you as a question.
5. **Writes two outputs.** A structured summary for the model (`sessions/summaries/`) and the Chronicle, a newspaper-style PDF for you (`sessions/chronicles/`). The Chronicle shows what was captured, where it went and what the system is unsure about; reading it is how you review the system (see [review-loop.md](review-loop.md)). If you use [provenance tags](provenance.md), both outputs carry them, so an idea the agent proposed is never reported as one you decided.

Two further steps have worked well for me. They are optional, and their text is in [kernel-additions.md](kernel-additions.md):

6. **Backfills evolution crystals.** A change made since the last run without a changelog line gets one, a genuine shift in thinking gets a full crystal, and lessons that generalise are proposed for the crystal library. See [evolution-crystals.md](evolution-crystals.md) and [crystallisation.md](crystallisation.md).
7. **Checks the always-read index.** The root README is read on every request, so its size is a cost paid every time. The update can report the size and trim index rows back to keywords, a path and one sentence, moving detail down to the area README, which is read only when its topic comes up.

Adapt any of it: the command is a plain markdown file, and its steps, outputs and formats are yours to change.

## Optional: run it nightly

Running `/update` every night works well: the palace is current each morning and nobody has to remember. It is a suggestion, not a requirement.

`/update` runs headless with `claude -p`. Run it from the palace folder, so that the project settings and the kernel load. Cron and launchd start with a minimal `PATH`, so give the full path to `claude` (find it with `which claude`). The paths and the time below are placeholders.

### cron (Linux or macOS)

```cron
30 3 * * * cd /path/to/MindPalace && /path/to/claude -p "/update" >> sessions/nightly.log 2>&1
```

### systemd user timer (Linux)

`~/.config/systemd/user/mindpalace-update.service`:
```ini
[Unit]
Description=MindPalace nightly update

[Service]
Type=oneshot
WorkingDirectory=/path/to/MindPalace
ExecStart=/path/to/claude -p "/update"
```

`~/.config/systemd/user/mindpalace-update.timer`:
```ini
[Unit]
Description=Run the MindPalace update every night

[Timer]
OnCalendar=*-*-* 03:30
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now mindpalace-update.timer
loginctl enable-linger "$USER"   # lets the timer run while you are logged out
```

### launchd (macOS)

`~/Library/LaunchAgents/com.example.mindpalace-update.plist`:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.example.mindpalace-update</string>
  <key>WorkingDirectory</key><string>/path/to/MindPalace</string>
  <key>ProgramArguments</key>
  <array>
    <string>/path/to/claude</string>
    <string>-p</string>
    <string>/update</string>
  </array>
  <key>StartCalendarInterval</key>
  <dict>
    <key>Hour</key><integer>3</integer>
    <key>Minute</key><integer>30</integer>
  </dict>
  <key>StandardOutPath</key><string>/path/to/MindPalace/sessions/nightly.log</string>
  <key>StandardErrorPath</key><string>/path/to/MindPalace/sessions/nightly.log</string>
</dict>
</plist>
```

```bash
launchctl load ~/Library/LaunchAgents/com.example.mindpalace-update.plist
```

A Mac that is asleep at the scheduled time runs the job when it wakes.

### Permissions

An unattended run cannot answer a permission prompt. The template's `.claude/settings.json` already allows the tools `/update` needs and denies writes to the kernel, and a headless run reads the same file. Prefer those rules to a flag that skips permission checks altogether. If you do use such a flag, make the kernel read-only on disk first (see *Protecting the Kernel* in the main README), because nothing else will stop a write.

### Check that it ran

A scheduled job that fails tells nobody. The next morning:

- `sessions/README.md` shows a new cursor;
- a new summary exists in `sessions/summaries/`;
- the end of `sessions/nightly.log` shows no error.

The commonest silent failure is a CLI that has been logged out: the job starts, cannot authenticate and stops. The simplest alarm is the Chronicle itself. If there is no new Chronicle in the morning, the night did not run.
