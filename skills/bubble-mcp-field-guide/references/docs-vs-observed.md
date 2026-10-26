# Where the docs and observed behaviour differ

Docs: https://manual.bubble.io/help-guides/ai/bubble-mcp. Read first on 2026-10-19, and
read again on 2026-10-25, when the page showed "Last updated 4 days ago" and some of
its statements had changed (rows below give both dates where they differ). Read once
more on 2026-10-26 for the pages-and-elements rows. Observed: one test build on
2026-10-17, beta server, and a second test, focused on pages and elements, on
2026-10-26.

An agent that trusts the docs alone will be wrong on these points. Re-check them
against the live tool list in your own session: either side may have changed.

| Topic | Bubble docs say (read 2026-10-19 and 2026-10-25) | Observed on 2026-10-17 |
|---|---|---|
| Logs | 2026-10-19: the server cannot read logs. 2026-10-25: the agent can check server logs ("Debug") | 2026-10-17: `query_logs` and `query_app_metrics` were in the tool list; `query_logs` read the logs of both the development and the live version (see `logs-and-debugging.md`). The docs now agree on logs; they do not mention live logs or `query_app_metrics` |
| Live data | 2026-10-25: the agent can read the live database and cannot change it (development: read and write) | 2026-10-17: search, lookup by id, create and bulk create against live were all refused with a message pointing to development. Treat live reads as unavailable until your own session shows otherwise (`dev-live-and-plans.md`) |
| Tool list | Both readings list `grant_agent_data_access`, `get_app_user_impersonation_token`, `create_test_cases`, `get_debug_mode_session` (2026-10-25: in the collapsed "Tool details") | `grant_agent_data_access` was not in the tool list; the server had tools the docs page does not list: `query_app_metrics`, `initialize_api_call`, `get_versions` |
| AI access to data types | Record tools work only for types an admin granted (`grant_agent_data_access`, "Requires an admin") | Record tools worked on all new types without a grant |
| Plan limits | The MCP acts within the plan's limits (still so on 2026-10-25: "Your plan applies") | Missing features were refused cleanly (savepoint, live version), but a database trigger was created on a plan without triggers, with no error and no issue; only an editor banner showed it |
| Workflow API | Enable before creating backend events | Already enabled on a new app |
| Plan flags and record tools | The plan applies to what the agent can do | On the Free plan the plan context showed `api: false`, yet the record tools (search, bulk create) worked, and `exposesWfApi` was true. Read the flags per feature, do not infer one from another |
| Savepoint | Ask for one before large changes | Refused on the Free plan ("plan does not include version control"); worked after upgrading |
| API call initialization | Calls must be initialized in the editor | `initialize_api_call` worked through the MCP |
| Seeing the app as a user | `get_app_user_impersonation_token` gives temporary sign-in as an app user | `get_preview_link` with `runAsAppUserId` was not available to sessions signed in with Bubble; the impersonation token was not tried |
| Privacy in backend workflows | Privacy rules apply to API workflows unless "Ignore privacy rules" is ticked (workflow API privacy rules manual) | Matches the docs: the checkbox defaulted to off. The server's own guide stated the opposite |
| Connecting Claude Code | `--scope user`, or a single project (2026-10-25) | Local scope in one folder also works, keeping the tools out of other projects (see `setup-and-account.md`) |
| Popups and tables | No separate tools for popups, inputs or tables; they go through the general element tools (`create_element`, `set_element_properties`) (read 2026-10-26) | 2026-10-26: matches. A popup is `create_element` with type Popup and the page as parent; a table is `create_element` with type Table, which comes with its own columns and rows as child elements |
| Mobile views | `create_mobile_view` needs the app to include a mobile app (read 2026-10-26) | 2026-10-26: refused with "the app has no mobile app", while `get_mobile_views` listed a view that came with the app template. A listed view does not mean the app can have new ones |
| Preview link and consent | `get_preview_link` is listed among the checking tools; consent and scopes are not mentioned (read 2026-10-26) | 2026-10-26: `get_preview_link`, called with a consent granted earlier (2026-10-17), dropped the whole MCP session with a "needs additional permissions" message (a run/preview scope) instead of a refusal. After the user signed in again it worked |

Rule: when they disagree, trust what the server does, verify by reading back or by the
user's eyes, and tell the user which source you followed.

On 2026-10-25 the page names `grant_agent_data_access`, `get_app_user_impersonation_token`,
`get_debug_mode_session` and `query_logs` in its collapsed "Tool details" tables (expanded and
checked that day). Check your own tool list in any case.
