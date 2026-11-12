# Making your tools agent-safe

The harness calls your validator, query client and git tooling. An agent can only be as
reliable as the tools it calls, and tools written for humans tend to fail in ways an agent
can't see: prompts that hang, errors replaced by "something went wrong", or 50,000 rows dumped
into its context.

You don't need new tools. These rules turn the ones you have into tools an agent can use safely.

## The rules

1. **Use distinct exit codes.** At minimum, "the input is bad" must be distinguishable from "the
   tool broke". The harness assumes:

   | Exit | Meaning | What the agent does |
   |---|---|---|
   | `0` | OK | Continues |
   | `1` | Upstream/runtime error | Shows the raw error and stops or retries once |
   | `2` | Missing config or credentials | Sends the user to `/setup` |
   | `3` | Validation violations | Fixes or reports the violations |

   A plain exit `1` for everything collapses "your rule is wrong" into "my tool crashed", and
   the agent will edit a good rule to work around a broken tool.

2. **Print raw upstream errors.** The SIEM's parse error with its line and column is the most
   useful thing an agent can see. Pass it through word for word on stderr. Never replace it with
   "query failed".

3. **Name the missing thing and the fix.** `MISSING_CREDENTIAL: ES_API_KEY not set (run /setup)`
   beats `401`.

4. **Never prompt.** An interactive prompt hangs the agent. Watch for git asking for
   credentials, an editor, or a merge message. Take everything as arguments or environment
   variables, and fail fast when something is missing.

5. **Offer `--json`.** Give each violation or result a stable name (`[lookback_zero]`,
   `"check": "mitre_parent"`), so the agent acts on structure instead of parsing tables.

6. **Return the count separately, and never truncate it.** `{"count": 48210, "events": [...first 500...]}`.
   A count inferred from a truncated sample is wrong.

7. **Write large output to disk.** Above a threshold, write the results to a file and return a
   summary plus the path. The agent reads the file with targeted reads.

8. **Read-only by default.** The query tool's credential should only be able to search. Make that
   a property of the credential, not a line in a prompt. Keep the one tool that writes (the one
   that opens the PR) separate, with its own scoped token.

9. **Enforce limits in code.** Maximum time range, maximum rows, and a default window (such as
   24 hours) belong in the tool. A prompt can ask for small queries, but only the tool can
   guarantee them.

10. **Don't restate the tool in the prompt.** `DETECTION.md` names the command and its exit codes.
    The tool's own `--help` is the source of truth for the rest.

## Questions to ask before giving an agent a tool

- If the agent ran this by mistake at 3 a.m., would you care? If so, the credential is too broad.
- Can the agent tell from the exit code alone whether to fix the input or stop?
- When it fails, does the agent see what a human would see?
