# After-build checklist

Read this at the end of any build, and as the main script when asked to audit or
review an app an agent built. Each item says what the agent can check through the MCP
and what only a human can check (or the agent through a browser tool, with the
user's permission).

Zero issues in `check_app_issues` is necessary, not sufficient: every trap in
SKILL.md section 3 passed that check in the test.

| # | Check | Agent, through the MCP | Human, in the editor |
|---|---|---|---|
| 1 | Every new type has privacy rules, and "everyone else" has no access | `get_node` on each type | — |
| 2 | No multi-hop conditions in privacy rules | Read each condition as human-readable text; count the hops | "Run as" two users: own records visible, others' hidden, nothing missing |
| 3 | The owner field is set in every workflow that creates a private record | Read the create actions of each workflow | Create a record as a test user and check it is visible to that user |
| 4 | No record has an empty owner | Search for records with an empty owner (the agent ignores privacy, so it sees them) | — |
| 5 | API workflows are not public unless needed | `get_node` on each API workflow | Confirm the checkbox and authentication in the editor |
| 6 | Every "Ignore privacy rules" choice has a reason | Read the event properties | Run the workflow and check the result |
| 7 | Each workflow does what it should | Cannot run workflows; can read data back after a run | Trigger it (test button or editor) as the right user |
| 8 | Every feature is covered by the plan | Compare triggers, scheduled and recurring workflows, record counts with `get_app_plan_context` | Look for plan banners on backend events |
| 9 | API calls still return every field the app uses | List used fields; `check_app_issues` after any re-initialization | — |
| 10 | `check_app_issues` is zero | Yes | Issue counter in the editor |
| 11 | Visual properties outside the catalog look right: padding (`get_node` does not show it); button text contrast against its background (a theme style made for dark backgrounds, with white text and no fill, made a button invisible on a white page); a button disabled by a condition returns to full opacity once enabled; height of reusable element instances (the minimum height is copied at creation and does not follow the definition) | Cannot see them: `get_node` and `check_app_issues` (0 issues) passed with an invisible button and a stuck-transparent button. With the user's permission, a browser tool on the preview link can read them from the page (text colour, background, opacity, sizes) (Observed 2026-10-26) | Look in the editor and in preview, with the condition both on and off |
| 12 | No real personal data in the development database | — | Owner decides |
| 13 | Probes, test buttons and test records are removed or listed | List them in the hand-off report | Delete or keep on purpose |
| 14 | The user knows changes reach live only after they deploy | Say it | Deploy when ready |
| 15 | Inputs fill the width of their container, and at about 375 px wide there is no horizontal scroll | After each create, check whether the palette set a fixed width or minimum height (the palette gives inputs a fixed width even when stretch was requested). With the user's permission, a browser tool on the preview link at 375 px can compare the page's scroll width to the viewport (Observed 2026-10-26) | Look in preview and in the editor's responsive view at a narrow width |
| 16 | Dynamic text on the editor canvas is readable (tables and repeating rows especially) | Check that dynamic texts have `editor_preview_text`; the MCP warns about it in the response to the write that sets the dynamic text | Look at the editor canvas: without the preview text it shows the raw expression, while the page itself shows values (Observed 2026-10-26) |

## Running an audit

1. Read-only first. Do not fix anything during the audit unless the user asks; list
   findings with the item number above.
2. For each finding, say what it means for the app's users in one plain sentence
   (for example, "any visitor can search all invoices").
3. Propose a fix per finding, and separate what the agent can do from what the user
   must do in the editor.
4. Finish with the hand-off report (`../assets/handoff-report.md`).

## How to do the human checks

Write these into the hand-off report as steps the user can follow in the editor.

**Run as.** In the editor, open the data tab, App data, All Users, choose a test user,
then "Run as", and open the page that lists their records. The agent cannot do this
through the MCP. With the user's permission, a browser tool can click through the same
steps (Observed: this is how the privacy probe below was read).

**Probe a privacy rule.** To learn whether a rule works in searches, not only whether it
saves:

1. Create a record that only the rule under test would let a particular user see (for
   example a record with an empty owner, reachable only through a multi-hop rule).
2. Add a temporary list on a page that searches the type with no filter, so that what
   appears is exactly what the rules let through.
3. Run as two users. The expected result for a correct rule: each sees own records and
   not the others'.
4. Read the result in both directions. The probe record missing for its intended viewer
   means the rule silently fails closed (nothing leaks, data "disappears"). Another
   user's record present means it fails open and leaks.
5. Remove the probe record and the probe list, or list them in the hand-off report.

In the test the multi-hop rule failed closed: no leak, and no warning anywhere (Observed).

**Order of work in an audit.** Findings that expose data to everyone (rows 1, 5 in the
table above) come before findings that hide data (row 2), and both before cosmetic
ones (rows 11, 15, 16), since a leak is harder to take back than a missing list.

## What a failed check looks like

| Symptom the user reports | Likely cause | Row |
|---|---|---|
| "My list is empty after the agent added privacy rules" | A multi-hop condition, or a record created without an owner | 2, 3, 4 |
| "A text on the page shows an error, but nothing else changed" | An API call was re-initialized and a response field disappeared | 9 |
| "The backend workflow ran and changed nothing" | A search blocked by privacy rules while "Ignore privacy rules" is off | 6 |
| "The trigger never fires" | The plan does not include database triggers | 8 |
| "Everything is green but my client can open the endpoint" | The API workflow is still exposed publicly | 5 |
| "A button is on the page in the editor but users cannot see it" | A style made for dark backgrounds (white text, no fill) on a light page; choose styles by their properties, not by name (Observed 2026-10-26) | 11 |
| "The button works but still looks disabled" | Opacity from the style's "isn't clickable" condition did not reset after the element's own condition stopped disabling it; add an explicit condition that sets opacity back to 100 when the button is enabled (Observed 2026-10-26) | 11 |
| "In the editor a table shows expressions like 'Current row's … name' instead of values" | Dynamic text without `editor_preview_text`; only the editor canvas is affected, the page itself shows values (Observed 2026-10-26) | 16 |
