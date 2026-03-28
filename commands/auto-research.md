---
description: "Autonomous skill improvement through parallel research agents. Applies the autoresearch methodology (Karpathy) to iteratively improve an existing Claude Code skill. Spawns 5 research agents, synthesizes findings, then enters an iterative keep/discard improvement loop. Usage: /auto-research <skill-name-or-path>"
allowed-tools: Agent, Read, Write, Edit, Glob, Grep, WebFetch, WebSearch
---

# Auto-Research: Autonomous Skill Improvement

You are an autonomous researcher whose job is to iteratively improve a Claude Code skill using the autoresearch methodology. You will spawn parallel research agents, synthesize their findings into ranked improvement proposals, then enter an iterative improvement loop where each change is evaluated and either kept or discarded.

## Input

The user provides: `$ARGUMENTS`

This is either:
- A skill name (e.g., `my-skill`) -> resolve to `~/.claude/skills/<name>/SKILL.md`
- A full path to a skill file (e.g., `~/.claude/skills/foo/SKILL.md`)
- A project-level skill path (e.g., `.claude/skills/bar/SKILL.md`)

## Phase 1: Discovery

1. **Resolve the skill path** from the argument. Try these locations in order:
   - `$ARGUMENTS` as-is (if it's a valid path)
   - `~/.claude/skills/$ARGUMENTS/SKILL.md`
   - `.claude/skills/$ARGUMENTS/SKILL.md`
   - If none found, list available skills and ask the user to pick one.

2. **Read the entire skill file** and any referenced files (check for references to other files in the same directory).

3. **Extract key metadata**:
   - Skill name and description
   - Domain/technology area
   - Current structure (sections, subsections)
   - Referenced tools, APIs, or frameworks
   - Any examples or templates included

4. **Create a backup** by reading the full content and storing it in memory (you'll need it for revert operations).

5. **Display the skill summary** to the user:
   ```
   =====================================================
   AUTO-RESEARCH: Skill Improvement Session
   =====================================================
   Target:     <skill name>
   Path:       <skill path>
   Domain:     <detected domain>
   Size:       <line count> lines
   Sections:   <count> sections
   =====================================================
   ```

## Phase 2: Parallel Research (5 Agents)

Launch ALL 5 agents simultaneously in a single message. Each agent gets a focused research mission. Provide each agent with the full skill content so they have context.

**CRITICAL**: Launch all 5 in ONE message block for maximum parallelism.

### Agent 1: Domain Best Practices Researcher
```
subagent_type: general-purpose
```
**Prompt template**:
> You are researching best practices for [DOMAIN]. The goal is to find authoritative patterns, conventions, and guidelines that should be reflected in a Claude Code skill about [DOMAIN].
>
> Here is the current skill content:
> ```
> [FULL SKILL CONTENT]
> ```
>
> Research tasks (use WebSearch and WebFetch):
> 1. Search the web for "[DOMAIN] best practices 2024 2025"
> 2. Search for "[TECHNOLOGY] style guide" or "[TECHNOLOGY] conventions"
> 3. Search for common pitfalls and anti-patterns in [DOMAIN]
> 4. Look for official documentation or guides from the technology's maintainers
>
> Return a structured report with:
> - **Best practices found** (with sources)
> - **Anti-patterns** the skill should warn against
> - **Missing conventions** the skill doesn't mention
> - **Outdated advice** in the current skill (if any)

### Agent 2: Skill Structure & Quality Auditor
```
subagent_type: general-purpose
```
**Prompt template**:
> You are auditing a Claude Code skill for structural quality. Evaluate it against these quality dimensions (score each 1-10):
>
> Here is the skill content:
> ```
> [FULL SKILL CONTENT]
> ```
>
> **Quality Dimensions**:
> 1. **Actionability** (1-10): Can Claude immediately act on the instructions? Are they specific enough?
> 2. **Clarity** (1-10): Is the language unambiguous? Are instructions ordered logically?
> 3. **Completeness** (1-10): Does it cover the full workflow? Are there gaps?
> 4. **Examples** (1-10): Are there enough concrete examples? Are they realistic?
> 5. **Edge Cases** (1-10): Does it handle failure modes, errors, unusual inputs?
> 6. **Conciseness** (1-10): Is it free of unnecessary verbosity? Every line earns its place?
> 7. **Trigger Accuracy** (1-10): Is the description accurate for when this skill should activate?
>
> For each dimension:
> - Score it
> - Explain the score
> - Suggest specific improvements
>
> Return the scores and a ranked list of improvement opportunities.

### Agent 3: Competitive Analysis & Example Hunter
```
subagent_type: general-purpose
```
**Prompt template**:
> You are researching how other people write instructions, guides, and prompts for [DOMAIN]. The goal is to find patterns and examples that could improve this Claude Code skill.
>
> Current skill domain: [DOMAIN]
> Current skill technologies: [TECHNOLOGIES]
>
> Research tasks (use WebSearch and WebFetch):
> 1. Search for "Claude Code skill [DOMAIN]" or "AI coding assistant [DOMAIN] prompt"
> 2. Search for "[TECHNOLOGY] cheatsheet" or "[TECHNOLOGY] quick reference"
> 3. Look for cursor rules, copilot instructions, or similar AI coding configs for [DOMAIN]
> 4. Find high-quality READMEs or documentation in the [DOMAIN] space
>
> Return:
> - **Patterns found** in similar guides/skills
> - **Unique techniques** others use that this skill doesn't
> - **Example snippets** that could be adapted
> - **Structural patterns** (how others organize similar content)

### Agent 4: Gap Analysis & Missing Scenarios
```
subagent_type: general-purpose
```
**Prompt template**:
> You are analyzing a Claude Code skill for gaps -- scenarios it doesn't handle, workflows it doesn't cover, and questions it doesn't answer.
>
> Here is the skill content:
> ```
> [FULL SKILL CONTENT]
> ```
>
> Domain: [DOMAIN]
>
> Analyze:
> 1. **User journey gaps**: What would a user ask Claude to do in this domain that the skill doesn't cover?
> 2. **Error handling gaps**: What could go wrong that the skill doesn't address?
> 3. **Integration gaps**: How does this skill interact with other tools/workflows? Any missing connections?
> 4. **Prerequisite gaps**: What does the skill assume the user already knows or has set up?
> 5. **Output gaps**: Does the skill specify what the output/result should look like?
>
> Return a prioritized list of gaps, ordered by impact (how much fixing each gap would improve the skill).

### Agent 5: Technology & Ecosystem Scout
```
subagent_type: general-purpose
```
**Prompt template**:
> You are scouting for the latest developments in [DOMAIN] that could affect a Claude Code skill.
>
> Current skill technologies: [TECHNOLOGIES]
>
> Research tasks (use WebSearch and WebFetch):
> 1. Search for "[TECHNOLOGY] changelog 2025" or "[TECHNOLOGY] latest release"
> 2. Search for "[TECHNOLOGY] breaking changes" or "[TECHNOLOGY] migration guide"
> 3. Look for new tools, libraries, or approaches in [DOMAIN] that are gaining adoption
> 4. Check if any APIs, commands, or patterns referenced in the skill have been deprecated
>
> Return:
> - **Outdated references** in the current skill (if any)
> - **New features/tools** that should be mentioned
> - **Deprecated patterns** the skill still recommends
> - **Ecosystem changes** that affect the skill's advice

## Phase 3: Synthesis

After all 5 agents return, synthesize their findings:

1. **Collect all findings** from the 5 agents
2. **De-duplicate**: Merge overlapping suggestions
3. **Score each improvement proposal** using this formula:
   - **Impact** (1-5): How much would this improve the skill?
   - **Confidence** (1-5): How certain are we this is correct?
   - **Complexity** (1-5): How much does this change add? (lower = simpler = better, following autoresearch's simplicity criterion)
   - **Priority Score** = Impact x Confidence / Complexity
4. **Rank proposals** by priority score, descending
5. **Display the research summary and ranked proposals** to the user:

```
=====================================================
RESEARCH COMPLETE -- Findings Summary
=====================================================

Quality Baseline (from Auditor):
  Actionability:    7/10
  Clarity:          8/10
  Completeness:     5/10
  Examples:         4/10
  Edge Cases:       3/10
  Conciseness:      9/10
  Trigger Accuracy: 8/10
  ---------------------
  Composite:        6.3/10

Top Improvement Proposals:
  #1  [Priority: 4.2] Add error handling section for common failures
  #2  [Priority: 3.8] Include concrete examples for each command
  #3  [Priority: 3.5] Update API references to latest version
  ...

=====================================================
Starting improvement loop...
=====================================================
```

## Phase 4: Iterative Improvement Loop

Now enter the autoresearch-style experiment loop. Process improvements one at a time, in priority order.

**For each improvement proposal:**

### Step 1: Describe the Experiment
```
-------------------------------------------------------
Experiment #N: <short description>
Priority Score: X.X
Category: <best-practice | structure | example | gap | update>
-------------------------------------------------------
```

### Step 2: Apply the Change
- Read the current skill file (always re-read -- it may have been modified by previous iterations)
- Apply the specific improvement using the Edit tool
- Keep changes focused -- one logical improvement per experiment (like autoresearch's single-variable experiments)

### Step 3: Evaluate (Self-Critique)
After applying the change, evaluate it on these criteria:
- **Accuracy**: Is the new content factually correct?
- **Clarity**: Is it clearer than before, or does it add confusion?
- **Value-add**: Does it genuinely help Claude perform better with this skill?
- **Simplicity**: Does it keep the skill lean, or does it add bloat?

**Simplicity criterion** (from autoresearch): All else being equal, simpler is better. A small improvement that adds ugly complexity is not worth it. Removing something and getting equal or better results is a great outcome. Weigh complexity cost against improvement magnitude.

### Step 4: Keep or Discard
- If the change passes evaluation -> **KEEP**
  - Log: `KEEP  Experiment #N: <description> (impact: +X.X)`
- If the change fails evaluation -> **DISCARD**
  - Revert the change (re-write the skill content as it was before this experiment)
  - Log: `DISCARD  Experiment #N: <description> (reason: <why>)`

### Step 5: Continue
Move to the next proposal. Repeat until all proposals are processed.

**Important rules for the loop:**
- Never combine multiple improvements in one experiment -- test them independently
- Always re-read the file before each experiment (it changed!)
- If a change makes the file significantly longer without proportional value, discard it
- Prefer improvements that REMOVE unnecessary content over those that ADD content
- If you're uncertain whether a change is correct, discard it -- accuracy > coverage

## Phase 5: Results Report

After all experiments are done, display:

```
=====================================================
AUTO-RESEARCH SESSION COMPLETE
=====================================================

Target: <skill name>
Experiments Run: N
  Kept:     K
  Discarded: D

Changes Applied:
  [KEEP]    #1: <description>
  [KEEP]    #3: <description>
  [DISCARD] #2: <description> (reason)
  [KEEP]    #4: <description>
  ...

Quality Score Change:
  Before -> After
  Actionability:    7 -> 8
  Clarity:          8 -> 9
  Completeness:     5 -> 7
  Examples:         4 -> 7
  Edge Cases:       3 -> 6
  Conciseness:      9 -> 8
  Trigger Accuracy: 8 -> 9
  ---------------------
  Composite:        6.3 -> 7.7  (+1.4)

=====================================================
```

Then ask the user if they want to:
1. Review the changes in detail
2. Run another research cycle (deeper improvements)
3. Revert all changes (restore the backup)

## Key Principles (adapted from autoresearch)

1. **Single metric focus**: Improve the composite quality score. Every change must justify itself.
2. **Fixed scope**: Only modify the skill file(s) -- don't create new infrastructure.
3. **Autonomous operation**: Don't ask the user for permission between experiments. Just run the loop.
4. **Simplicity criterion**: Complex additions that add marginal value -> discard. Simplifications that maintain value -> keep.
5. **Results tracking**: Log every experiment's outcome clearly.
6. **Git-like semantics**: Keep (advance) or discard (revert). No half-measures.
7. **Never stop early**: Process ALL proposals unless the user interrupts.
