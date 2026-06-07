---
name: skill-creator
description: Create new skills, modify and improve existing skills, and measure skill performance. Use when users want to create a skill from scratch, edit, or optimize an existing skill, run evals to test a skill, benchmark skill performance with variance analysis, or optimize a skill's description for better triggering accuracy.
---

# Skill Creator

Process: decide intent → write draft → run test prompts → evaluate (qualitative + quantitative) → rewrite → repeat.

## Creating a skill

### Capture Intent
1. What should this skill enable Claude to do?
2. When should it trigger?
3. Expected output format?
4. Do we need test cases?

### Write the SKILL.md
- **name**: Skill identifier
- **description**: When to trigger + what it does. Make it slightly "pushy" to avoid undertriggering.
- **body**: Instructions

### Test Cases
Come up with 2-3 realistic test prompts. Save to `evals/evals.json`. Run with-skill AND baseline subagents in the same turn.

### Evaluation Loop
1. Grade assertions → aggregate benchmark → analyst pass → launch eval viewer
2. User reviews → reads feedback.json → improve skill → repeat
3. Stop when user is happy, feedback is empty, or no meaningful progress

## Improving a skill
- Generalize from feedback — avoid overfitting to examples
- Keep the prompt lean — remove what isn't pulling weight
- Explain the why — smart models respond better to reasoning than rigid MUSTs
- Look for repeated work across test cases → bundle as scripts/

## Description Optimization
After skill is stable, run the trigger eval optimization loop:
```bash
python -m scripts.run_loop --eval-set <path> --skill-path <path> --model <model-id> --max-iterations 5
```
Apply best_description to frontmatter.

## Cowork-specific
- Use `--static <path>` for eval viewer (no browser)
- GENERATE THE EVAL VIEWER before evaluating yourself
- Packaging: `python -m scripts.package_skill <skill-folder>`
