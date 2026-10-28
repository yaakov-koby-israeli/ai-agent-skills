---
name: senior-skill-architect
description: "Design, build, test, and improve reusable agent skills and their platform equivalents (SKILL.md for Claude Code or Gemini, Cursor rules, AGENTS.md / GEMINI.md sections, or custom agent instructions). Use when the user asks to create a new skill, turn a prompt or repeated workflow into a reusable skill, port a skill to another AI tool, or review, fix, score, or optimize an existing skill. Do NOT use for general coding, one-off tasks, project-wide coding conventions with no reusable workflow, or system prompts that will not be saved and reused."
---
# Senior Skill Architect

## Core Principles

1. **Plain, direct language.** Start instructions with an action verb ("Read", "Ask", "Format", "Verify"). Avoid buzzwords and filler.
2. **The description is the trigger, where triggering exists.** On platforms that load skills by description, put all "when to use" rules in the `description` because the model reads the body only after the skill triggers. State what it does, when to use it (using natural user phrasing), and when not to use it. On platforms without description-based loading, open the body with these rules instead (see Step 2.5).
3. **Choose the interaction mode for the generated skill yourself.** Select a two-turn flow when the generated skill alters existing work or depends on unverified facts (Turn 1: assess and ask; Turn 2: execute). Select one-shot execution for deterministic transformations. State your choice with a one-line reason. If the user asks to skip the questions, proceed and mark every assumption explicitly.
4. **Explain the why.** Attach a brief reason to every hard rule in a generated skill, because models generalize better from reasons and apply bare rules either too literally or not at all.
5. **No invented facts.** Instruct generated skills to never invent missing data, fake metrics, or reuse unconfirmed chat assumptions. Require them to ask the user or insert a clearly marked placeholder.
6. **Progressive disclosure.** Keep SKILL.md under 500 lines. Move content into `references/` when it is needed only in some runs, and deterministic logic into `scripts/` regardless of length. Link each file with an explicit trigger condition ("Read `references/api.md` only when the user mentions the API").
7. **Cover the request cluster.** Design for closely related requests and edge cases in the same domain, not a single exact scenario, so the skill does not break on the first variation.
8. **Write for the weakest model that will run it.** Make every step explicit and give it a clear end condition, because smaller models do not infer unstated intent.
9. **Language.** Write generated skills in English (including all code comments) unless the user asks otherwise, because most models follow English instructions most precisely. Talk to the user in their language.

## Workflow

### Step 0: Identify the Mode
- **Mode A:** Build a new skill from scratch, an existing prompt, a workflow, or by porting a skill to another platform.
- **Mode B:** Diagnose and improve an existing SKILL.md or platform rule file.

### Step 1A: Align (Mode A)
When converting an existing prompt, retain stable instructions and strip one-off context (specific names, dates, data). Always propose the generated skill's execution mode (One-Shot vs. Two-Turn) yourself.

Proceed directly to Step 2 if the request already includes a concrete example and a clear output. Otherwise, ask at most 3 questions if critical context is missing:
1. One real example of input and the desired output.
2. The 1-2 mistakes models usually make on this task.
3. Where it runs (one or more platforms) and whether it needs scripts or auxiliary files. Default to standard `SKILL.md` if the user does not say, and state that default.

If you ask questions, **stop and wait for answers**. Do not build in the same turn, because a skill built on guesses has to be rebuilt.

### Step 1B: Diagnose (Mode B, Turn 1)
1. Read the full skill file and every referenced file.
2. Score the skill against the Core Principles and Step 3 validation rules. Start at 10 and subtract 3 per High, 1 per Medium, and 0.5 per Low (minimum 1). Show the arithmetic (`10 - ... = Score`), so the score can be checked.
   - **High:** broken or missing triggering, invalid frontmatter, missing workflow, a promise the skill cannot deliver, or a step with no end condition.
   - **Medium:** ambiguity a model could misread, an unverifiable check, an unused input, missing guardrails against invented data/metrics, or contradictory rules.
   - **Low:** wording, padding, or minor style.
3. List issues by severity (High / Medium / Low), stating the reason and a concrete fix for each.
4. Preserve sections that work. Run the Step 3 checks against your proposed diff now, not after approval, so the user approves a change that has already been tested.
5. Output the diagnosis, the proposed diff, and its Step 3 validation summary, then **stop and wait for approval**. On approval, apply the approved diff and go to Step 4.

### Step 2: Build (Mode A)
Include these required sections:
- YAML frontmatter (`name`, `description`)
- `# Title`
- `## Workflow`
- `## Output Format` (a template or one concrete example). If the skill's result is an action rather than text (edit files, deploy), define the completion report instead.

Add optional sections only when they earn their place:
- `## Core Principles` when the task requires judgment calls
- `## Gotchas` for known, specific failure modes
- `references/`, `scripts/`, `assets/` for content not needed on every run

Fill every section with concrete, domain-specific rules and never leave generic placeholders. If `references/example-skill.md` exists next to this SKILL.md, read it for tone and structure; otherwise, mirror the structure and density of this skill.

### Step 2.5: Adapt to Target Platform
Skip this step if the only target is a standard `SKILL.md` (Claude Code, Gemini, or Claude API).
1. If `references/platforms.md` exists, read it.
2. Map the skill to each target's format: file name, frontmatter fields, trigger mechanism, and install path.
3. If a target has no description-based triggering (e.g., always-on `AGENTS.md` or `GEMINI.md`), move the "when to use" and "do not use" rules into the opening section of the body, because nothing reads the description before the content loads.
4. If `references/platforms.md` is missing, does not cover the target, or you are unsure of the current format, state that clearly and ask the user for the format or docs. Do not guess, because platform specs change often.

### Step 3: Validate
1. **Technical check:**
   - For `SKILL.md` targets: `name` uses lowercase letters, digits, and hyphens only; max 64 chars; matches the folder name. Avoid "claude" and "anthropic" (rejected by Claude, harmless elsewhere). `description`: max 1024 chars; wrapped in double quotes or a `>` folded scalar; 
   - For other targets: apply the constraints from `references/platforms.md` or user-provided docs.
   - Verify every referenced file exists (or is generated in the current output).
   - If tools allow, run each script once with sample input. Otherwise mark it `UNTESTED` in the output, so the user knows it is unverified.
2. **Trigger test:** Write 3 prompts that should trigger the skill and 2 near-misses that should not. Each near-miss must share at least one keyword with the description, so the test checks the negative rules instead of passing trivially. Tighten the description if a near-miss would match; broaden it if a should-trigger prompt is borderline.
   - If a subagent or fresh session is available, give it only the name and description plus each prompt, and record its decision. Otherwise label the table "Simulated", because a model grading its own description is biased toward Pass.
3. **Coverage check:** For each section in the generated skill's Output Format, name the workflow step that produces it (`[Output Section] <- [Step X]`). Flag any section with no producing step, and any step with no end condition.
4. **Weak-model check:** Find any step that relies on an unstated assumption and make it explicit before outputting.

### Step 4: Deliver
1. Output the full skill file and any auxiliary files in chat.
2. Save or update the skill only if the user asked and skill/file tools are available. Use the skills directory or tool of the current agent (for example `.claude/skills/<name>/` in Claude Code, `.agents/skills/<name>/` in tools that follow the shared convention, or built-in skill storage), at personal or project scope. Ask if unclear.
3. In Mode B, write the exact diff approved in Step 1B. If anything must change after approval, show the new diff and ask again.

## Output Format

**Mode A:**
1. Execution mode of the generated skill with a one-line reason.
2. The full `SKILL.md` in a four-backtick block.
3. Each auxiliary file in its own block, labeled with its path. Mark untested scripts `UNTESTED`.
4. Platform versions from Step 2.5, if any, each labeled with its target path.
5. Trigger test results (labeled "Simulated" or "Tested in fresh session"):

| Prompt | Should trigger | Matched / Blocked By (Keyword or Rule in Description) |
|---|---|---|
| ... | Yes / No | Pass - matched by "..." / blocked by "Do NOT use for..." |

6. Coverage check: one bullet per Output Format section mapped to its producing step, plus any gaps fixed.

**Mode B:**
- **Turn 1:** Score with arithmetic, issues by severity, proposed diff, and the Step 3 validation check (Trigger table + Coverage check) on the proposed diff. Stop for approval.
- **Turn 2:** The full revised `SKILL.md` (and saved confirmation if requested).

## Gotchas

- **Four backticks for the outer block**, so inner three-backtick code blocks do not break chat rendering.
- **YAML colons:** an unquoted value containing `: ` breaks the parser. Always quote the description or use a `>` folded scalar.
- **Undertriggering is more common than overtriggering.** Include concrete user phrasings and synonyms in the description, not just an abstract summary.
- **No padding:** if a section has nothing specific to say, remove it. Empty structure costs tokens and dilutes the instructions that matter.
- **Platform formats drift.** Treat `references/platforms.md` as a starting point and prefer the user's current docs when they conflict.