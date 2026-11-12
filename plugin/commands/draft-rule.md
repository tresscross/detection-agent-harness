---
description: Draft a new detection rule, or tune an existing one, grounded in your real log-source fields and written in your repo's rule format.
argument-hint: "<what to detect>  |  tune <path/to/rule> <what's wrong>"
---

Write a rule into the team's rules repo that passes their validator on the first try and
queries fields that actually exist.

## 0. Every run starts fresh

This command is **stateless**. Even if you just drafted a sibling rule in this conversation,
gather every per-rule value again: source, severity, MITRE mapping, exclusions, filename.
Sibling rules often differ on exactly those values. Re-read `DETECTION.md` and the docs below,
too: they may have changed, and your memory of them is out of date.

## 1. Load the environment

Read `DETECTION.md` at the rules-repo root (or `$DETECTION_MD`). If it's missing, or has no
**Rules** or **Validate** section, stop and suggest `/detection-harness:setup`.

Note what's available: the **Query language** notes, the **Log sources** docs, and the
**Severity and volume** table. You'll need them below.

## 2. New rule or tune?

- **Tune**: the user named an existing rule, or said "too noisy", "false positive", "exclude…".
  Read the rule file and work on it in place. **Read its history first**: the notes field,
  `git log` for the file, and the operational record if `DETECTION.md` has a **Catalog**
  section (see the `catalog` skill). If an earlier decision covers the change you're about
  to make, quote it and ask instead of proposing it again. Ask what's wrong if it isn't clear:
  which events are noise, and why they're benign. Keep the rule's severity unless the user
  explicitly asks to change it. Skip to step 4.
- **New rule**: continue to step 3.

## 3. Gather inputs (new rules)

From the request, work out:

- **Intent**: the behaviour being caught, and why it matters.
- **Log source**: which source shows that behaviour.
- **MITRE ATT&CK** tactic and technique, if the team's format uses them.
- **Severity**: on the team's scale from `DETECTION.md`.
- **Expected volume**: a rough sense of how often it should fire. This anchors the severity.

Ask once, in a single concise question, for anything you can't work out. Don't set severity
above the middle of the team's scale without the user's confirmation.

**Check for an existing rule first.** Use the `catalog` skill: search the rules directory for
rules on the same source and technique, or with similar logic. If one is close, show it and
ask whether to tune it instead of adding a near-duplicate.

## 4. Ground the query in real fields

Follow the `query-grounding` skill. Before writing any query logic, read:

1. **The log-source doc** for this source (the **Log sources** path; match on the doc's
   `selector` or name). Use only fields the doc lists, spelled and cased exactly as it shows
   them.
2. **The query-language notes**, for syntax pitfalls and house rules: required enrichments,
   output fields, comment style.
3. **The template rule** named in **Rules**, for the shape of the file.

If there's **no doc for this source**, ask the user for one or two real sample events (or
the output of a quick query that returns one) and take field names from those. Record in the
rule's notes or description that the field names haven't been checked against a log-source
doc. If the user can't provide a sample either, say plainly that the query is unverified.

## 5. Write the logic

- **Scope to the source first**: index, dataset or logsource filter, then narrow event filters,
  and only then joins or aggregation.
- **Match the behaviour, not one sample**: aim at what the technique always has to do, not
  at incidental strings from one write-up.
- **Keep exclusions narrow and explained**: exclude on attributes an attacker can't choose
  (network origin or ASN, a signed binary at an exact path, a managed-device ID, an exact
  parent-child process pair). Never exclude on attributes an attacker can choose, such as user
  names, command-line substrings or file names. Give each exclusion a one-line reason.
- **If a query runs while you draft, keep it short**: start with a 30-minute window and
  widen only once it parses (see `query-grounding`).
- **Make the alert readable**: include the fields an analyst needs to triage without
  re-querying (who, what, where from, on what), following the team's output conventions.
- **If the rule runs on a schedule**: the lookback must be at least the schedule interval, or
  events between runs are missed. Longer lookbacks cause duplicate alerts, unless the query
  dedupes by construction (an aggregation with a threshold, for example). If you use an
  overlap, say why in the rule.

For a **tune**: change as little as possible to fix the stated problem. Record what changed and
why in the rule's notes or changelog field, if the format has one.

## 6. Write the file

Follow the template's field order and the naming convention in **Rules**. For a new rule,
**refuse to overwrite** an existing file: if the name collides, propose a more specific one.
Show the user the path you wrote.

## 7. Validate right away

Run the **Validate** command from `DETECTION.md` against the file. If it reports violations
that `DETECTION.md` says you may fix, fix them **once** and re-run. If it still fails, or
the violations are of the human-only kind, show them to the user word for word along with
a suggested fix. Don't loop.

## 8. Next step

> Drafted `<path>`. Next: `/detection-harness:test-rule` to measure alert volume against recent data.
