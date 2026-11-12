---
description: Turn a hunting or investigation question into a query grounded in your log-source docs, run it, and summarize what it found. Read-only; never changes rules.
argument-hint: "<question in plain words>  |  <a query to run as-is>"
---

Answer a question from the team's telemetry. This command is outside the rule lifecycle: it
never creates or edits a rule. If a hunt turns up something a rule should catch, say so and
point to `draft-rule`.

This command is **stateless**: re-read `DETECTION.md` every time.

## 1. Load the environment

Read `DETECTION.md` (rules-repo root, or `$DETECTION_MD`). You need **Query**. You'll also use
**Query language** and **Log sources** if they're present.

If there's no **Query** section, you can still write the query: give it to the user to run, and
then interpret the results they paste back.

## 2. Pin down the question

Get these clear before writing anything. Ask only for what's missing:

- **What** behaviour or entity you're looking for (a user, host, IP, hash, or an action pattern).
- **Where**: which log source(s) would show it.
- **When**: the time window. Default to the last 24 hours, and say so.

If the user handed you a ready-made query, skip to step 4 and run it as written. Mention any
field names that don't match the log-source doc, but don't rewrite their query unless they ask.

## 3. Write a grounded query

Follow the `query-grounding` skill. In short:

- Read the **log-source doc** for each source first, and use only the fields it lists, exactly as
  spelled and cased. With no doc, run a tiny sample query first (one source, a short window, a
  few events) to see the real field names, and say the names were taken from samples.
- Read the **query-language notes** for syntax pitfalls.
- Scope to the source first, filter narrowly, and aggregate last. Prefer returning a summary
  (counts by user, host, action) over raw events, unless the user wants the events.
- For a wide window on a high-volume source, say roughly what the query will cost and offer a
  narrower first pass.

Show the query before running it.

## 4. Run it

Run it the way **Query** describes: pass the query as a file if the tool supports that, and read
large results from the file the tool writes rather than inline.

- Missing credentials → point to `/detection-harness:setup`. Never ask for a secret in the chat.
- A query error → read the raw error and fix exactly what it says, rather than rewriting from
  memory. Re-run. If it fails again, show both attempts and stop.
- Zero rows → work through the zero-rows checklist in `query-grounding` before concluding
  anything.

## 5. Summarize

- Lead with the answer to the question, then the evidence: counts, top entities, the time range
  covered, and a few representative events (trim tokens and secrets).
- Say what the query **can't** show: sources not checked, gaps in the window, fields that were
  null or missing.
- **No results** means no matches in this window and this source. It isn't proof the activity
  didn't happen. Say what you checked.

## 6. Suggest the next step, if there is one

- Found behaviour that should alert, with no rule covering it → check the `catalog` skill for an
  existing rule, then suggest `/detection-harness:draft-rule <what to detect>`.
- Found an existing rule that's noisy or misses this → suggest
  `/detection-harness:draft-rule tune <path> <what's wrong>`.
- Otherwise, stop after the summary.
