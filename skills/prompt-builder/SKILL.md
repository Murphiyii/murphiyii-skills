---
name: prompt-builder
description: Rewrite a prompt for a specific Claude model using Claude prompting best practices. Use whenever the user shares a prompt and wants it improved or adapted, even if they only say "improve it".
---

# Prompt builder

Rewrite a prompt to be more effective with a **specific Claude model**, then return it ready to copy and paste.

The right rewrite depends on which model the prompt targets. Newer models (the 5-series: Fable 5.1, Opus 5.5, Sonnet 5.5 and their 5.0 predecessors) follow instructions literally and respond to brief direction, so piling on rules degrades their output; older models often need more explicit, emphatic guidance. Adapt to the chosen model, not to a generic "Claude."

## Process

1. **Separate the prompt to adapt from the instructions about the adaptation.**
   - The prompt to adapt is whatever the user put inside a fenced code block or `<prompt>…</prompt>` tags. Everything else in the message is instructions to you (target model, length limits, things to keep or drop, tone), not part of the prompt.
   - If nothing is delimited, infer the split yourself: the verbatim prompt body vs. imperatives addressed to you ("adapt this for X", "make it shorter", "keep the JSON examples"). Treat those imperatives as constraints on the rewrite. Notify the user if you had to infer the split. **Never fold them into the rewritten prompt.**

2. **Ask which model the prompt is for.** Use `AskUserQuestion` with exactly these four choices:
   - **Fable 5.1** : Claude Fable 5.1 (this reference also covers Mythos 5.1)
   - **Opus 5.5** : Claude Opus 5.5
   - **Sonnet 5.5** : Claude Sonnet 5.5
   - **Older model** : Claude Fable 5 / Mythos 5, Opus 5, Sonnet 5, Opus 4.8, Opus 4.7, Opus 4.6, Sonnet 4.6, or Haiku 4.5

   Skip the question only if the user already named the target model. Do not adapt the prompt until the model is known. If you had to **infer** the split in step 1 (nothing was delimited), state your interpretation in the message accompanying this question ("I read X as the prompt and Y as instructions") so the user can correct it before you adapt. If you skipped the question and still had to infer the split, state that interpretation in one line before the rewrite instead; it is the only preamble the Output rules allow. If the user delimited the prompt, no confirmation is needed.

3. **Always read `references/prompting-best-practices.md` in full, whatever model was chosen.** Then also read the matching model-specific reference in full before rewriting:

   | Chosen model | Also read this file |
   |---|---|
   | Fable 5.1 | `references/prompting-fable-5.1-best-practices.md` |
   | Opus 5.5 | `references/prompting-opus-5.5-best-practices.md` |
   | Sonnet 5.5 | `references/prompting-sonnet-5.5-best-practices.md` |
   | Older model | (none, the general guide is the only reference) |

   Read each file with a single Read call and no `offset` or `limit`. If the output is truncated, keep reading until the end of the file.

   The model-specific files build on the general guide, so both are required. Where they conflict, the model-specific file wins.

   `prompting-best-practices.md` is the current cross-model guide: it opens with a table pointing to the three model-specific files, then gives techniques for all current models, with inline notes that name a specific model (for example Opus 5 self-verifying, Fable 5.1 formatting less, Opus 4.6 overtriggering). Apply an inline note only to the model it names. The model-specific files describe differences from each model's 5.0 predecessor, so the guide's inline notes on Fable 5, Opus 5, or Sonnet 5 still hold for the newer model unless its own file says otherwise. When the target is **Older model**, use the general principles plus any inline note naming that exact model, and skip notes about the 5.1 / 5.5 models.

4. **Rewrite the prompt**, applying the baseline techniques below plus the model-specific guidance from the reference you read. Apply the user's adaptation instructions as constraints, not as content. Then output it per the Output rules.

## Baseline techniques (all models)

Work through these; use judgment about which actually help this prompt. Not every one applies. **The one exception is XML structure, which is mandatory on every adapted prompt (see below).**

### Clarity and directness
- Replace vague verbs ("handle", "deal with", "look at") with precise ones ("validate and return an error", "extract the top 3 items").
- Turn suggestions into direct commands: "Can you suggest changes?" → "Make these changes."
- Give the reason behind a constraint. Models generalize better from an explained rule than a bare one: "Never use ellipses, this will be read aloud by a TTS engine that can't pronounce them" beats "NEVER use ellipses."
- If you want above-and-beyond output, say so explicitly.
- Break ordered or must-be-complete instructions into a numbered list.

### XML structure (mandatory, always)
- Wrap the adapted prompt's parts in XML tags on every generated prompt, for every model and every prompt, including single-sentence ones. This overrides the "use judgment" framing above and any per-model brevity caution below: XML tagging is structural, not prescriptive reasoning scaffolding, so it applies even where the model-specific guidance favors brief instructions.
- Wrap each distinct part in its own descriptive tag so the model can't confuse them: `<instructions>`, `<context>`, `<input>`, `<constraints>`, `<output_format>`, and `<example>` / `<examples>`, etc. Pick tag names that match the prompt's actual sections.
- Even a short, single-section prompt must be wrapped in at least one descriptive tag (e.g. `<task>…</task>`).
- Place long documents or data **before** the query and instructions; this materially improves quality.
- Keep template placeholders labeled: `{{VARIABLE_NAME}}`.

### Output format
- State what the output should look like when the prompt doesn't.
- Prefer positive instructions over negations: "Write in flowing prose paragraphs" beats "Don't use bullet points."
- The prompt's own formatting style influences the output's; a heavily bulleted prompt tends to elicit a bulleted answer.
- Add a role if it would focus the response.

### Don't over-prompt
This is the change that most often *fixes* a modern prompt. Newer models follow instructions literally and respond to brief direction; emphatic stacking ("CRITICAL", "You MUST", repeated all-caps) now causes overtriggering and worse output. Prefer the lightest instruction that produces the behavior. The model-specific reference says where each model's baseline sits, so defer to it before adding anything.

### Agentic and tool-use patterns
Apply only when the prompt involves tools, autonomous agents, or multi-step / long-horizon work. The specific patterns (parallel tool calls, bias-to-action vs. confirm-on-risky-action, state tracking across context windows, subagent control, thinking calibration) differ by model. Take them from the reference you read, not from memory.

## Highest-impact per-model cautions

The things most likely to be wrong if you adapt from memory. The reference covers the rest.

- **Fable 5.1, Opus 5.5, Sonnet 5.5:** Replace any instruction to write out or show reasoning in the response (including chain-of-thought in `<thinking>` tags) with a request for a short explanation or a summary of actions taken; reasoning-in-response prompts can be declined as `reasoning_extraction`. Remove narration-suppressing lines such as "hold all findings for the final response".
- **Fable 5.1:** It already formats less than earlier models; replace anti-markdown blocks with a rule saying when lists and headers fit.
- **Opus 5.5:** Keep Opus 5's verification instructions ("double-check your answer") stripped; it self-verifies. Wrap text the user pasted from elsewhere in `<pasted_content id="…">` tags and add the reference's system note.
- **Sonnet 5.5:** Remove "don't think" / "think less" rules; they don't reduce thinking and can leak internal XML tags into output.
- **Older model:** Fable 5, Opus 5, and Sonnet 5 follow the brief-direction rule above and share the `reasoning_extraction` risk on Fable 5 and Opus 5. The 4.x models and Haiku 4.5 tolerate, and sometimes need, more explicit and emphatic instruction, and manual chain-of-thought (reasoning in tags, answer in `<answer>`) is fine when thinking is off.

## Output rules

- No preamble before the rewrite. Don't write "Here is the improved prompt." The single exception is the one-line split interpretation from step 2, and only when you skipped the model question.
- Output the adapted prompt inside a fenced markdown code block so it is ready to copy and paste. Use one block; if the input had both a system prompt and a user message, use one labeled block for each (`### System prompt`, `### User message`).
- Inside the fenced code block, wrap each distinct section in a descriptive XML tag (see "XML structure" above).
- If the rewrite depended on something the prompt doesn't state, such as effort level or whether the agent runs unattended, add one line after the code block naming it (for example: "Assumed: runs unattended at medium effort."). Skip this line when nothing was assumed.
- End with a single line stating you are done and ready to receive a new prompt to adapt (for example: "Done. Send me the next prompt to adapt whenever you're ready."). Output nothing else after the rewrite.
- Preserve the original intent exactly. Improve the writing, not the goal.
- Keep any part that already follows best practice as-is.
- Don't add agentic patterns to a prompt that has nothing to do with tools or agents.
- Match the output to the input structure: system prompt in → system prompt out; user message in → user message out; both in → output both, labeled as above.
