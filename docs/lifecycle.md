# The detection lifecycle

Eight stages, each with a clear owner. The harness automates the middle of it, from research
through to the review request. Intake and operation stay with people, and so does the decision
to deploy.

| # | Stage | Owner | What happens | In the harness |
|---|---|---|---|---|
| 1 | **Intake** | Human | An idea arrives from an incident, threat intel, a hunt, a red-team finding or a coverage review. The output is a hypothesis and the reason it matters | `catalog` checks whether it's already covered |
| 2 | **Research** | Model | Confirm the telemetry exists and learn what its fields really look like | `draft-rule` must read the log-source doc first (`query-grounding`) |
| 3 | **Draft** | Model, with human input | Write the logic and metadata. The only truly creative stage | `/draft-rule` |
| 4 | **Validate** | Script, then model | Mechanical checks, each with exactly one correct answer. The model fixes what it may and reports the rest | `/validate-rule` runs *your* linter |
| 5 | **Test** | Script, then human judgment | Run the logic over history, compare volume with the severity table, read sample rows | `/test-rule` |
| 6 | **Review** | Human | A second person checks intent, severity against volume, mapping and exclusions | `/submit-rule` opens the PR/MR and stops |
| 7 | **Deploy** | Human merge, then CI | **The merge is the deploy.** Opening a PR doesn't make a rule live; merging it does | Never automated: the agent never merges |
| 8 | **Operate** | Humans, fed by data | False positives turn into tunes, volume drift triggers review, dead rules are retired deliberately | Tunes re-enter at stage 3 and go through every gate again. `catalog` covers review and retirement |

## Commands, skills, scripts

| | What it is | Example |
|---|---|---|
| **Command** | An ordered procedure a person invokes | `/test-rule` |
| **Skill** | Knowledge the agent loads when it's relevant | `query-grounding` |
| **Script** | One deterministic job, run only when called | Your linter, your query client |

**The one-correct-answer test.** If a step has exactly one correct output for a given input, it
belongs in a script: bracket balance, tactic ID format, technique-to-tactic parentage, counting
matches. If it needs judgment, it stays with the model, and a skill writes down how to think about
it: whether the logic catches the behaviour, whether an exclusion is safe, which severity fits.
"Usually right" isn't good enough for a check that runs on every rule.

This repo ships only the commands and skills. The scripts are yours, connected through
`DETECTION.md` (see [hooking-up.md](hooking-up.md)).

## Design rules the commands follow

- **Stateless.** Every command says so, and re-reads `DETECTION.md` and the rule each time. A
  second rule drafted in the same session must not inherit the first rule's severity, mapping or
  exclusions.
- **Each command names the next one** on success, so the lifecycle moves forward without anyone
  having to remember the order.
- **The router has a "when NOT to use" section.** It matters as much as the routing table: it
  keeps hunts, triage and coverage questions out of the rule lifecycle.
- **"Tune" and "too noisy" go through the same gates as a new rule.** A tune changes detection
  logic, so it gets validated, tested and reviewed like any other change.
- **Ask, do, enforce.** The skill asks for the standard, your script does the work, and
  enforcement sits outside the conversation. CI is the reliable layer. A post-write hook is the
  faster one.
- **The linter lives with the rules.** CI and the agent run the same code, from the same repo.
- **Clean-machine test.** Everything the workflow depends on has to be in the plugin or in the
  rules repo, never in one person's home directory or personal memory. Install it on a fresh
  machine and run one rule all the way to a PR.

## Zero results

A rule that has never been seen to fire is a hypothesis, not a detection. `/test-rule` treats
zero matches as suspicious: confirm the fields and values exist, then prove the rule fires on a
known-positive event (a historical true positive, a replay, or a simulation in a lab) before
calling it done.

For automating that proof with an attack simulation, a lab host and SIEM polling, see
Kongsgård's [end-to-end detection validation](https://blog.cauchy.org/blog/detection-validation/).
This harness leaves that step to your own tooling.
