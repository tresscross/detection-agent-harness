---
name: detection-engineering
description: Use when the user is writing, changing, testing or shipping a detection rule — "new rule", "write a detection for X", "tune this rule", "too noisy", "false positives", "lower the severity", "map this to MITRE", "submit/deploy this rule". Routes the work through the detection-harness lifecycle (draft → validate → test → submit) against the team's own rules repo, instead of ad-hoc edits or SIEM UI changes.
---

# Detection engineering lifecycle

This skill routes detection work through four gates. Each gate is a command. The team's own
tools do the deterministic work, and you do the judgment.

| Step | Command | What it settles |
|---|---|---|
| 1 | `/detection-harness:draft-rule` | Writes a rule (new or tuned) grounded in real field names |
| 2 | `/detection-harness:validate-rule` | Checks that the rule passes the team's linter |
| 3 | `/detection-harness:test-rule` | Measures alert volume and checks it fits the severity |
| 4 | `/detection-harness:submit-rule` | Opens a review request. A human merges it |

`/detection-harness:setup` writes or checks `DETECTION.md`. `/detection-harness:hunt` runs
one-off queries outside the lifecycle.

Three supporting skills hold the knowledge the commands rely on:
- **`query-grounding`**: how to write any query against real fields, test it short, and read
  errors and empty results.
- **`log-source-docs`**: how to build and refresh a per-source field reference from real telemetry.
- **`catalog`**: coverage questions, reading a rule's history before tuning, and review and
  retirement.

Every command is **stateless**. It reads `DETECTION.md` and the rule file each time and
relies on nothing from earlier in the conversation. On success, each command names the next one.

## When to use it

Route here when the user says or means any of:

- "new rule", "write a detection for…", "can we alert on…"
- "tune", "retune", "too noisy", "false positives", "exclude X", "this keeps firing"
- "raise/lower severity", "is this the right severity"
- "map to MITRE", "what technique is this"
- "submit", "ship", "deploy", "push this rule"

**Tuning is not a shortcut.** A tune changes detection logic, so it goes through the same
gates as a new rule: draft (edit the existing file), validate, test, submit. Don't edit a rule
and go straight to submit, and don't advise the user to change it in the SIEM UI.

## Before anything: find `DETECTION.md`

Look for `DETECTION.md` at the root of the rules repo. That is the current repo, or the
path in `$DETECTION_MD` if it is set. It describes this team's rule format, validator, query
tool, docs, git workflow and severity table.

- **Missing** → stop and suggest `/detection-harness:setup`. Don't guess at the team's tooling.
- **Present** → each command reads only the sections it needs. Only **Rules** and **Validate**
  are required. When an optional section is missing, the command says what it can't do
  and falls back to asking the user. It never makes up a tool or a command.

## Principles you must hold to

- **Scripts settle questions with one correct answer; you settle the rest.** Pass/fail on the
  linter, alert counts and branch creation come from the team's tools. Whether the logic
  catches the behaviour, whether an exclusion is too broad, and which severity fits are
  judgment calls: reason about them out loud and let the user decide.
- **Field names come from documents, never from memory.** Read the log-source doc (or a real
  sample event) before writing a query. A rule built on a field that doesn't exist is silent
  forever.
- **Zero results is a warning sign, not a pass.** A quiet rule and a broken rule look the same.
- **Never merge, approve, or push to the base branch.** A human merging the change is the
  deploy.
- **Never weaken a check to get past it.** Don't add skip flags, delete required fields or
  loosen thresholds just to make validation or testing pass.
- **Ask, then do, then enforce.** You ask, the team's tools do the work, and their CI enforces the result.

## When NOT to use this skill

- The user is investigating or hunting, with no rule change in mind → `/detection-harness:hunt`.
- The user is asking what existing rules cover ("do we have a rule for X?") → the `catalog`
  skill. Don't draft something new until you've checked.
- The user is triaging a single alert → answer the question. Suggest a tune only if they
  conclude the rule is wrong.
- The work is on the SIEM platform itself (parsers, ingestion, dashboards): this lifecycle covers
  rules only.
