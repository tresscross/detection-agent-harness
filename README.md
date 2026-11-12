# detection-agent-harness

A skill suite that runs the detection engineering lifecycle with a coding agent
(**draft → validate → test → submit**) on whatever SIEM, rule format and git host you
already use.

It ships **no code**: only commands, skills and a guide. Your own linter, query tool and
log-source docs do the deterministic work. The agent does the judgment and stops at a pull
request. A human merges, and the merge is the deploy.

## How it fits together

```
you ─▶ /draft-rule ─▶ /validate-rule ─▶ /test-rule ─▶ /submit-rule ─▶ PR ─▶ human merge = deploy
           │               │                 │              │
           ▼               ▼                 ▼              ▼
      log-source docs   your linter      your query tool   your git host
      query notes                        severity table
                     └──────────── all described in DETECTION.md ────────────┘
```

- **Commands** are the procedures: `setup`, `draft-rule`, `validate-rule`, `test-rule`,
  `submit-rule`, `hunt`.
- **Skills** hold the knowledge:
  - `detection-engineering` routes requests through the lifecycle
  - `query-grounding` covers writing queries against real fields
  - `log-source-docs` builds field references from real telemetry
  - `catalog` covers coverage questions, rule history, review and retirement
- **`DETECTION.md`** sits in your rules repo and tells the agent how *your* environment works.

## Quickstart

1. **Install the plugin** in Claude Code:
   ```
   /plugin marketplace add tresscross/detection-agent-harness
   /plugin install detection-harness@detection-agent-harness
   ```
2. **Open your rules repo** and run:
   ```
   /detection-harness:setup
   ```
   It looks at your repo (CI config, Makefile, existing rules, git remote), asks for whatever it
   can't work out, writes `DETECTION.md`, and checks that each piece works.
3. **Write a rule:**
   ```
   /detection-harness:draft-rule alert when someone stops CloudTrail logging
   ```
   Each command ends by naming the next one.

The only requirements are a rules directory and a command that validates one rule. Everything
else (query tool, log-source docs, PR command, severity table, catalog) is optional. When a
piece is missing, the commands say what they can't do and ask you instead.

## Docs

| | |
|---|---|
| [Hooking up your environment](docs/hooking-up.md) | What goes in `DETECTION.md`, section by section |
| [The lifecycle](docs/lifecycle.md) | The eight stages, who owns each, and the design rules |
| [Agent-safe scripts](docs/agent-safe-scripts.md) | How to make your existing tools safe for an agent to call |
| [Severity and volume](docs/severity-volume.md) | The severity/volume table and how to calibrate it |
| [Log-source template](docs/log-source-template.md) | The format for per-source field references |

## Examples

[`examples/`](examples/) is laid out as the rules repo of a fictional team that writes Sigma,
converts it for Elastic and reviews on GitHub:

- [`DETECTION.md`](examples/DETECTION.md): a filled-in environment file
- [`rules/aws/cloudtrail_logging_stopped.yml`](examples/rules/aws/cloudtrail_logging_stopped.yml): one rule, including its notes and baseline
- [`docs/log-sources/aws-cloudtrail.md`](examples/docs/log-sources/aws-cloudtrail.md): a log-source doc (illustrative numbers)
- [`docs/query-language.md`](examples/docs/query-language.md): query notes with a bad-forms table
- [`.claude/settings.json`](examples/.claude/settings.json): an optional hook that runs the linter after every rule edit

The tools that example names are the fictional team's own, not part of this repo.

## Background

This repo comes out of these posts on [tresscross.blog](https://tresscross.blog/):

- [Using coding harnesses to streamline detection engineering](https://tresscross.blog/using-coding-harnesses-to-streamline-detection-engineering/): the lifecycle, commands vs. skills vs. scripts
- [How to get your idiot LLM to write better queries](https://tresscross.blog/how-to-get-your-idiot-llm-to-write-better-queries/): grounding queries in real telemetry
- [Maintaining a detection catalog in the LLM era](https://tresscross.blog/maintaining-a-detection-catalog-in-the-llm-era/): catalog and review
- [Building cloud control-plane detections with chokepoints](https://tresscross.blog/building-cloud-control-plane-detections-with-chokepoints/): the idea behind the example rule

Prior work: Kongsgård's [end-to-end detection validation](https://blog.cauchy.org/blog/detection-validation/)
shows how to automate proving that a rule fires.

## License

[Apache-2.0](LICENSE)
