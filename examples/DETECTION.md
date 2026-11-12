# DETECTION.md

<!--
EXAMPLE. This is DETECTION.md for "Example Corp", a fictional team. They write Sigma
rules, convert them for Elastic, and review on GitHub. The tools named below (tools/lint.py,
tools/es_query.sh, the Makefile targets) are their tools, not part of the harness. Swap in yours.
Copy this to the root of your rules repo, or run /detection-harness:setup to draft your own.
-->

## Rules
- Rules live in `rules/<platform>/`, one rule per file, named `<platform>_<behaviour>.yml`
  in snake_case (e.g. `rules/aws/cloudtrail_logging_stopped.yml`).
- Format: Sigma YAML. Team metadata goes under a top-level `x-harness` key: `state`, `owner`,
  `severity_override` (rarely used) and `notes`. `level` is the severity.
- Template: `rules/aws/cloudtrail_logging_stopped.yml`.

## Validate
- Command: `make lint RULE=<rule>`. CI runs the same target on every PR.
- Exit `0` = pass. Exit `3` = violations, printed as `[check_name] message (hint)`.
  Any other exit code means the tool broke.
- The agent may fix these on its own: `[schema]` missing fields, `[id]` missing UUID,
  `[tags]` tactic/technique tag format, `[filename]`, `[x_harness]` missing state or owner.
- Leave these to a human: `[duplicate_id]`, `[duplicate_logic]`, `[mitre_parent]`
  (technique doesn't belong to the tactic), `[convert]` (pySigma can't convert the rule).

## Query
- Get the query from a rule: `sigma convert -t lucene -p tools/pipelines/ecs.yml <rule>`.
  It prints the Lucene query on stdout.
- Run a query: `tools/es_query.sh --index <index> --start <ISO8601> --end <ISO8601> --query-file <file> --json`.
  - Prints `{"count": N, "events": [...]}`. Above 500 events it writes them to `/tmp/es_query/<id>.ndjson`
    and prints `{"count": N, "path": "..."}` instead.
  - Exit `1` = Elasticsearch error (raw response on stderr). Exit `2` = `ES_URL` or `ES_API_KEY` not set.
- Credentials: `ES_URL` and `ES_API_KEY` env vars (read-only API key, search privilege only).
- The index for each source is in its log-source doc under `query_scope`.
- Redaction: never print `aws.cloudtrail.user_identity.access_key_id` in full; show the last 4 characters.

## Query language
- `docs/query-language.md`: Lucene and Sigma conversion notes, bad-forms table, platform limits.

## Log sources
- `docs/log-sources/`, one doc per source, generated with the `log-source-docs` skill.
- Match a rule to a doc by the rule's `logsource` block against the doc's `selector`.

## Submit
- Base branch: `main`. Branch: `detection/<slug>`.
- Commit message: `detection: <rule title>` for new rules, `tune: <rule title>` for changes.
- Open review: `gh pr create --draft --base main --head <branch> --title "<title>" --body-file <file>`.
- Reviewers: assigned automatically by `CODEOWNERS`.

## Severity and volume
These are our starting numbers. We recalibrate them every quarter against analyst capacity.

| Sigma `level` | Max alerts/day |
|---|---|
| informational | no cap |
| low | 20 |
| medium | 5 |
| high | 1 |
| critical | 0.3 (about 2/week) |

## Catalog
- Per-rule records: `make catalog RULE=<rule-id>` prints JSON with `fires_30d`, `last_fired`,
  `tp`, `btp`, `fp`, `last_reviewed`, `last_tuned`.
- Join key: the Sigma `id`.
