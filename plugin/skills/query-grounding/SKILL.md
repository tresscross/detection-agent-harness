---
name: query-grounding
description: Use whenever you are about to write, fix or run a SIEM query — for a detection rule, a hunt, or a test baseline. Covers loading the log-source doc first, testing short before widening, reading raw errors instead of rewriting from memory, and what to check when a query returns zero rows. Applies to any query language.
---

# Grounding queries in real telemetry

A model can't learn your parser from a prompt. Without facts about the real data, it invents
field names, guesses value formats, and then broadens the logic when the query comes back
empty. The fix is **facts first, then feedback**: load what the data actually looks like, write
the query, run it small, and let the real result correct you.

## Before you write a filter

1. **Load the log-source doc for each source you'll query.** It's listed in `DETECTION.md`
   under **Log sources**. Use the field names exactly as the doc spells and cases them. Check
   the doc's **fill rate** for each field: a field that's populated in 2% of events can't
   carry a rule.
2. **Load only the docs you need.** Loading every doc bloats your context and blurs the one
   that matters.
3. **Load the query-language notes** (`DETECTION.md` → **Query language**). Read the
   bad-forms table and platform limits before writing. Those notes record mistakes the team
   has already paid for.
4. **No doc?** Run a tiny sample query on that source (short window, a handful of events),
   read the real field names and values, and say that your field names came from samples
   rather than a doc.

## Value formats are where queries go quietly wrong

Field *names* fail loudly once the query runs. Field *values* fail silently: a country written
as `US` versus `United States`, mixed case, `true` versus `"true"`, arrays versus scalars,
domain-qualified versus bare user names. A wrong guess just returns nothing. Take value
formats from the doc's example values or from a sample event, never from memory or from the
vendor's documentation. The parser is the authority, not the vendor.

## Run short, then widen

- Start with a **30-minute window**, and widen only once the query parses and returns
  sensible rows.
- Then run the full window the task needs, such as 24 hours for a hunt or the test-rule
  baseline window.
- Respect the platform limits in the query-language notes (maximum lookback, row caps,
  expensive functions). A long search that fails at the end wastes more time than a short one
  that fails right away.

## When it fails: read the error

- **Read the raw error text** and fix exactly what it says: wrong argument count, unknown
  function, type mismatch. Don't rewrite the query from scratch from memory. That swaps a
  known error for an unknown one.
- If your tooling replaces the real error with a bare "query failed", tell the user. That's a
  tooling gap worth fixing (see `agent-safe-scripts.md` in the harness docs).
- A query that fails the same way twice goes to the user, along with both attempts.

## When it returns zero rows

Zero rows is information, not a cue to broaden the logic. Work through this list in order:

1. **Fill rate**: is each field you filter on actually populated for this event type?
2. **Value format**: does the value you compare against match the format in the doc or a sample?
3. **Scope**: are the source, index, dataset and event-type filters right, and does the window
   cover a period when the activity could have happened?
4. **Feed health**: is the source sending data at all? (A doc's event count, or a bare count
   query over the window, will tell you.)

Change the detection logic only after all four check out. Then say why the query is quiet:
"Fields and values verified; no matching activity in the window" is a real answer.

## Exclusions

Build exclusions on attributes an attacker **can't choose**: network origin (ASN), a signed
binary at an exact path, a managed-device identifier, an exact parent-child process pair.
Don't build them on attributes an attacker can choose or fake: user names, display names,
command-line substrings, file names. Give every exclusion a one-line reason. If the rule's
notes show an exclusion was rejected before, don't propose it again. Ask instead.

## Always report

- The **row count**, stated separately from the rows. Never infer it from a truncated sample.
- The window you actually queried.
- Anything the query couldn't see (missing sources, unpopulated fields).

## Feed the lessons back

When a mistake happens twice (a bad form, a wrong value format, a platform limit), offer to add
a row to the team's bad-forms table in their query-language notes, with three columns: form,
symptom, replacement. That table is how the next session avoids repeating the mistake.
