# Log-source doc template

Put one doc per source in the folder that `DETECTION.md` names under **Log sources**. The agent
reads the matching doc before it writes a query against that source, so field names and value
formats come from here.

**Generate it from your real data, not from vendor documentation.** Parsers rename, drop and
nest fields. The `log-source-docs` skill builds the doc from three queries over about 30 days of
data and refreshes it later, so drift shows up as a diff.

---

```markdown
---
source: <short name, e.g. aws-cloudtrail>
selector: <how a rule names this source in your rule format.
           Sigma: {product: aws, service: cloudtrail}. Otherwise an index, tag or dataset filter.>
query_scope: <the exact filter that limits a query to this source, e.g. index=cloudtrail>
generated: <YYYY-MM-DD>, from <N> days of data
---
Events in window: <count>   <!-- keep this on its own line near the top: a dead feed shows up as a drop in the diff -->

# <Source name>

## What it is
One or two sentences: what produces these events, and what they're good for detecting.

## Top event types
| Event type / action | Events | Share |
|---|---|---|

## Field reference
Fields exactly as your SIEM stores them. Fill = % of events in which the field has a value.

| Field | Fill | Example (sanitized) | Notes |
|---|---|---|---|
| | | | Value format: case, units, array vs. scalar, enum values |

If fields differ a lot between event types, add one table per major event type.

## Known empty
Fields the vendor documents but your pipeline never fills. Listing them stops the agent from
building on them.

| Field | Why it's empty (if known) |
|---|---|

## Gotchas
- Which timestamp to use (event time vs. ingest time), ingest delay, duplicate events.
- Fields that are null on failure, values that change format between event types.

## Hand-written notes
Anything that came from people rather than queries. A refresh never overwrites this section.
```
