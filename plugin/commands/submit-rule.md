---
description: Ship a validated rule for human review — branch, commit, push, and open a pull/merge request. Never merges.
argument-hint: "[path/to/rule]"
---

Put one rule change in front of a human reviewer. **The agent never merges.** A human merging
the change is the deploy, and the moment the rule goes live.

This command is **stateless**: re-read `DETECTION.md` and the rule file every time.

## 1. Load the environment

Read `DETECTION.md` (rules-repo root, or `$DETECTION_MD`). You'll use **Rules**,
**Validate** and **Submit** (base branch, branch naming, the command that opens a review,
reviewers). Without a **Submit** section you can still validate, branch and commit. Step 6
explains what to do then.

## 2. Pick the rule

Use the path the user gave, or the most recently changed rule (as in `validate-rule`). Name the
file you picked.

## 3. Gate: it must validate

Run the **Validate** command. If it doesn't pass, **stop** and suggest
`/detection-harness:validate-rule`. Don't submit a rule that fails validation, even if the user
plans to fix it later.

Check the baseline: look in the rule's notes or metadata for a `Baseline …` line from
`test-rule`. If there isn't one, ask whether to run `/detection-harness:test-rule` first or to
submit without one. If they submit without, say so in the review description.

## 4. Gate: the tree contains only this change

Run `git status --porcelain`. The only pending changes should be this rule and any files that
belong with it, such as a test fixture or an exclusion list it references. If anything else is
modified or staged, stop and ask the user to resolve it. Don't stash, discard or commit
unrelated work on their behalf.

## 5. Branch, commit, push

1. Update the base branch from **Submit** (default: the repo's default branch) with a
   fast-forward pull.
2. Create a branch using the naming convention in **Submit**, or `detection/<rule-slug>` if
   none is given. Take the slug from the rule's name, not its filename.
3. Stage **only** the rule and the files that belong with it, by path. Commit with a message
   like `detection: <rule name>` or `tune: <rule name>`, or follow the convention in
   `DETECTION.md`.
4. Push the branch. If the push fails (auth, protected branch), show the raw error and stop.

Never push to the base branch, force-push, or rewrite history that's already been pushed.

## 6. Open the review

**With a Submit section:** run its review command. Pass the description as a file if the
command supports it (for example `gh pr create --body-file`). Open it as a draft if the team
prefers that. Capture the URL it prints.

**Without one:** stop after the push and tell the user the branch name, the base branch, and
that they need to open the review in their git host.

Description template:

```markdown
## Summary
<one paragraph: what the rule detects and why it matters; for a tune, what was wrong and what changed>

## Rule
- Name: <name>
- File: <path>
- Severity: <severity>
- MITRE: <tactic> / <technique>
- Log source: <source>

## Volume baseline
<the Baseline line from the rule, or "Not baselined: submitted without /test-rule">

## Review checklist
- [ ] Logic catches the stated behaviour, not just one sample
- [ ] Exclusions are narrow and each has a reason
- [ ] Severity matches measured volume
- [ ] Field names match the log-source doc
- [ ] MITRE mapping is accurate
```

Add reviewers from **Submit** if the git host doesn't assign them automatically.

## 7. Hand off

```
Submitted: <URL>
Branch:    <branch>

A reviewer merges it, or asks for changes. Merging is the deploy.
```

**Don't** approve, merge, or turn on auto-merge, even if the user asks you to in passing.
Merging is the human deploy step, so tell them to do it in their git host. If the reviewer asks for
changes: edit the rule on the same branch, run `/detection-harness:validate-rule` (and
`test-rule` if the logic changed), then commit and push to the same branch.
