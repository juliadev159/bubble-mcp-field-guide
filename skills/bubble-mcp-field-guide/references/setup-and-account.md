# Setup and account

Read this when connecting Claude Code to the Bubble MCP, when sign-in lands in the
wrong account, or when the user works with more than one Bubble account.

## Connecting: choose the scope

The server is `https://mcp.bubble.io/mcp`, over HTTP (Bubble docs). Two ways to add it
in Claude Code, and when to pick each:

| Option | Command | When |
|---|---|---|
| User scope (as in the Bubble docs) | `claude mcp add --transport http --scope user bubble https://mcp.bubble.io/mcp` | The user builds in Bubble in most projects and wants the tools everywhere. |
| Local scope (one folder) | from inside the project folder: `claude mcp add --transport http bubble https://mcp.bubble.io/mcp` | The user wants the Bubble tools only in one project, so that other projects never see an app-editing server. This is how the test was run (Observed). |

The docs also describe adding the same server to a project's `.mcp.json`, which
shares the configuration with everyone who opens that project. Never put tokens or
credentials into a file that gets committed.

After adding the server, run `/mcp` in Claude Code and authenticate in the browser.
`claude mcp list` in a folder shows whether the server is connected there (Observed).

## Signing in: check the email on the consent page

- Sign-in is OAuth in the browser. The consent page shows the email of the Bubble
  account being connected, and each permission is a separate checkbox (Observed).
  Ask the user to read the email before allowing.
- A "continue with Google" sign-in can open a different Bubble account than the user
  intends, for example an old one tied to the same Google address (Observed).
- To switch accounts: `/mcp`, clear authentication for the server, authenticate
  again (Observed).

## After sign-in: who am I, which apps

- `whoami` returns a user id and a client id, no email (Observed). Rough heuristic: a
  Bubble user id starts with a millisecond Unix timestamp of when the account was
  created, so an unexpectedly old date is a hint you are in the wrong account
  (Observed; a heuristic, not documented).
- `list_apps` includes apps shared with the account, not only owned ones (Observed).
  A brand-new account may already contain an empty app that Bubble created at
  signup (Observed). Do not open or change any app the user did not name.
- Client apps: only with the client's permission. Per the Bubble docs, a collaborator
  gets the same access through the MCP as in the editor, and no more. Not verified in
  the test.

## Two Bubble accounts on one machine

- Add the second account as a server with a **different name** (for example
  `bubble-second`), in its own folder (Observed).
- The OAuth token appears to be stored per server name: adding a server with the same
  name in another folder showed it as already connected, and running
  `claude mcp remove bubble` in a temporary folder signed out the main one too
  (Observed). So never reuse the name `bubble` for a temporary server, and never remove
  `bubble` from a folder you do not own.

## Creating an app

- `check_app_name` first, then `create_app` (Bubble docs; Observed).
- New apps are created on the Free plan (Bubble docs). Run `get_app_plan_context`
  right after.

## Cost

MCP usage does not draw on Bubble AI credits; it is billed by the model provider
(Bubble docs; https://bubble.io/blog/agent-2/).

## Good habits for the first session

- **Name the app** in the request when the account has more than one, so the agent edits
  the right one (Bubble docs).
- **Start on a test app** the first time (Bubble docs). A new app on the Free plan costs
  nothing and has little to lose.
- **Permissions are separate choices** on the consent page and can be removed later in
  the account settings (Observed). Suggest the user grant only what the task needs.
- **A sign-in can drop mid-session.** In the test the session ended and needed a new
  sign-in several times (after a call that targeted live, after removing a server with
  the same name). After a re-sign-in, `whoami` and `list_apps`
  again, since the account can change (Observed).
