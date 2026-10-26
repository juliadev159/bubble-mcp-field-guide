---
name: bubble-mcp-field-guide
description: Use this skill for ANY work on a Bubble.io app through Bubble's MCP server (mcp.bubble.io) from Claude Code, including small or quick requests: adding a field or data type, a page or button, privacy rules, API Connector calls, backend or API workflows, test data, connecting or troubleshooting the MCP, and reviewing, checking or auditing what an agent built in a Bubble app (even a quick "is it ok?" look at a page or at privacy rules). Use it even when the user only says "in my Bubble app" and never says "MCP". The MCP does not flag unsafe defaults (new data types and API workflows are public, multi-hop privacy conditions silently fail, re-initializing an API call drops fields), and a review needs the post-build checklist. Not for Bubble plugin code, Bubble's built-in editor AI, Data API scripts, or other platforms. Unofficial; not affiliated with Bubble.
license: MIT
---

# Building on Bubble through its MCP server: field guide

An independent, unofficial skill. Bubble® is a trademark of Bubble Group, Inc.; this
project is not affiliated with or endorsed by Bubble.

The guidance below comes from two places, and every rule says which:

- **Observed 2026-10-17**: one hands-on build of a small test app with Claude Code,
  while the Bubble MCP was in beta.
- **Observed 2026-10-26**: a second session on the same kind of app, focused on pages
  and elements (forms, popups, reusable elements, custom states, conditionals, tables).
- **Bubble docs**: the public manual, mainly
  https://manual.bubble.io/help-guides/ai/bubble-mcp (read 2026-10-19 and again
  2026-10-25; the page was updated in between).

The server changes faster than this file. Before you rely on anything here, look at
the tool list the server actually exposes in this session. When the server, the
docs and this skill disagree, trust the server's behaviour, then confirm with your
own eyes (a read-back call, or the user looking in the editor). The known
disagreements are listed in `references/docs-vs-observed.md`.

The one idea to carry through every task: **the MCP is a good builder but not a
reviewer.** Its checks report what is malformed, not what is unsafe or what silently
does nothing. Your job is to build with safe defaults, verify after every step, and
end with an honest list of what you could not verify.

## 0. Before the first write

Do these in order. They are cheap, and skipping them is how agents end up editing the
wrong app or promising features the plan does not have.

1. **Confirm the account.** `whoami` returns only ids, not an email (Observed), so you
   cannot tell which account you are in. Say that to the user in plain words. The
   email is visible on the OAuth consent page in the browser during sign-in, so ask the
   user to check it there. A Google sign-in can land in a different, older Bubble
   account than the user expects (Observed).
2. **Confirm the app.** Name the app in the request when the account has more than one
   (Bubble docs). `list_apps` also includes apps shared with the account
   (Observed). If you see apps the user did not mention, or the list looks unfamiliar,
   stop and ask. Work only in the app the user named; never open a client's app
   without the user saying so.
3. **Read the plan.** Call `get_app_plan_context` and note the walls: flags such as
   `api`, `scheduler`, `dataTrigger`, record limits like `maxTotalDbThings`. The MCP
   does not enforce these when you write (see trap 5), so you have to.
4. **Stay in development.** All work targets the development version. Do not pass a
   live app version to any write tool (section 6).
5. **Savepoint before large changes**, if the plan allows it. Bubble docs recommend
   it; on the Free plan `create_savepoint` was refused with a clear "plan does not
   include version control" message, and it worked after upgrading (Observed).

Setup, sign-in and multi-account details: `references/setup-and-account.md`.

## 1. Use the server's own guides, but keep the user's scope

The server asks you to read its own guides at certain points (for example before
`create_app`, and a privacy guide) (Observed). Read them: they describe the privacy
traps well. Two cautions from the test:

- The build guide pushes toward a full production app. Build what the user asked for,
  not more.
- One statement about backend workflows and privacy was inaccurate (see trap 4).

This skill does not copy those guides; it adds what they leave out.

## 2. The build loop

1. **Scout.** `get_app_outline`, `get_node` on what you will touch. Know the current
   state before planning.
2. **Plan** the change in plain words and share it with the user if it is non-trivial.
   Include privacy rules in the same plan as the data types, not as a later step.
3. **Validate expressions first** with `validate_expression`.
4. **Write in small batches** with `apply_changes`, using temp ids to link a new event
   to its actions in one batch. Batches of around ten operations worked well
   (Observed). `apply_changes` is all-or-nothing: on a failed operation the response
   reports the failing index and that the batch was rolled back (Bubble docs;
   Observed).
5. **Check after every batch** with `check_app_issues`, and always after re-initializing
   an API call (trap 6).
6. **Read errors, do not guess.** Server errors are specific: a wrong property name
   comes back with the list of valid ones; a bad privacy condition comes back with the
   reason (Observed). Fix from the message.
7. **Report briefly** what changed, then continue.

Catalog calls such as `list_action_types` return very large responses, and the
backend event catalog did not fit in a single tool response (Observed). Request them
once, narrowly, and only when you need them.

## 3. Six silent traps (safe defaults)

In the test, nothing flagged any of these at the moment it happened: no error, and no
issue from the call itself (some surface later, see 6). Apply the safe default without
being asked, and say that you did.

1. **New data types are open to everyone.** A freshly created type lets "everyone"
   view and search all fields (Observed, via `get_node`). Write privacy rules in the
   same step as the type, with "everyone else" getting no access. On 2026-10-26 the
   first `create_privacy_rule` on a new type also switched "everyone" to no access in
   the same call (Observed); the type before that first rule was not re-checked, so
   still treat it as open.
2. **Privacy conditions must be one hop.** `This Order's owner is Current User` works.
   A multi-hop condition such as `This Installment's order's owner is Current User` is
   saved without any error, `check_app_issues` reports nothing, the editor shows it as
   normal, and records that depend on it silently disappear from searches (Observed;
   Bubble docs say multi-level references cannot grant access in searches). It fails
   closed: nothing leaks, but the owner's data "vanishes". Fix by denormalizing: put an
   `owner` field directly on the type and set it in every workflow that creates one.
3. **API workflows are created public.** "Expose as a public API workflow" defaults to
   on (Observed). Set `expose: false` unless the user needs an external endpoint, and
   choose authentication deliberately.
4. **"Ignore privacy rules" in backend workflows defaults to off.** Privacy rules apply
   to API workflows unless that checkbox is ticked (Bubble docs, workflow API privacy
   rules). The server guide said otherwise; the editor agreed with the docs (Observed).
   With strict rules, a backend search may return nothing unless you tick it. Decide
   per workflow, and tell the user why.
5. **Plan limits were not checked on write in the test.** The docs say the plan
   applies ("Your plan applies", read 2026-10-25). A database trigger was created on a plan
   with `dataTrigger: false`: no error, zero issues in `check_app_issues` and in the
   editor's issue counter; only a banner in the editor showed that the event is inactive
   on this plan (Observed). Compare every feature against `get_app_plan_context`
   yourself.
6. **Re-initializing an API call silently drops fields.** If the new test response
   lacks a field, that field leaves the response schema; the re-initialization itself
   returns no issues (Observed). Make the test call return a superset of the fields
   you use, then run `check_app_issues`. Details: `references/api-connector.md`.

## 4. Who the agent is

The agent acts as the signed-in Bubble user, an editor of the app (Bubble docs).
Privacy rules do not apply to it: in development it saw and searched every record,
including fields hidden from other users, and records it created were attributed to
an internal admin user (Observed).

Two consequences to state to the user:

- A clean result from the agent's own search says nothing about what an app user sees.
- Do not keep a copy of real users' personal data in the development database if an
  agent has data access.

## 5. Secrets

The MCP does not accept real keys or passwords: use the placeholder `REPLACE_ME` and
let the user type real values in the editor. Saved API responses come back as
`[REDACTED]` (Observed). OAuth2 setup for the API Connector belongs in the editor
(Bubble docs). Never ask the user to paste a key into the chat, and never print one.

## 6. Development versus live

- **Data:** the docs (read 2026-10-25) say the agent reads and writes the development
  database and can only read the live one. In the test (Observed 2026-10-17) reads
  against live were refused too, so do not count on them; check in your session. If they
  work, what you see is real people's data: read only what the task needs.
- **Do not write with the live version.** The docs say the agent cannot change live
  data or deploy. If a call that targets live fails or the session drops, do not retry
  it: the user signs in again.
- **Deploying and moving data to live** are done by the user in the editor (Bubble
  docs). When asked to "copy the test data to live", also ask whether those records are
  fake or real people's data before offering the editor steps.

Plans, savepoints, undo and rollback: `references/dev-live-and-plans.md`.

## 7. What the agent cannot verify

Say these out loud at the end of every build; they are where "built" and "works" part.

- **Running a workflow.** There is no "run" tool (Observed). The agent can build a
  backend workflow but cannot prove it changes data. Verification: a test button on a
  page that schedules the workflow, clicked by the user (or in the editor), then a
  read-back of the data.
- **Seeing the app as a user.** `get_preview_link` with `runAsAppUserId` was not
  available to sessions signed in with Bubble (Observed). Test users created through
  the MCP have no passwords, so nobody can log in as them in preview. Verification: the
  user opens the editor's App data, uses "Run as" on two different users, and checks
  that each sees their own records and nothing else. A browser tool, with the user's
  permission, can do the same clicks. `get_app_user_impersonation_token` is listed in
  the docs but was not tried.
- **Seeing the page at all.** Read-back and `check_app_issues` do not show how a page
  looks. On 2026-10-26 `get_preview_link` (without `runAsAppUserId`) worked after the
  user signed in again, and a browser tool on that link, with the user's permission,
  found what zero issues had hidden: an invisible button and a button that stayed
  faded after it was enabled (Observed). Visual checks:
  `references/after-build-checklist.md`.
- **Plan banners.** Inactive-on-this-plan features are visible only in the editor.

## 8. Hand-off report

End every build or audit with the template in `assets/handoff-report.md`: what was
built, what the agent checked (with results), what a human must check (step by step:
where in the editor, as which user, what to expect), what was left in the app (probes,
test records), and what was not done and why.

## 9. Where to look next

| Task | Read |
|---|---|
| Connecting, signing in, two accounts, choosing scope | `references/setup-and-account.md` |
| Data types, fields, option sets, test data, privacy rules | `references/data-and-privacy.md` |
| Pages, elements, forms, popups, tables, conditionals, styles | `references/elements-and-design.md` |
| API Connector collections and calls | `references/api-connector.md` |
| Backend and API workflows, triggers | `references/backend-workflows.md` |
| Live, plans, savepoints, undo | `references/dev-live-and-plans.md` |
| Server logs, what a workflow or a user actually did | `references/logs-and-debugging.md` |
| Where the docs and observed behaviour differ | `references/docs-vs-observed.md` |
| Auditing a build, or finishing one | `references/after-build-checklist.md` |

Further reading in the Bubble manual (links only):

- Bubble MCP: https://manual.bubble.io/help-guides/ai/bubble-mcp
- Privacy rules: https://manual.bubble.io/help-guides/data/the-database/protecting-data-with-privacy-rules
- Workflow API privacy rules: https://manual.bubble.io/help-guides/integrations/api/the-bubble-api/the-workflow-api/workflow-api-privacy-rules
- API Connector security: https://manual.bubble.io/help-guides/security/api-security/api-connector-security
- Server logs: https://manual.bubble.io/help-guides/maintaining-an-application/testing-an-application/using-server-logs
- Version control: https://manual.bubble.io/help-guides/maintaining-an-application/version-control
- Plans: https://bubble.io/pricing
