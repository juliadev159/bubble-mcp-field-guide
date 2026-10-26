# API Connector

Read this when creating or changing API Connector collections and calls, or when
something broke after a call was re-initialized.

## Order of operations

1. Create the collection (authentication type, shared headers).
2. Create the call. Creating it sends nothing (Observed).
3. `initialize_api_call` sends one real request and stores the response schema
   (Observed). For non-GET calls the tool requires an explicit confirmation flag; ask
   the user before sending, because a POST may create real data on the other side
   (Observed).

The Bubble docs say calls must be initialized in the editor. In the test,
`initialize_api_call` was in the tool list and worked (Observed); check your server's
tool list.

## Secrets

- Use `REPLACE_ME` for keys and passwords; the user enters real values in the editor.
  The MCP does not accept them (Observed).
- Stored responses come back as `[REDACTED]` (Observed).
- OAuth2 setup stays in the editor (Bubble docs).
- Never ask for a key in the chat. See also API Connector security:
  https://manual.bubble.io/help-guides/security/api-security/api-connector-security

## Re-initialization drops fields silently

What happened (Observed):

1. A call returned a field (a currency rate) that a text element on a page used.
2. A parameter changed, so the new test response returned a different field.
3. After re-initialization, the old field was gone from the schema. The
   re-initialization itself returned no issues.
4. A separate `check_app_issues` caught the broken text element, but its message
   ("dynamic data … must be printable as text") does not mention the missing field.
5. Changing the request so that the response contained both fields brought the old
   field back under the same key, and the issue disappeared.

Rules that follow:

- Before re-initializing, list which response fields the app uses.
- Make the test request return a superset of those fields (all optional fields
  filled, all variants requested).
- After re-initializing, always run `check_app_issues`, and compare the schema with
  your list.
- Symptom to cause: "must be printable as text" right after a re-initialization
  usually means a response field disappeared.

## Small things

- `update_api_call` with the full parameter list issued new parameter ids (Observed).
  Do not cache parameter ids across updates.
- Initialization stored the response time (Observed).

## Symptom to cause

| What you see | Likely cause |
|---|---|
| "Dynamic data … must be printable as text" right after a re-initialization | A response field the page used disappeared from the schema |
| The tool warns that the call was initialized with old settings | A parameter changed after the last initialization; re-initialize, then run `check_app_issues` |
| A field is back after you changed the request | Field keys are stable: the old key returned once the response contained that field again (Observed) |

## A minimal first call

The test used a public GET endpoint that needs no key: create the collection with no
authentication, create the call (set to be used as data), initialize it. Three tool
calls, and the schema (the fields of the response) was stored (Observed). Start with
such an endpoint to learn how your server behaves before you wire in a service that
needs credentials or a POST.
