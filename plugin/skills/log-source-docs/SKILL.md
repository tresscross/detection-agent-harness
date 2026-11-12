---
name: log-source-docs
description: Use when a log source has no field reference doc, when a doc looks stale, or when the user asks to "document this log source", "profile this source", "what fields does X have", or "refresh the log-source docs". Builds a per-source doc from real telemetry (event types, field fill rates, example values, known-empty fields) so queries are grounded in what the parser actually produces.
---

# Log-source docs from real telemetry

A log-source doc is the most effective way to improve the queries an agent writes. It says
which fields actually exist, how often each one is populated, and what its values look like.
Build it from **your own data**, because vendor documentation describes the product, not your
parser.

The format is the template in the harness repo's `docs/log-source-template.md`: front matter
(`source`, `selector`, `query_scope`, `generated`), an event count line, then sections for top
event types, field reference with fill rates, known-empty fields, gotchas, and hand-written
notes. Store docs in the folder named under **Log sources** in `DETECTION.md`.

## Building a doc

You need the **Query** section of `DETECTION.md`. Without it, give the user the three queries
below to run, and build the doc from the output they paste back.

1. **Pin down the source scope.** Find the exact filter that isolates this source (index,
   dataset, tag), and how the team's rule format refers to it (the `selector`). Confirm both
   with the user if they aren't obvious from existing rules.
2. **Pick the window.** The default is 30 days. Use a shorter window for very high-volume
   sources, but keep it long enough to catch weekly patterns. Record the window you used.
3. **Run three queries** (follow the `query-grounding` skill: start with a short window):
   1. **Event types by volume**: count events grouped by the event-type or action field.
   2. **Fields per event type with fill rate**: for the top event types, the share of events
      in which each field is populated.
   3. **One example value per field.** Take each example from a real event, then sanitize it
      (see below).
   If the query language can't compute fill rate across every field at once, sample: take
   a few thousand events per major event type and count field presence yourself.
4. **Write the doc** following the template.
   - Put the total event count on the line under the front matter.
   - In each field's notes, record its value format (case, enum values, array vs. scalar, units).
   - Fields the vendor documents but that are populated 0% of the time go under **Known empty**.
   - Fields filled in under ~5% of events: keep them, but say how rare they are.
5. **Show the user** the doc and point out anything surprising: an important field that's
   rarely filled, an event type they'd expect but don't see, a value format that differs from
   the vendor docs.

## Sanitizing example values

These docs are committed and shared, so example values must not leak real data:

- Replace real user names, emails, host names, internal domains, account IDs and IP addresses
  with clearly fake values in the same **format**: `jdoe@example.com`, `host-01`, `203.0.113.10`
  (documentation IP ranges), `123456789012`.
- Keep the format exact, because that's the point of the example. A UPN stays a UPN; an ARN
  keeps its full structure.
- Never include tokens, keys, session IDs or passwords, even sanitized.
- If you can't tell whether a value is sensitive, sanitize it.

## Refreshing a doc

Docs go stale as parsers change. To refresh one:

1. Re-run the same three queries over the same window length.
2. Rewrite every generated section. **Keep the hand-written notes and gotchas as they are.**
3. Show the user the diff, and call out:
   - a big drop in the event count (the feed may be broken),
   - fields that appeared, disappeared, or changed fill rate a lot (a parser change),
   - changed value formats.
4. Suggest checking any rules that use a field which disappeared or dropped in fill rate.
   The `catalog` skill can list them.

Teams that want drift to surface on its own can run these queries on a schedule in their own
CI and commit the result. The harness doesn't ship that job. Suggest it, and they build it
with their own tools.

## When NOT to use this skill

- The user wants to know what happened in the logs. That's `/detection-harness:hunt`.
- A doc exists and is recent. Read it; don't regenerate it.
