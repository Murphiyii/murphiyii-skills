---
title: Prompting Claude Sonnet 5.5
description: "Prompting patterns specific to Claude Sonnet 5.5: effort, initiative and scope, running without up-front thinking, JSON output, progress updates, tool use, mid-turn messages, coding verification, visual inputs, and refusals."
---

This guide covers the prompting patterns specific to Claude Sonnet 5.5.

## Contents

* Calibrate effort
* Steer initiative and scope
* Running without up-front thinking
* Reasoning tasks with JSON output
* User-facing progress updates
* Tool use in chat and knowledge work
* Mid-turn user messages
* Verification on coding tasks
* Tools for complex visual inputs
* Safeguard refusals

## Calibrate effort

Lower effort changes how Claude Sonnet 5.5 finishes agentic work. At `low`, it keeps its thinking short and can skip verifying a change. At `low` and `medium`, on long agentic tasks, it's more likely to stop and check in with the user before it finishes.

Asking it in the system prompt to think less doesn't reliably reduce its thinking.

## Steer initiative and scope

How far Claude Sonnet 5.5 goes on its own depends on the effort level and the request. At lower effort, it sometimes checks in before a coding task is done. At higher effort, or on an open-ended request, it can do more than you asked. Steer it with instructions in your system prompt.

**Carrying work through.** On agentic coding tasks at `low` and `medium` effort, the model sometimes checks in before the work is done. It might pause to confirm a plan, ask a question it could answer itself, or stop after one part of a multipart task to ask whether to continue. To keep the model working, add this to your system prompt:

```text wrap
Keep working until everything the user asked for is done, and only stop to ask when you can't go on without the user or before a risky step.

When the work the user asked for is done and checked, stop and report. Don't add features, tests, files, docs or refactors that weren't asked for. If you think one would help, mention it at the end instead of doing it.
```

With this prompt, the model carries more of the work through at `low` and `medium` effort, so sessions at those levels run longer and cost more. The prompt doesn't replace your own rules about risky or irreversible actions. Keep those rules in your system prompt.

**Unrequested additions when coding.** The model tends to add tests, documentation, and small supporting files that fit your repository's conventions, even when you don't ask for them. It does this at every effort level, and more at higher effort. The requested change itself stays close to what was asked. Most teams will welcome this. If you prefer changes limited to what was explicitly requested, add only the second paragraph of that prompt, which starts "When the work the user asked for is done". At `xhigh` and `max` effort, that paragraph reduces these additions and makes changes smaller overall.

**Thoroughness at `xhigh` and `max` effort.** At these levels the model is especially thorough. After it finishes a task, it can start its own rounds of review and verification, sometimes with subagents if your harness provides them. It can also make related fixes it noticed along the way. This takes more time and tokens, and it's rare at `high` effort or below. If you do want that extra thoroughness of these effort levels, but want to direct it to the task itself, add this to your system prompt:

```text wrap
When the work the user asked for is done and its checks pass, stop and report. Don't start extra rounds of review or hardening on your own, and don't launch reviewer sub-agents unless the user asked for a review. If you think a deeper review is worth doing, say so at the end.
```

In testing on coding tasks at `max` effort, this stopped the model from launching reviewer subagents and cut session cost by about a third, with no change in quality. It makes self-started review rounds by the main agent less frequent but doesn't remove them entirely.

**Open-ended requests.** When a request is open-ended, for example "show me what you can do with this", the model can start building a presentation, report, or video when you only wanted ideas. If you want ideas or a plan first, say so in the request, or add this to your system prompt:

```text wrap
When the user asks for ideas, options or a plan, give them that and stop. Don't start building or changing anything until they say to go ahead.
```

## Running without up-front thinking

If you run without up-front thinking, remove any instruction that tells the model not to think. Such instructions make it more likely that the model writes internal XML tags in its visible output.

## Reasoning tasks with JSON output

This section applies when you ask Claude Sonnet 5.5 for a JSON answer to a task that needs a few steps of working out. Examples include totaling figures from a document, applying a rule, or ranking items. On tasks like these, the model often answers without thinking first, particularly at `low` and `medium` effort.

With structured outputs, the response text holds only the JSON, so the model can work the problem out only in its thinking. When it skips thinking, it can be less accurate on these tasks.

**Ask the model to think first.** Add this line to the end of your system prompt:

```text wrap
Think the problem through before you answer.
```

With this line, the model more often thinks before it answers. At `high` effort, the line brings accuracy close to what the model reaches at `xhigh`, for a modest increase in output tokens. At `low` and `medium` effort, it raises accuracy, though not to what the model reaches at `high`, and the increase in output tokens is larger.

In testing, splitting the request in two, one request for the answer and one for the JSON, led to high answer accuracy and JSON compliance, but at very high cost and latency.

If you can't use structured outputs, ask for JSON in the prompt instead. The model then often works the problem out in the response text and writes the JSON at the end. The JSON usually holds the right answer, but a parser that expects the whole response to be JSON fails. The model occasionally writes a draft before its final JSON.

## User-facing progress updates

Between tool calls, Claude Sonnet 5.5 writes user-facing notes about what it just found and what it's doing next. This matters most in chat interfaces and other products where the user follows the model's work in real time.

Sometimes the model needs to show the user exact text partway through a long turn, such as a code snippet or a question it needs answered. For that case, give it a simple tool for sending the user a message. Tell the model to use that tool only for such content.

Remove older instructions such as "hold all findings for the final response". If you then want updates at predictable points, for example a line on what the model is about to do before its first tool call and a short recap at the end, say so in the system prompt. The model follows instructions like this. Updates at set points help most in human-in-the-loop work.

If long tool-calling turns still go quiet for longer than you want, your harness can prompt an update. Have it count consecutive tool-calling steps that send the user no text or progress update. After several in a row, for example five, append a one-turn reminder after the latest tool results, with text like this:

```text wrap
The user hasn't heard from you in a while — say in a few words what you're doing, then continue.
```

If the turn stays quiet, stop sending reminders after the second or third. Frequent harness text after tool results can make the model suspect a prompt injection, as Mid-turn user messages explains. At `high` effort, with a tool for sending the user a message available, the reminder leads the model to update the user more often and shortens its longest silent stretches, with no measurable change in task quality.

## Tool use in chat and knowledge work

On chat and knowledge-work tasks, Claude Sonnet 5.5 sometimes answers from its training knowledge when a web search would catch details that have changed. Examples include what is allowed, required, or charged.

First, check your prompt for language that discourages tool use, such as "only use tools when strictly necessary" or "minimize tool calls", and remove it. Then, if your product gives the model a search tool, add this to your system prompt:

```text wrap
Use the search tool to check specifics that may have changed since your training, such as what is allowed, required or charged, even when you feel confident. For researched work such as a report or a comparison, gather current sources rather than writing from your training knowledge.
```

This matters most for research and support products, where answers depend on current details.

## Mid-turn user messages

Claude Sonnet 5.5 is trained to resist indirect prompt injection, meaning malicious instructions that arrive through tool results and other content it reads during a task. Sometimes it treats a genuine user message as a possible injection. Suppose a message the user typed mid-task reaches the model as a mid-conversation system message placed directly after a tool result, or inside a `tool_result` block. The model can then tell the user that the tool result contained text posing as a message from them, and ignore the message or ask the user to confirm it.

A token countdown that your harness adds after every tool result can cause this. So can letting users send messages while the model is partway through a multistep turn, or having your harness add instructions or context after the tool results on every step. In each case, text arrives right after the tool results. With a countdown or per-step instructions, that can happen on every tool call. An occasional one-turn reminder, like the one in User-facing progress updates, arrives far less often. If you see this reaction to a reminder of your own, send the reminder less often. To avoid the misread:

* Never put user text inside a `tool_result` block. The model misreads that placement most often.
* Deliver mid-turn user input as a user turn. Append the user's words as a text block in the user message that carries the `tool_result` blocks, after the last `tool_result`.
* Keep harness notices, such as reminders, in a separate mid-conversation system message after the user's words. Never put a notice and the user's words in the same block.
* In interactive sessions where users can type mid-turn, don't add your own token or budget countdown after tool results.

## Verification on coding tasks

On agentic coding tasks, Claude Sonnet 5.5 generally checks its work before it reports a change as done. At `low` effort, though, it sometimes reports a change as done without running a check that exercises it. For example, it might skip the project's tests because the project's dependencies aren't installed.

If you see changes reported as complete without test or build output in the transcript, add this paragraph, or one like it, to the system prompt. At `low` effort, it makes skipped or superficial checks rare, with no measurable change in task quality and only a slightly higher cost per task:

```text wrap
When you change code that can be run, built, or type-checked, run a real check that exercises the change before reporting it done: the project's tests, type-checker, or build, or the changed command itself. A syntax-only check, or a check command that failed to start, does not count; if all that is missing is the project's declared dependencies, install them with its own package manager and lockfile (e.g. npm install, pip install -r requirements.txt), never via sudo or the system package manager, unless told not to. Only if no real check can run here, say which one you did not run and why instead of reporting the change as done.
```

## Tools for complex visual inputs

For dense charts and technical drawings, give Claude Sonnet 5.5 a way to crop, zoom, or run code on the image. With such tools, the model reads these inputs markedly more accurately. On charts, the tools help at every effort level. On technical drawings, they help only from `high` effort up, and most at `xhigh` and `max`. For charts, adding tools helps more than raising effort: in testing, with tools at `high` effort, the model read charts more accurately than without tools at `max` effort, at a fraction of the cost.

## Safeguard refusals

Claude Sonnet 5.5 runs safety classifiers that can decline a request.

If your prompts ask the model to include its reasoning in the response, remove those instructions, because they invite `reasoning_extraction` declines. You can still ask for a short explanation of the answer or a summary of the actions taken.