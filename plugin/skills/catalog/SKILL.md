---
name: catalog
description: Use for questions about the existing detection catalog — "do we have a rule for X?", "what covers this technique / log source?", "which rules use field Y?", "what's overdue for review?", "which rules never fire?", "should we retire this?" — and before drafting any new rule, to avoid duplicates. Also covers reading a rule's history before proposing a tune. Reads the rules repo, plus the team's operational catalog when DETECTION.md has a Catalog section.
---

# The detection catalog

The catalog is what the team knows about each rule: what the rule is, whether it's deployed,
how it performs, and what people have decided about it. Agents have no memory between
sessions, so the catalog is also the only defence against an agent confidently re-proposing an
exclusion that someone already rejected.

## Where the facts live

Each kind of fact has one authoritative source:

| Fact | Source |
|---|---|
| What the rule **is** (logic, severity, mapping, notes) | The rules repo (`DETECTION.md` → **Rules**) |
| What's **deployed** and what **fired** | The SIEM |
| How alerts were **resolved** (true positive, benign true positive, false positive) | The team's case management |

If `DETECTION.md` has a **Catalog** section, it says where the per-rule operational record
lives (fire counts, dispositions, last reviewed, last tuned) and how to read it. Without that
section, answer from the rules repo alone, and say that performance data isn't available.

## Coverage questions

"Do we have a rule for X?" means **search before you answer, and search before you draft**.

1. Search the rules directory by **log source** (selector or source field), **MITRE technique**,
   and **behaviour keywords** in the names, descriptions and query logic. Search for the
   technique's parent and sub-techniques too.
2. Report what you find: each match with its path, name, severity and state, plus one line on
   what it actually detects. Matching on a keyword isn't coverage, so read the query logic.
3. Say what's **partially** covered: the same behaviour on a different source, a narrower
   variant, or a rule that's disabled or retired.
4. For a real gap, suggest `/detection-harness:draft-rule`. For a near miss, suggest tuning the
   existing rule instead.

"Which rules use field Y / source Z?" means searching the query logic. Use it after a log-source
doc refresh shows a field disappeared, to find the rules that will go silent.

## Before proposing any tune

**Read the rule's notes and history first**: the notes field, the git log for the file, and
the operational record if there is one. If a previous decision covers what you're about to
propose (for example, "decided against excluding X because…"), don't propose it again. Turn it
into a question for the human, quoting that decision.

Every decision about a rule goes in its notes with a reason, including "reviewed, no change".
A decision that isn't written down gets proposed again by the next session.

## Lifecycle states

**draft → in review → deployed → tuning → retired.** Merging into the base branch is the move
to deployed. A tuning merge marks the rule as tuned. A rule leaves the catalog through a
reviewed change, never by quietly being deleted or disabled.

## Reading the operational record

If there's a **Catalog** section, these fields drive decisions: lifecycle state, deployed date,
last fired, fires in the last 30 days, TP / benign-TP / FP counts, severity, last reviewed,
last tuned, notes.

Traps to avoid:

- **A blank cell isn't a zero.** Blank means unknown: a job didn't run, or the rule was renamed
  and the record was orphaned. Say which you suspect.
- **"Never fired" needs a manual check.** Before calling a rule dead, run its query yourself
  (`/detection-harness:test-rule`). The record may be wrong.
- **Low-severity rules often produce no cases.** Judge them on volume and last-fired, not on
  missing dispositions.
- **Renames break joins.** If fire or disposition data doesn't line up with a rule, check
  whether its name or ID changed.

## Review questions

When asked what needs attention, sort into:

- **Overdue for review.** Count from the rule's **last review**, not from when it was deployed.
  Counting from deployment floods the queue with old rules that were reviewed recently.
  New rules get a first review about 30 days after deployment.
- **Over the volume budget** for their severity (`DETECTION.md` → **Severity and volume**).
- **Rising false-positive ratio**, or **fires collapsing onto one entity**. One noisy user or
  host usually means a missing exclusion.
- **Quiet**: medium severity or above, zero dispositions, but healthy fire counts. Add a note and
  a next review date. That isn't a tuning job.

Decisions that tend to hold up (offer them as defaults; they're not rules):

- A noisy low-severity rule is better rolled up into risk-based scoring or a summary alert
  than deleted.
- A tune has to cut volume or fix the logic. Changing only the severity label isn't a tune.

## Retiring a rule

1. Confirm the reason is real. If the telemetry is gone, check that the feed is truly gone and
   not just quiet (event count, the log-source doc).
2. Retire it through the normal lifecycle: a change that removes or disables the rule, with the
   reason, through `/detection-harness:submit-rule`. If the team's process uses two steps
   (mark for deletion, then remove), follow it.
3. Record the decision and the reason in the operational record, if there is one.

## When NOT to use this skill

- Writing or changing a rule → the `detection-engineering` router and the lifecycle commands.
- Questions about events, not rules → `/detection-harness:hunt`.
