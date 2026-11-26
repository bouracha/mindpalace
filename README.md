# MindPalace

A README-centric knowledge operating system for Claude Code: retrieval-augmented generation without the vector database.

Companion repository to the talk *A Year Living in MindPalace* (LLMday London, 26 November 2026).

## Retrieval without the vector database

Context windows are now large enough to hold a great deal. They still cannot hold everything, and something has to choose what goes in.

The usual answer is a vector database: cut documents into chunks, embed them, and paste in the chunks nearest to the question.

MindPalace does it the way a person uses the index of a book:

- **A README in every folder.** Each one is an index of what sits below it and a summary of it.
- **The model navigates.** It reads the root index, follows a link to the right area, reads that area's README and opens only the files the question needs.
- **Links across the tree.** Pages point to each other with `[[wiki-links]]`, so a person's page can lead to the projects they are part of.

The model reads whole pages it chose, rather than fragments that happened to sit near the question. The price is curation: the indexes have to be kept current, and the root index travels with every request. Keeping it current is the hard part, and the mechanisms below are what have worked for me.

## Mechanisms

**Use it your way.** Everyone uses MindPalace differently, and its structure grows from your own use. Each mechanism below is a pattern you can adopt, adapt or skip; the names, formats, tags and schedules are yours to change.

1. **[The update](docs/update.md).** You run `/update` when you choose. It reads every session since the last run, folds them into the READMEs from the bottom up, and writes a summary and a newspaper-style Chronicle that asks you about anything it could not place. Running it nightly works well, but it is optional.
2. **[Crystallisation](docs/crystallisation.md).** A lesson that will matter again can be distilled into a short, named claim, a crystal, and kept in a [crystal library](mindpalace/crystal-library/README.md).
3. **[Evolution crystals](docs/evolution-crystals.md).** A meaningful change gets a short record of what changed and why, kept in the area it belongs to and read only when someone asks why things are the way they are.
4. **[Provenance](docs/provenance.md).** A captured idea says whose it is: the human's, the agent's (unvetted), adopted, joint, or raised and left unanswered. Silence is not adoption.
5. **[Conversation on uncertainty](docs/review-loop.md).** After each update, a summary and the Chronicle, a newspaper of what changed. What it could not place comes back as questions for you instead of guesses, and your answers are routed. Reading it is also the check on what was captured and where.

The idea behind all five: [give the model latitude, fix the mechanism](mindpalace/crystal-library/give-the-model-latitude-fix-the-mechanism.md). Models change, and they reason differently from one another; the mechanisms are the parts meant to hold whichever model runs inside them.

If you want the kernel to know about these mechanisms, [docs/kernel-additions.md](docs/kernel-additions.md) has short, optional text for the protected kernel files. Take what suits you. What changed in this repository, and why, is in [CHANGELOG.md](CHANGELOG.md).

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
   git clone https://github.com/bouracha/mindpalace.git
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

5. **Run `/update` whenever you like.** It picks up every session since the last run. If you want it to happen by itself, it works well as a nightly job: see [docs/update.md](docs/update.md#optional-run-it-nightly).

## Usage

### Daily use
Just talk to Claude. It listens, engages, and builds structure as patterns emerge.

### Bring the palace up to date
Run `/update` when you choose. It reads the sessions since the last run and will:
- extract insights from them
- route information to the relevant folders and cascade the indexes up to the root
- generate a structured summary and a PDF newspaper, the Chronicle

### Find things
Ask naturally: "What do I know about X?" Claude navigates the README tree to find the context.

## Structure

```
mindpalace/
├── README.md             ← Root index, read on every request
├── .claude/              ← The OS (protected, don't edit)
│   ├── CLAUDE.md         ← Kernel instructions
│   ├── settings.json     ← Permissions
│   ├── commands/         ← Slash commands (/update)
│   └── templates/        ← Chronicle PDF template
├── sessions/             ← Logs, summaries, chronicles
├── crystal-library/      ← Portable lessons, one claim per file (an example)
├── evolution-crystals/   ← Why the palace itself changed (an example)
├── me/                   ← Your profile (created on first run)
└── [emergent...]         ← whatever your own use calls for (people/,
                            projects/, topics/...), each with its own
                            README and, if you like, evolution-crystals/
docs/                     ← How the mechanisms work
```

## How it works

1. **README-centric**: every folder has a README that indexes and summarises its contents
2. **Navigation, not similarity**: the model reads the indexes and chooses what to open
3. **Timestamps**: READMEs record when they were last updated
4. **Bottom-up cascade**: changes bubble up through the README tree to the root index
5. **The update**: `/update` folds every session since the last run into the indexes (it works well as a nightly job)
6. **Evolution crystals** (optional): the reasons for each change, kept beside the current state and not loaded unless asked for
7. **Crystallisation** (optional): lessons that generalise, kept as crystals in a library
8. **Provenance** (optional): captured ideas tagged with whose they are
9. **Conversation on uncertainty**: the Chronicle is the check, and its questions replace guesses

## Protecting the Kernel (Optional but Recommended)

The `.claude/` folder is the OS kernel: it defines how MindPalace behaves. For extra protection, make it read-only:

```bash
chmod -R a-w .claude/
```

This prevents accidental modifications at the filesystem level.

To modify kernel files when needed:
```bash
chmod u+w .claude/CLAUDE.md
# make changes manually
chmod u-w .claude/CLAUDE.md
```

## License

Free for personal use. Commercial use prohibited without permission.
See [LICENSE](LICENSE) for details.

---

Built with [Claude Code](https://claude.ai/code) 🧠
