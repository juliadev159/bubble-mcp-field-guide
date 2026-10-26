# Logs and debugging

Read this when something ran (or should have run) and you need to know what actually
happened: a workflow that changed nothing, an unexpected change, an external request, a
suspicious cost.

## What the server can do

- **Bubble docs, read 2026-10-25:** the agent can check the app's server logs to see
  what happened when a workflow ran and to find the cause of errors
  (https://manual.bubble.io/help-guides/ai/bubble-mcp, "Debug"; `query_logs` is in the
  collapsed "Tool details" tables). The reading of
  2026-10-19 said the opposite; the page was updated since.
- **Observed 2026-10-17:** `query_logs` and `query_app_metrics` were in the tool list and
  worked, and `query_logs` read the logs of the development version and of the live
  version. The page text does not say whether live logs are included, and does not name
  `query_app_metrics`.

The server's own tool list in your session is still the reference: if `query_logs` is
there, use it; if not, logs are read in the editor (Server logs manual:
https://manual.bubble.io/help-guides/maintaining-an-application/testing-an-application/using-server-logs).
Say which source you used.

Reading live logs does not mean touching live data. Logs are a read-only view of what
happened; they do not change anything, and writing to live stays off limits
(`dev-live-and-plans.md`).

## Writing queries

- Queries are written in OPAL, the query language of the log backend (Observed). The
  tool's own description tells you the syntax; write queries from it.
- Most common first error: selecting columns without the `timestamp` column is
  rejected, with a message that names the missing column (Observed). Read the message,
  fix, retry once.
- Ask a narrow question per query (one workflow, one record id, one time window). Log
  output is long, and the entries carry personal data (see below).

## Questions the agent can answer from logs

All four worked in the test (Observed). The shapes below are reminders, not copied
syntax.

1. **Which external requests did the app make?** Filter entries tagged as HTTP
   requests. API Connector initializations showed up here, including the target URL.
2. **Who spent how much workload?** Group workflow entries by the current user and sum
   the workload units. Every action carried its own workload cost, so the agent can
   also compare workflows by cost.
3. **Any errors or warnings?** Filter by log level. An empty result is a result: say
   "no errors or warnings in this window", not "everything works".
4. **What happened to record X?** Search for the record id and walk the entries in time
   order. In the test this showed: which click scheduled which workflow, which search
   constraints it used, how many records it changed and which ids.

`query_app_metrics` returned workflow run counts per 30-minute window, and the metrics
were shared across versions rather than split into development and live (Observed).

## Using logs to verify a workflow

The MCP cannot run a workflow (`backend-workflows.md`), but once the user has run it
(for example with a test button), logs show what it did:

- **Count changed.** A run reported how many records it changed, with their ids. Compare
  with what you expected.
- **Run it twice.** In the test the second run changed 0 records, which is what an
  "only pending ones" workflow should do. A non-zero count on a repeat run means either
  the workflow is not idempotent or something outside it changed the data.
- **A record reappeared.** A third run in the test changed two records again, because
  someone had put them back by hand between the runs. The logs showed the workflow
  runs but not the manual edit.

## Limits

- Times are in UTC (Observed). Convert for the user, and say which time zone you used.
- Edits made by hand in the editor's data tab did not appear in the logs (Observed). If
  a value changed between two workflow runs and no workflow did it, the logs cannot say
  who did.
- Logs do not replace a read-back: they say what the workflow did, not what the data
  looks like now.

## Privacy of logs

- Log entries contained the current user's email (Observed): an agent with log access
  sees app users' personal data, even when the record tools show only fake test users.
- A deploy left a service-call entry in the live logs (Observed): not every entry is
  something your app did.
- Never paste raw log output into public places, issues or posts. Summarize instead, and
  leave out emails, ids and header values.
