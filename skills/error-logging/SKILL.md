---
name: error-logging
description: Appends problems encountered during work to `agent-errors.log` in the working directory so the next agent inherits a warning trail. Use this skill whenever a tool errors, a build/test/lint/type-check fails, an MCP call returns something unexpected, an assumption has to be revised mid-task, a workaround is applied because the intended approach did not work, or any unexpected tooling or codebase behavior occurs, even if the issue is fixed afterwards. Trigger on the event itself, not on a user request to log.
---

# Error Logging

This skill maintains a forward-facing warning trail for the next agent that picks up this work. Whenever something goes wrong during a task, append a short, structured entry to `agent-errors.log` in the working directory (the directory you were invoked in, usually the repo root). Do not search around for a "more correct" root, consistency beats cleverness here.

The point is not a debugging journal for the current session, it is a heads-up for the *next* agent: "here is a problem I hit, here is what I now know about it."

## When to log

Append an entry whenever any of these happen during work:

- A tool call (Bash, MCP, anything) errors, exits non-zero, or returns an unexpected result
- A build, test, lint, or type-check fails, even if you fix it afterwards
- An assumption you started the task with has to be revised mid-task (e.g. wrong node ID format, instance vs. source component, wrong file path, API shape differs from expectation)
- You apply a workaround because the intended approach did not work
- Tooling behaves unexpectedly (python, npm, the codebase itself, a missing node, empty metadata, etc.)
- Anything you would want to warn the next agent about

If a single task surfaces several distinct problems, write several entries, one per problem.

## When NOT to log

Skip the log for:

- Successful steps and routine progress
- Expected MCP behavior
- Tasks that complete with no problems (in that case, write nothing at all; an empty append is worse than no append)

The signal-to-noise ratio matters. A log full of "everything went fine" entries trains the next agent to ignore it.

## Format

One entry per distinct problem. Two problems in one task means two entries, back to back:

```
--- 2026-10-04 19:03 ---
Task: <one-line summary>
Problem: <what went wrong, exact error message if any>
Cause: <root cause if identified, or "unknown">
Resolution: <how it was fixed, or "unresolved" / "worked around by X">

--- 2026-10-04 19:11 ---
Task: <same task or the follow-on>
Problem: <the second, distinct problem>
Cause: <if it only surfaced because of the first problem, say so here>
Resolution: <...>
```

Never fold two problems into one entry as `Problem 1:` / `Problem 2:`. A cascade still gets its own entry, with the link back stated in its `Cause:` line.

The four fields are fixed: do not add, drop or reorder them. A filled-in entry:

```
--- 2026-10-04 19:03 ---
Task: Add pagination to the orders list
Problem: `npm test` failed with "Cannot find module '@testing-library/dom'"
Cause: Peer dependency missing after upgrading @testing-library/react
Resolution: Worked around by running `npm install -D @testing-library/dom`
```

**Take the timestamp from the clock**, in the same call that writes the entry (`Get-Date` / `date`, see below). You are told today's date but never the current time, so a hand-written time is a guess, and `00:00` placeholders make entries impossible to order against git history.

**Write entries in English**, whatever language the session is in. Quote exact error messages verbatim, including any non-ASCII they contain. Vague paraphrases ("something failed") are nearly useless to the next agent.

## Before appending

Grep the log for the problem first: `grep -i '<keyword>' agent-errors.log` (PowerShell: `Select-String -Pattern '<keyword>' agent-errors.log`). If an entry already describes it, still append, but make `Problem:` a back-reference ("same MSYS2 path rewriting as 2026-08-20") plus only what is new about this occurrence. Re-describing a known problem from scratch buries the signal under repetition.

## Rules

- **Always append.** Never overwrite or truncate the log. Existing entries are not yours to revise.
- **Create the file on first write** if it does not yet exist.
- **No em dashes**, per the global writing rule.

## How to append

Two commands per entry: one for the header, so the timestamp is real, and one for the body, so nothing inside it gets expanded. Use whichever shell the session is already in.

PowerShell:

```powershell
$ts = Get-Date -Format 'yyyy-MM-dd HH:mm'
$body = @'
Task: <summary>
Problem: <what went wrong>
Cause: <root cause or "unknown">
Resolution: <fix or "worked around by X">
'@
[IO.File]::AppendAllText(
  (Join-Path $PWD 'agent-errors.log'),
  "`n--- $ts ---`n$body`n",
  (New-Object System.Text.UTF8Encoding $false)
)
```

Three things in there not to "simplify":

- `AppendAllText` with `UTF8Encoding $false`, not `Add-Content -Encoding utf8`: on PowerShell 5.1, `utf8` means UTF-8 *with BOM*, and that BOM lands at the top of the log.
- `Join-Path $PWD`: .NET resolves a relative path against the process working directory, which is not PowerShell's `$PWD`. A bare `'agent-errors.log'` silently writes somewhere else.
- `@'...'@` (literal) for the body, `"..."` (interpolating) for the wrapper. Swapping them either loses the timestamp or expands `$vars` and backticks sitting inside your error text.

Bash:

```bash
printf '\n--- %s ---\n' "$(date +'%Y-%m-%d %H:%M')" >> agent-errors.log
cat >> agent-errors.log <<'EOF'
Task: <summary>
Problem: <what went wrong>
Cause: <root cause or "unknown">
Resolution: <fix or "worked around by X">
EOF
```

The heredoc stays quoted (`<<'EOF'`) so that `$`, backticks and quotes in a pasted error message survive intact. A quoted heredoc cannot interpolate `$(date)`, hence the separate `printf`.

The leading blank line in each entry keeps entries visually separated when the file is read top to bottom.

## Why this matters

LLM agents start each session cold. Without a trail, every new agent walks into the same problems, re-discovers the same broken assumption, re-applies the same workaround, re-debugs the same flaky tool. A short, honest log of problems encountered is one of the cheapest, highest-leverage things you can leave behind.
