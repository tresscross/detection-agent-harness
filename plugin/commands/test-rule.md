---
description: Run a rule's query against recent data, measure alert volume, and check it against your severity/volume table before submitting.
argument-hint: "[path/to/rule] [--days N, default 7]"
---

Measure how often the rule would have fired, and say whether that volume fits its severity.
The query tool supplies the counts. The verdict and what to do about it are your judgment.

This command is **stateless**: re-read `DETECTION.md` and the rule file every time.

## 1. Load the environment

Read `DETECTION.md` (rules-repo root, or `$DETECTION_MD`). You'll use:

- **Query**: how to run a query, how to get a query out of a rule file, what the output looks like.
- **Severity and volume**: the maximum acceptable alerts per day at each severity.

If there's no **Query** section, use the **manual fallback** in step 3. If there's no
**Severity and volume** section, report the rates without a verdict, and point the user to
that section of the hook-up guide.

## 2. Pick the rule and get its query

- Use the path the user gave, or the most recently changed rule (as in `validate-rule`).
  Name the file you picked.
- Get the runnable query in the way **Query** describes: the rule's query field, or an
  extraction or conversion command for formats like Sigma.
- Window: `--days N` if given, otherwise 7 days. A longer window gives a better sample for rare
  rules, but it costs more to run on a large source. Say which window you used.

## 3. Run it

**With a Query section:** run the command as described. Pass the query in the way
`DETECTION.md` specifies (a file is safer than inline text when the query contains quotes or
pipes). If the output is written to a file, read the file rather than pasting it inline.

- If credentials or config are missing, stop and point to `/detection-harness:setup`.
- If the query errors, show the raw error. A syntax error goes back to `draft-rule`, not to
  silent edits here.

**Manual fallback (no Query section):** give the user the exact query and window to run in
their SIEM. Ask for the total count and, if possible, the top values of the main grouping
field. Continue from there.

## 4. Report the volume

```
Rule:          <name>
Window:        last <N> days
Total matches: <count>
Per day:       <count / days>
First / last:  <timestamps>

Top groups (by <field>):
  <value>   <count>
  ...
```

Use the rule's own grouping or aggregation key for "top groups" if it has one. Otherwise use the
field that best explains the volume (user, host, source IP, process).

## 5. Verdict

Compare the per-day rate with the rule's severity in the **Severity and volume** table, and
give a one-line verdict:

- **Within range** → fine to submit.
- **Over the cap** → say how far over it is, and which groups drive the volume. Recommend tuning the
  query first: carve out the specific benign drivers you can see in the top groups, and look at
  sample rows, not just counts. If most of the volume comes from one entity, a missing exclusion
  is the likely cause. Moving to a lower severity is the user's call, never yours, and it doesn't
  count as a tune.
- **Zero results** → this is **suspicious, not a pass**. A rule that never fires looks the same as
  a broken one. First work through the zero-rows checklist in `query-grounding`: field fill
  rate, value format, scope, feed health. Then ask how to confirm the rule can fire:
  - check one or two known-positive events from this source against the query, or
  - widen the window to see whether it fires at all.
  Never broaden the detection logic just to get results.
  Don't call it working until one of these is done, or the user knowingly accepts the risk.

## 6. Show some evidence

Show 3–5 sample matches so the user can see what the rule actually catches. Trim long tokens and
secrets, and follow any redaction rules in `DETECTION.md`.

## 7. Record the baseline

Commands don't share memory, so write the result where `submit-rule` can find it. If the rule
format has a notes or metadata field, offer to add one line to it:
`Baseline <YYYY-MM-DD>: <per-day>/day over <N>d (<verdict>)`.

## 8. Next step

- **Within range** →
  > Next: `/detection-harness:submit-rule` to open a review.
- **Too noisy** → go back to `/detection-harness:draft-rule tune <path> …`, then re-test.
- **Zero results, not yet confirmed** → confirm first (step 5), then submit.
