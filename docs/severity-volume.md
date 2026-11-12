# Severity and volume

Severity debates ("this feels like a high") are hard to settle. Volume can be measured. The
severity ↔ volume table turns the debate into a check: a rule of a given severity may fire at
most this often, and `/test-rule` measures whether it does.

## Example table

**These numbers are examples. The right ones depend on your team.**

| Severity | Max alerts/day | If higher |
|---|---|---|
| Informational | No cap | Always fine |
| Low | 20 | Warn |
| Medium | 5 | Warn |
| High | 1 | Warn |
| Critical | ~0.3 (about 2/week) | Warn |

Put your own table in `DETECTION.md` under **Severity and volume**, using your own severity scale.

## Calibrating it

Start from capacity, not instinct:

1. **How many alerts can the team properly triage per day**, at each severity? A critical
   should get immediate, deep attention, so you can afford very few. A low might be batch-reviewed.
2. **Divide that capacity across the rules** at each level. If the team can absorb 10 medium
   alerts a day and has 40 medium rules, most of them need to be quiet most days.
3. **Check against reality** after a quarter: which rules break their cap, and do analysts agree
   with the severity of what fired? Adjust the table, not just the rules.

## What the verdict means

- **Within range**: the severity is consistent with the measured volume. Submit.
- **Over the cap**: tune first. Look at the top groups and the sample rows, and carve out the
  benign drivers with narrow exclusions. Lowering the severity is a human decision, and it isn't a
  tune: relabelling a noisy rule doesn't make it any less noisy.
- **Zero results**: suspicious, not a pass. Either the behaviour is rare or the query is broken,
  and volume alone can't tell which. Check the fields and values, then prove the rule fires on a
  known-positive event.

The measured rate is also the rule's **baseline**: `/test-rule` records it in the rule, so later
drift is visible during operation.

## Noisy but valuable

A low-severity rule that is noisy but carries real signal often works better rolled up into
risk-based scoring or a periodic summary than deleted or forced under the cap.
