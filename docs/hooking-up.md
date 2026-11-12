# Hooking up your environment

This repo is a set of skills and commands. It doesn't include a validator, a query client, or a SIEM.
You bring those. The skills walk the agent through the lifecycle (**draft → validate → test →
submit**) and call your tools at each step.

To connect your tools, write **one markdown file** in your rules repo, `DETECTION.md`, that tells
the agent how your environment works. The agent reads it at the start of every command, so it
works the same way on every machine and for everyone on the team.

`/setup` drafts this file for you by looking at your repo and asking about anything it can't find.
You can also copy [`examples/DETECTION.md`](../examples/DETECTION.md) and edit it.

## What `DETECTION.md` needs

Use these headings so the commands can find each section. Only **Rules** and **Validate** are
required. Leave out the rest until you need them.

### Rules (required)
- Where rule files live, and the naming convention.
- The rule format, in plain words (Sigma, KQL in YAML, SPL saved searches, your own schema…).
- A path to one good existing rule the agent should copy the shape from.

### Validate (required)
- The exact command that validates one rule, e.g. `make lint RULE=<path>`.
- How to tell pass from fail. Best case: exit `0` for pass and `3` for violations, with the
  violations listed. Any readable output works, but say what success looks like.
- Which violations the agent may fix on its own (format, missing fields), and which need a human
  (logic, MITRE mapping, duplicate rules).

### Query (optional: used by `/test-rule` and `/hunt`)
- The command that runs a query over a time range, and where the query text goes.
- How to get a query out of a rule file, if the rule isn't itself a runnable query.
- Credential setup, and what the error looks like when credentials are missing.
- The output to expect, and where large results land.

Without this section, `/test-rule` gives you the query and asks you to run it, then makes its
verdict from the count you paste back.

### Query language (optional, strongly recommended)
- A path to your query-language notes: syntax pitfalls, house style, required enrichments or
  output fields. This can be an existing wiki page exported to markdown.

### Log sources (optional, strongly recommended)
- A path to a folder with one doc per log source, following
  [`log-source-template.md`](log-source-template.md).
- The agent reads the matching doc **before** it writes any query. Field names should come from
  your real telemetry, not from the model's memory. That's the biggest quality lever in this repo.

Without these docs, `/draft-rule` asks for one real sample event before writing the query, and
marks the field names as unverified.

### Submit (optional)
- Base branch, branch naming, and the command that opens a review: `gh pr create`,
  `glab mr create`, or whatever your git host uses.
- Who reviews, if your git host doesn't assign reviewers automatically.

Without this section, `/submit-rule` validates the rule, commits it on a branch, and stops.
**The agent never merges.** A human merging the change is the deploy.

### Severity and volume (optional)
- Your severity levels and the maximum alerts per day you'll accept at each one. The numbers
  depend on your team's capacity; [`severity-volume.md`](severity-volume.md) explains how to
  pick them.
- `/test-rule` compares measured volume against this table. Zero results is reported as a
  **suspicious** result, not a pass.

### Catalog (optional)
- Where per-rule operational data lives, and how to read it: fires in the last 30 days, last
  fired, TP / benign-TP / FP counts, last reviewed, last tuned. A command, a file path or an
  export all work.
- Which key joins it to the rules repo (rule ID or name).

The `catalog` skill uses this for review questions ("what's overdue?", "what never fires?") and
reads the history before proposing a tune. Without it, the skill answers coverage questions from
the rules repo alone.

## Making your own tools agent-friendly

You don't need new tools, but small changes to the ones you have help a lot. See
[`agent-safe-scripts.md`](agent-safe-scripts.md). The short version:
- Use distinct exit codes: `0` ok, `1` upstream error (raw message on stderr), `2` missing
  config or credentials, `3` validation violations.
- Never prompt for input, and stay read-only unless the command exists to write.
- Offer `--json`, and write large output to a file and print the path.

## Optional: enforce validation with a hook

The skill asks for validation, your validator does the checking, and CI enforces it. To also
enforce it during local agent sessions, add a Claude Code `PostToolUse` hook that runs your
validate command whenever a rule file is written. The hook config is a few lines of JSON that
call your own command. See the example in
[`examples/.claude/settings.json`](../examples/.claude/settings.json): it runs the validate
command on any file written under `rules/`. Exit code `2` sends the violations back to the agent.

## Checking the hook-up

Run `/setup` again. It re-reads `DETECTION.md`, runs the validate command on your template rule,
and runs a tiny query if the Query section exists. It reports what works and what's missing.
