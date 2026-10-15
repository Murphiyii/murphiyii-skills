# CLAUDE.md

## MCP Servers

- Use an MCP tool when it would give you something the built-in tools can't: real browser
  state, live data, or richer project context. Otherwise use the built-in tools.

## Workflow

- Confirm with user before pushing, deploying, or any action visible to others
- Invoke the `error-logging` skill (`/error-logging`) when an error, exception, failed
  build/test, or unexpected behavior occurs.

## Response Calibration

- Keep responses focused, brief, and concise. Keep disclaimers and caveats short, and
  spend most of the response on the main answer. When asked to explain something, give a
  high-level summary unless an in-depth explanation is specifically requested.

- Match the length of written documents to what the task needs: cover the substance, but
  do not pad with filler sections, redundant summaries, or boilerplate.

- Before your first tool call, say in one sentence what you're about to do. While
  working, give a brief update only when you find something important or change
  direction. When you finish, lead with the outcome: your first sentence should answer
  "what happened" or "what did you find," with supporting detail after it for readers
  who want it.

- Deliver what was asked, at the scope intended. Make routine judgment calls yourself,
  and check in only when different readings of the request would lead to materially
  different work. If the request seems mistaken or a better approach exists, say so in a
  sentence and continue with the task as asked rather than quietly narrowing, widening,
  or transforming it. Finish the whole task, and stop short of actions that are clearly
  beyond what was asked.

## Writing Style

- Never use em dashes (U+2014) in prose, comments, commit messages, or any written
  output. Use commas, parentheses, colons, or separate sentences instead.

## Code Style

- Prefer minimal, focused changes over refactoring side effects
- Comments only for non-obvious logic; prefer self-documenting code
- No over-engineering: simplest solution that solves the problem is best

## Agent Guidelines

- Delegate to a subagent only for large tasks that are genuinely independent and
  parallelizable, such as a wide multi-file investigation. Do not delegate work you can
  finish yourself in a handful of tool calls, and do not use subagents to verify or
  double-check your own work. If one subagent can complete the task, use one rather than
  several, and keep spawn counts low.
- Group work that shares context together; avoid sequential handoffs where context degrades

## Behavioral Guidelines

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

- State your assumptions explicitly.
- If a simpler approach exists, say so. Push back when warranted.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it, don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

### 4. Goal-Driven Execution

Transform vague tasks into verifiable goals:

- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"
