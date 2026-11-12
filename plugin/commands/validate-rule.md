---
description: Run your repo's own validator against one rule; explain the violations and fix the ones you're allowed to fix.
argument-hint: "[path/to/rule] — defaults to the most recently changed rule"
---

Run the team's validator on one rule and turn its output into a short list of fixes. The
validator decides pass or fail. Your job is to explain the result, not to second-guess it.

This command is **stateless**: re-read `DETECTION.md` and the rule file every time.

## 1. Load the environment

Read `DETECTION.md` (rules-repo root, or `$DETECTION_MD`). You need **Rules** (where rules
live) and **Validate** (the command, how to read its output, and which violations you may
fix). If either section is missing, stop and suggest `/detection-harness:setup`.

## 2. Pick the rule

- If the user gave a path, use it.
- Otherwise use the most recently changed file in the rules directory: check
  `git status --porcelain -- <rules dir>` first, then modification time. Name the file you
  picked. If it's ambiguous, ask.

## 3. Run the validator

Run the **Validate** command exactly as `DETECTION.md` gives it, with the rule path filled in.
Read the result as **Validate** describes. If it doesn't say, assume:

- exit `0`: pass
- exit `3`, or violations in the output: fail, and the rule needs changes
- anything else: **the validator itself failed** (missing dependency, bad config, crash).
  Show the raw error and stop. Don't edit the rule to work around a broken tool.

## 4. Report

**Pass**: one line naming the rule, plus any warnings the validator printed.

**Fail**: group violations by check or category so the big picture comes first. For each one,
give the check name, the validator's message word for word, and a concrete fix.

## 5. Fix what you're allowed to fix

`DETECTION.md` lists which violations you may fix on your own (typically format, missing
metadata, naming). For those, offer to fix them in place, keeping the detection logic
unchanged. Then re-run the validator **once** and report the final state.

Never fix these yourself. Explain them and let the user decide:

- anything `DETECTION.md` marks as human-only
- detection logic, thresholds or exclusions
- MITRE mappings that don't fit (the user should confirm what's actually covered)
- duplicate or uniqueness conflicts with another rule

**Never make a rule pass by weakening it.** Don't add skip or exemption flags, delete required
fields, or comment out logic.

## 6. Next step

> Validated clean. Next: `/detection-harness:test-rule` to measure alert volume.
