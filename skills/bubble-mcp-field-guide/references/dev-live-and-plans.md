# Development, live and plans

Read this when the user asks about live, deploying, plan limits, savepoints or
rolling changes back. Both topics are about walls that are not the agent's mistake.

## Development versus live

Sources: Bubble docs read 2026-10-25 (https://manual.bubble.io/help-guides/ai/bubble-mcp,
"Access to your app's data" and "Current limitations"); Observed 2026-10-17.

| | Through the MCP | Source |
|---|---|---|
| Development data | Read and write with the record tools | Bubble docs; Observed |
| Live data | Docs: read only, no writes. Observed: reads were refused too (details below). Do not count on live reads, and never plan a live write | Bubble docs 2026-10-25; Observed 2026-10-17 |
| Writing with the live version | Do not. The docs say the agent cannot change live data or deploy. If a call that targets live fails or the session drops, do not retry it: the user signs in again | Bubble docs 2026-10-25 |
| Deploying, copying data to live | The user, in the editor | Bubble docs 2026-10-25 |

Practical rules:

- Every write targets the development version. If the user asks to "fill live" or
  "deploy", explain that this is done in the editor, and offer a checklist for them.
- One failed live call is enough to learn the lesson; do not retry it, since a retry
  can cost the user another sign-in.
- On the Free plan there is no live version at all; read requests for live structure
  were refused for that reason (Observed).

## Plan walls

- First call in any app: `get_app_plan_context`. It returns flags such as `api`,
  `scheduler`, `dataTrigger`, limits such as `maxTotalDbThings`, and the allowed
  recurrence for scheduled workflows (Observed).
- Refusals for missing plan features were clear, with the reason (version control,
  live version) (Observed).
- Not everything is refused: the write tools accepted a database trigger the plan did
  not cover, and no check flagged it (Observed; see `backend-workflows.md`). The plan
  context is the reference; compare against it yourself.
- The Bubble docs say the MCP acts within the plan's limits. In practice that held for
  refusals, not for every write (Observed).

Do not quote prices or plan names to the user from memory; point to
https://bubble.io/pricing and the plan context.

## Rolling back

| Tool | What it does | Notes |
|---|---|---|
| `apply_changes` | All-or-nothing batch | A failed operation rolls back the whole batch (Bubble docs; Observed) |
| `undo` | Undoes MCP changes, also per conversation | Does not touch the editor's own undo history and does not bring back deleted records (Bubble docs). Option set writes skip the undo stack (Observed) |
| `create_savepoint` | A restore point | Depends on the plan: refused on Free, worked after an upgrade (Observed) |
| `restore_savepoint` | Returns the app to a savepoint | Replaces all later changes, including the user's own manual edits (Bubble docs). Not tried in the test; never run it without the user's explicit yes |

See also the version control manual:
https://manual.bubble.io/help-guides/maintaining-an-application/version-control

## Reading the plan context

- Flags can look contradictory: the Free plan context showed `api: false` and
  `exposesWfApi: true` at the same time, and the record tools worked anyway (Observed).
  Treat each flag as describing one feature and test what you need.
- `get_versions` and `create_savepoint` were refused on Free with the same short reason
  (Observed): "plan does not include version control". The first refusal is enough.
- Reading the structure of the live version on Free failed with a message that the plan
  has no live version (Observed).

## Live data: the docs and the test differ

- **Bubble docs, read 2026-10-25:** the agent can read the live database and cannot
  change it; development is read and write. The page was updated a few days before.
- **Observed 2026-10-17:** search, lookup by id, create and bulk create against the live
  version were all refused with a clean message that points to the development
  version, and the session continued. The refusal did not depend on whether live had
  any data, and a database copied to live later did not change it.

Both are dated, and the server is in beta, so check what your own session does. What
holds either way: "fill live with the same test data" is never a task for the agent;
offer the user the editor's own way to copy data.

If live reads do work in your session, the agent can see real people's records, and
privacy rules do not apply to it (`SKILL.md` section 4). Read only what the task needs,
and do not copy it into development or into your answer.

Before you offer a data copy, ask where the "test" records came from. If they are fake
(`user-a@example.com`), say that live may be better off starting empty or with data the
owner seeds on purpose. If any of them are copies of real people, say that copying them
on spreads personal data into another database, and let the user decide.

## Other rollback tools in the docs

`list_pending_undo` and `acknowledge_undo_recovery` manage undo entries; records the
agent deleted cannot be brought back by undo (Bubble docs, not exercised in the test).
Do not delete records "to clean up" without the user's yes.

## Plugins

Paid plugins must be bought by the user before an agent can install them (Bubble docs,
not exercised in the test). Do not plan work that depends on one.
