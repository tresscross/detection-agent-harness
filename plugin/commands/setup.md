---
description: Hook detection-harness up to your environment — inspect your rules repo, draft DETECTION.md, and check that each piece works. Re-run any time to re-check.
argument-hint: "[path/to/rules-repo] — defaults to the current repo"
---

Write `DETECTION.md` at the root of the team's rules repo, then check that every section in it
works. This is the only place the harness learns about the team's tools. The other commands
read the file and nothing else, so be accurate rather than fast.

The format is described in `docs/hooking-up.md` in the harness repo. The headings are repeated
here so this command doesn't depend on that file being present. Use them exactly:
**Rules**, **Validate**, **Query**, **Query language**, **Log sources**, **Submit**,
**Severity and volume**, **Catalog**.

## 1. Find the repo, pick the mode

Use the path the user gave, or the git root of the current directory. If you're not in a git
repo, ask where the rules repo is.

- **`DETECTION.md` already exists** → **check mode**: go straight to step 5, then offer to fill
  in any sections that are missing or broken.
- **No `DETECTION.md`** → **create mode**: continue to step 2.

## 2. Inspect before asking

Work out as much as you can from the repo itself. Note where each answer came from, because
the user will confirm them.

- **Rules**: look for directories full of rule-shaped files. Identify the format from the content,
  not the extension. For example, a `logsource:` plus `detection:` block means Sigma; other formats
  have their own markers (TOML rule files, saved-search config, KQL or SPL in YAML). Pick one
  representative, well-formed rule as the template.
- **Validate**: **CI config is the best source.** Whatever CI runs against rule files is the
  validator the agent should run too, so both execute the same code. Check `.github/workflows/`,
  `.gitlab-ci.yml` and other CI files, then `Makefile` targets, then `scripts/` or `tools/`. Read
  the validator's usage or help to learn how to aim it at a single file, and what its exit codes
  mean.
- **Query**: look for a query or search script in the repo, a CLI the team already uses, or a
  documented API call. Note how it authenticates, by **variable names only** (never print or copy
  their values).
- **Query language / Log sources**: look for existing docs (`docs/`, a wiki export, per-source
  markdown). Existing docs count even if they don't match the template yet.
- **Submit**: read `git remote get-url origin` to identify the git host, and check whether its CLI
  is installed (`gh`, `glab`, …). Check the default branch, how existing branches are named
  (`git branch -r`), any `CODEOWNERS` file, and the commit-message style in `git log`.
- **Severity**: look for the scale in the rule schema or in existing rules.
- **Catalog**: look for anywhere per-rule operational data already lives, such as fire counts,
  case dispositions, or last-reviewed dates (a spreadsheet export, a database, a SOAR record
  store, a script that pulls them). It's optional, and many teams won't have it yet.

Run only read-only commands while inspecting. Don't install anything, don't log in to anything,
and don't run the query tool yet.

## 3. Ask for what the repo can't tell you

Ask in **one** batch, showing what you inferred so the user can confirm or correct it:

- Anything from step 2 you couldn't find, or weren't sure of.
- **Which validator violations the agent may fix on its own** (typically format and metadata)
  and which need a human (logic, mappings, duplicates).
- **Severity and volume**: the maximum alerts per day they'll accept at each severity. If they
  don't know, offer example values and label them clearly as starting points to calibrate later:
  low 20/day, medium 5/day, high 1/day, critical about 2/week.
- Where to keep log-source docs, if there are none yet.

Sections the user can't fill in yet stay out of the file. Every command copes with missing
optional sections. Only **Rules** and **Validate** are required.

## 4. Write `DETECTION.md`

- Use the headings exactly as above. Under each one, use short bullets with exact, copy-pasteable
  commands, and show where the rule path or query goes, e.g. `make lint RULE=<rule>`.
- **No secrets.** Refer to credentials by environment-variable or secret-manager name only. This
  file gets committed, and every agent session reads it.
- Write facts, not aspirations. If the validator can't target a single file, say so (for example,
  "validates the whole directory; filter the output for the rule's path").
- If a `DETECTION.md` exists and the user asked to change it, show a diff and get a yes before you
  overwrite it.

Show the user the finished file.

## 5. Check the hook-up

Test each section that's present, read-only, and report the results:

| Section | Check |
|---|---|
| Rules | The directory exists and the template rule exists |
| Validate | Run it on the template rule. It should **pass**. If it fails, either the template is bad or the command is wrong; find out which |
| Query | Run the smallest possible query: scoped to one source, the last hour, a few results. Expect events back. On a credentials error, tell the user which variable to set and let them set it. Never ask them to paste a secret into the chat |
| Query language | The path exists |
| Log sources | The path exists. List which sources used by existing rules **have no doc**. That list is the top quality gap |
| Submit | The git host CLI is installed, and its auth status command (e.g. `gh auth status`) succeeds. Don't open a test PR |
| Severity and volume | Every level in the rules' scale has a cap, or is explicitly uncapped |
| Catalog | The read command or path works and returns records for a few known rules. Check that rule IDs or names line up with the rules repo |

Report the results as a short table showing which sections work, which are missing, and which are broken (with the raw error).

## 6. Wrap up

- `DETECTION.md` is team configuration, so it should go in through the team's normal review. Offer
  to commit it on a branch; don't push it straight to the base branch.
- If some log sources lack docs, suggest the `log-source-docs` skill for the sources that rules
  use most.
- Then:
  > Hooked up. Next: `/detection-harness:draft-rule <what to detect>`.
