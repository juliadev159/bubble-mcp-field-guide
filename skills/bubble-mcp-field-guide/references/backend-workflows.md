# Backend and API workflows

Read this when creating backend workflows, API workflows, recurring events or database
triggers, and when someone asks whether a workflow "works".

## Workflow API

The Bubble docs say to enable the Workflow API with `set_workflow_api_enabled` before
creating backend events. On a new app it was already enabled: the call returned the
old value as true (Observed). Calling it is harmless; read the returned old value.

## Public by default

A new API workflow had "Expose as a public API workflow" turned on (Observed). Unless
the user needs an external endpoint, set `expose: false`, choose authentication on
purpose, and tell the user. After `expose: false`, the authentication field did not
show in `get_node` output (Observed), so confirm in the editor if it matters.

## Privacy inside backend workflows

- Privacy rules apply to API workflows by default; the "Ignore privacy rules when
  running the workflow" checkbox removes them (Bubble docs:
  https://manual.bubble.io/help-guides/integrations/api/the-bubble-api/the-workflow-api/workflow-api-privacy-rules).
- The checkbox defaulted to off (Observed). The server guide stated the opposite.
- With strict owner-only rules, a backend search that should touch everyone's records
  may return nothing unless the checkbox is on. In the test it was turned on for a
  maintenance workflow, and a real run confirmed it changed the right records
  (Observed).
- Decide per workflow: ignoring privacy is right for system jobs and wrong for
  workflows that act on behalf of one user. Write down why.

## Building

- Run expressions through `validate_expression` first.
- Create the event and its actions in one `apply_changes` batch, linked by a temp id
  (Observed).
- A wrong property name returns the list of valid property names (Observed); use it.
- `check_node_issues` did not accept the backend root id `api` that other tools use
  ("not found") (Observed). Use `check_app_issues` instead.
- Backend workflows created through the MCP landed in the uncategorized folder; no
  tool parameter for folders was found (Observed). Tell the user if they organize
  workflows in folders.

## Database triggers and plans

A database trigger was created on a plan whose context said `dataTrigger: false`: no
error from the write, zero issues from `check_app_issues` and the editor counter. Only
a banner in the editor said the event type needs a plan that supports triggers, and the
event stayed inactive (Observed). Before creating any trigger, recurring event or
scheduled action, check `get_app_plan_context`, and say so if the plan does not cover
it.

## The MCP cannot run a workflow

There is no tool to execute a workflow (Observed). "The agent built the backend logic"
is not "the agent verified it". To verify:

1. Add a test button on a page whose click schedules the API workflow now (works
   through the MCP).
2. The user clicks it as a test user (editor, "Run as"), or runs it from the editor.
3. The agent reads the affected records back and compares them with the expectation.
4. Remove the test button afterwards, or list it in the hand-off report.

The run can also be traced in the server logs (`logs-and-debugging.md`): how many
records it changed, and whether a repeat run changes none.

## What can be created

On the Free plan the backend event types offered were: API workflow, recurring event,
database trigger ("a thing is modified") and custom event (Observed). Being offered is
not the same as being active: see "Database triggers and plans" above.

## Shape of a typical maintenance workflow

The test workflow marked overdue payments in one action: "make changes to a list of
things", with a search (status is pending and due date earlier than now) and a new status
(Observed). The path that worked: validate the expression, then create the event and the
action in one `apply_changes` batch.

- A non-exposed workflow was given admin-only authentication (Observed). The
  authentication field then disappeared from `get_node` output, so confirm it in the
  editor.
- A workflow that searches on a state ("only pending") can be run twice safely; the
  second run should change nothing. Use that to verify it (`logs-and-debugging.md`).
- Dates compared with "now" depend on when the workflow runs; say which time zone the
  user will see.
