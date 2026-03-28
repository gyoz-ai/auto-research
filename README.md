# auto-research

Autonomous skill improvement for [Claude Code](https://docs.anthropic.com/en/docs/claude-code), inspired by Karpathy's [autoresearch](https://github.com/karpathy/autoresearch).

Instead of optimizing a neural network overnight, it optimizes your Claude Code skills through parallel research agents and an iterative keep/discard improvement loop.

## How it works

```
/auto-research my-skill-name
```

The command runs through 5 phases:

### Phase 1: Discovery
Reads your skill file, extracts metadata, and creates a backup.

### Phase 2: Parallel Research (5 agents)
Spawns 5 research agents simultaneously, each with a focused mission:

| Agent | Role | Method |
|-------|------|--------|
| Domain Expert | Finds best practices and conventions | Web search for guides, docs, style guides |
| Quality Auditor | Scores the skill on 7 dimensions | Structural analysis against quality rubric |
| Competitive Analyst | Finds how others solve similar problems | Searches cursor rules, AI prompts, cheatsheets |
| Gap Analyst | Identifies missing scenarios | User journey and edge case analysis |
| Tech Scout | Checks for outdated or deprecated content | Searches changelogs, breaking changes |

### Phase 3: Synthesis
Combines all findings, de-duplicates, and ranks improvement proposals using:

```
Priority Score = Impact x Confidence / Complexity
```

### Phase 4: Iterative Improvement Loop
Applies improvements one at a time (like autoresearch experiments):

1. **Apply** a single, focused change
2. **Evaluate** it (accuracy, clarity, value-add, simplicity)
3. **Keep** if it improves the skill, **Discard** if it doesn't
4. **Repeat** for the next proposal

### Phase 5: Results Report
Shows before/after quality scores, lists all kept/discarded experiments.

## The autoresearch parallel

| autoresearch (ML) | auto-research (Skills) |
|---|---|
| Modify `train.py` | Modify `SKILL.md` |
| Train for 5 min on GPU | Research with 5 parallel agents |
| Measure `val_bpb` (lower = better) | Score on 7 quality dimensions (higher = better) |
| Keep if metric improves | Keep if quality improves |
| `git reset` if worse | Revert edit if worse |
| `results.tsv` tracking | Experiment log with keep/discard |
| Simplicity criterion | Same -- complex additions with marginal value get discarded |

## Quality dimensions

The auditor agent scores skills on:

1. **Actionability** -- Can Claude immediately act on the instructions?
2. **Clarity** -- Is the language unambiguous?
3. **Completeness** -- Does it cover the full workflow?
4. **Examples** -- Are there enough concrete examples?
5. **Edge Cases** -- Does it handle failure modes?
6. **Conciseness** -- Every line earns its place?
7. **Trigger Accuracy** -- Does the description match when it should activate?

## Installation

### Option 1: Copy the file

```bash
# Create the commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Copy the command file
cp commands/auto-research.md ~/.claude/commands/auto-research.md
```

### Option 2: Symlink (stays updated with git pulls)

```bash
git clone https://github.com/gyoz-ai/auto-research.git ~/auto-research
ln -s ~/auto-research/commands/auto-research.md ~/.claude/commands/auto-research.md
```

## Usage

```bash
# In any Claude Code session:

# By skill name (looks in ~/.claude/skills/<name>/SKILL.md)
/auto-research my-skill

# By full path
/auto-research ~/.claude/skills/my-skill/SKILL.md

# By project-level path
/auto-research .claude/skills/my-skill/SKILL.md
```

The command runs autonomously -- it won't ask for permission between experiments. When it's done, you can:

1. **Review** the changes in detail
2. **Run another cycle** for deeper improvements
3. **Revert** all changes if you don't like them

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI, desktop app, or IDE extension
- An existing skill to improve (in `~/.claude/skills/` or `.claude/skills/`)

## Credits

Methodology inspired by [autoresearch](https://github.com/karpathy/autoresearch) by Andrej Karpathy -- the concept of autonomous, iterative experimentation with a keep/discard loop and simplicity criterion.

## License

MIT
