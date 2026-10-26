# Data and privacy

Read this when creating data types, fields, option sets or test records, and whenever
you write or review privacy rules.

## Option sets

- Create the set, then its options. Options created in parallel kept their order
  (Observed).
- Option set writes skip the undo stack: each one returned a warning that undo is
  unavailable (Observed). Plan option sets before you start; you cannot roll them back
  with `undo`.
- In records, an option is passed by its value, e.g. `"status": "paid"` (Observed).

## Types and fields

- One call per type, with its fields in the same call, worked cleanly (Observed).
- Bubble generates the field keys itself, such as `total_number` or
  `status_option_<setname>` (Observed). Take them from the response; do not guess.

### Two names for one field

The schema tools use field **keys** (`total_number`); the record tools use the Data API
**names** (`total`) (Observed). Mixing them up is the most common small error. Before
writing records, map each key to its name once and keep the map in front of you.

## Test records

- `bulk_create_things` creates records in batches; links between records are passed as
  record ids; dates as ISO strings (Observed).
- Count against the plan: the Free plan context showed a record cap
  (`maxTotalDbThings`) (Observed). Keep test data small.
- Test users created this way have no passwords, so nobody can log in as them in
  preview; they are usable only through "Run as" in the editor (Observed).
- Use obviously fake data (`user-a@example.com`). Never seed development with copies of
  real people's data: the agent sees all of it (SKILL.md, section 4).

## Privacy rules

### The default is open

A new type grants "everyone" view and search on all fields; you can see it with
`get_node` on the type (Observed). Write rules in the same step as the type.

### The owner pattern

What worked in the test (Observed; the shape matches the "everyone else" examples in
the privacy rules manual):

1. Add an `owner` (User) field directly on each private type, even when ownership
   could be derived through another type.
2. One rule per type with a one-hop condition: `This <Type>'s owner is Current User`
   grants view and search.
3. "Everyone else": no view, no search, no fields.

The price: every workflow that creates such a record must set `owner`. A record with an
empty owner is visible to nobody but the backend. Say this to the user and check it in
the audit.

### Multi-hop conditions fail silently

The privacy rules manual says multi-level references cannot grant access in searches.
Through the MCP, such a rule is saved without error, `check_app_issues` reports zero,
and the editor shows it as an ordinary rule. In the test, a record visible only through
a two-hop rule simply did not appear in the user's search results, while other users'
records stayed hidden (Observed). So it fails closed: no leak, but missing data that
the owner cannot explain. Never write one; if you find one in an audit, flag it and
propose the owner field.

### What the server does validate

The server rejects some invalid conditions with clear messages: a search or `:count`
inside a condition, and a type mismatch in a comparison (Observed). The condition also
comes back as human-readable text; read that text back to confirm the rule says what
you meant.

### Testing rules

Privacy rules do not apply to the agent (SKILL.md, section 4), so the agent's own
`search_things` cannot test them. Ask the user to use "Run as" in the editor with at
least two users: each should see their own records, not the others', and nothing that
should be there should be missing.

### Admin access

A real app usually also needs a rule for an admin role (for example a yes/no field on
User), which the test app did not have. See the admin field example in the privacy
rules manual:
https://manual.bubble.io/help-guides/data/the-database/protecting-data-with-privacy-rules

## AI access to data types

The Bubble docs say record tools work only for data types an admin has granted to the
AI tool, via `grant_agent_data_access`. In the test that tool was not in the server's
tool list, and the record tools worked on all new types without any grant (Observed).
If record tools refuse a type, check the tool list and the editor settings, and ask
the user.

## More that was seen on privacy rules

- **Exact shape of the default.** A new type granted "everyone" view all, search and all
  fields as constrainable (Observed, `get_node`). The first fix is to take all of these
  away from "everyone else", not just one of them.
- **A type tied directly to the user.** For a type with a User field of its own, the
  one-hop rule is `This <Type>'s user is Current User`; other types get an `owner` field
  (Observed).
- **Hidden fields.** Hiding a single field "from others" was done by hiding the whole
  record from them (Observed). If only one field should be hidden while the record
  stays visible, write that as its own rule and check it with Run as.
- **The agent sees hidden fields.** A search with a filter on a field hidden from users
  worked for the agent (Observed): never use a filtered search as evidence that the
  field is hidden.
- **Who the agent can act as.** `get_app_data_context` lists app users the agent can
  act as (Observed; Bubble docs). That helps choose test users for Run as; it is not a
  way to test rules from the agent's side (SKILL.md, section 7).
- **The Data API flag.** On the Free plan the record tools worked although the plan
  context said `api: false` (Observed). Do not conclude "no data access" from that flag.

## Test data size

The Free plan context showed a limit of 200 records in the test app (Observed). Three
users, three customers, five orders and fifteen installments were enough for owner-rule
tests. Stay far below the limit so a later `bulk_create_things` does not fail halfway
through.
