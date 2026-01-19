# MindPalace

A knowledge operating system powered by Claude Code. Talk naturally — it organizes as patterns emerge.

## What is this?

MindPalace is a personal knowledge OS that:
- **Captures** information from your conversations
- **Organizes** into folders as patterns emerge (people, projects, topics)
- **Retrieves** context when you need it
- **Summarizes** sessions into structured notes + fun newspaper PDFs

No setup required. Just start talking.

## Prerequisites

### Required
- [Claude Code CLI](https://claude.ai/code) installed and authenticated

### Optional (for PDF chronicles)
```bash
# Ubuntu/Debian
sudo apt-get install texlive-latex-base texlive-fonts-recommended texlive-latex-extra

# macOS
brew install basictex

# Or lightweight alternative
cargo install tectonic
```

## Installation

1. **Clone this repo:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/mindpalace.git
   cd mindpalace
   ```

2. **Copy the `mindpalace` folder** to wherever you want your knowledge base:
   ```bash
   cp -r mindpalace ~/MindPalace
   ```

3. **Start Claude Code in that folder:**
   ```bash
   cd ~/MindPalace
   claude
   ```

4. **Talk naturally.** On first run, it will ask your name and create your profile.

## Usage

### Daily use
Just chat with Claude. It listens, engages, and builds structure as patterns emerge.

### Process sessions
Run `/update` to:
- Extract insights from recent conversations
- Route information to relevant folders
- Generate a structured summary + PDF newspaper

### Find things
Ask naturally: "What do I know about X?" — Claude navigates the README tree to find context.

## Structure

```
mindpalace/
├── README.md           ← Root navigation
├── .claude/            ← The OS (protected, don't edit)
│   ├── CLAUDE.md       ← Kernel instructions
│   ├── settings.json   ← Permissions
│   ├── commands/       ← Slash commands (/update)
│   └── templates/      ← Chronicle PDF template
├── sessions/           ← Logs, summaries, chronicles
├── me/                 ← Your profile (created on first run)
└── [emergent...]       ← people/, projects/, topics/ appear as needed
```

## How it works

1. **README-centric**: Every folder has a README that indexes its contents
2. **Timestamps**: READMEs track when they were updated
3. **Context reconstruction**: Topic README → timestamp → session summary → full context
4. **Bottom-up cascade**: Changes bubble up through the README tree

## License

MIT — do whatever you want with it.

---

Built with [Claude Code](https://claude.ai/code) 🧠
