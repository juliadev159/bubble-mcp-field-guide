# bubble-mcp-field-guide

A skill that helps Claude Code build and check Bubble apps through Bubble's MCP.

By Julia Tarasenko, Bubble developer since 2021 and Bubble Ambassador (cohort 4).
Community project, not affiliated with or endorsed by Bubble.

## What it is, and who it is for

Bubble's MCP server lets an AI agent create and change things in a Bubble app: data
types, privacy rules, pages, API calls, backend workflows. It is a good builder. It
is not a reviewer: its checks report what is malformed, not what is unsafe or what
silently does nothing.

This skill gives Claude Code the habits of someone who has been bitten by that:

- **Safe defaults for six silent traps**, none of which anything flagged in the test:
  new data types open to everyone, multi-hop privacy conditions that quietly drop
  records from searches, API workflows public by default, the "Ignore privacy rules"
  default, plan limits that were not checked on write in the test (the docs say the
  plan applies), and API re-initialization that silently drops fields.
- **The line between development and live**, so the agent stays in development and
  does not promise what the plan or the MCP cannot do.
- **A post-build checklist and hand-off report**: what the agent checked, and what a
  human still has to verify, step by step.

It is for people who let Claude Code work in a Bubble app: freelancers, agencies and
founders. You do not need to be a Bubble expert to use it, but the point of the skill
is that every build ends with an honest list of what you still need to look at.

## Install

Copy the skill folder into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/juliadev159/bubble-mcp-field-guide.git
cp -r bubble-mcp-field-guide/skills/bubble-mcp-field-guide ~/.claude/skills/
```

To use it in one project only, copy it to `.claude/skills/` inside that project
instead (create the folder if it does not exist).

Then connect the Bubble MCP server as described in the
[Bubble manual](https://manual.bubble.io/help-guides/ai/bubble-mcp); the skill's
`references/setup-and-account.md` covers the connection and the account check. The
skill loads on its own when you ask Claude Code to work on a Bubble app.

## What's inside

```
skills/bubble-mcp-field-guide/
├── SKILL.md                      the core: before the first write, build loop, six traps,
│                                 who the agent is, secrets, development vs live
├── references/
│   ├── setup-and-account.md      connecting, signing in, two accounts, choosing scope
│   ├── data-and-privacy.md       data types, option sets, test data, privacy rules
│   ├── elements-and-design.md    pages, elements, forms, popups, tables, styles
│   ├── api-connector.md          API Connector collections and calls
│   ├── backend-workflows.md      backend and API workflows, triggers
│   ├── dev-live-and-plans.md     development vs live, plan limits, savepoints, undo
│   ├── logs-and-debugging.md     reading server logs
│   ├── docs-vs-observed.md       where the Bubble docs and the observed behaviour differ
│   └── after-build-checklist.md  auditing a build, or finishing one
├── assets/handoff-report.md      the report template the agent ends with
└── evals/                        offline scenarios and trigger checks
```

Every rule in the skill is marked as **Observed** (two hands-on test sessions:
2026-10-17, a whole small app; 2026-10-26, pages and elements) or
**Bubble docs** (with a link where there is one). The skill does not copy the Bubble
manual or the server's own guides; it points to them and adds what they leave out.

## Limits

- **Beta.** The Bubble MCP was in beta when the skill was written, and it changes.
  Before relying on anything here, look at the tool list your server actually exposes.
  When the server, the docs and the skill disagree, trust the server and say which
  source you followed.
- **Two test sessions behind it.** The field notes come from two test sessions with
  Claude Code, on 2026-10-17 and 2026-10-26. Other plans, apps and later server
  versions may behave differently. Mobile views were not covered: the test app had no
  mobile app.
- **Development only.** Use it on the development version of a test app. Do not point
  it at a client's app without the client's say-so, and do not write to the live
  version.
- **Not a review.** The skill makes the agent safer and more honest about gaps. It does
  not replace a person opening the editor, using "Run as" on two different users and
  checking what each one sees.
- **Tested offline.** It ships with offline evals (`skills/bubble-mcp-field-guide/evals/`):
  seven scenarios run with and without the skill, and a set of messages that should and
  should not load it. They check what the agent knows and decides, not live tool calls,
  and they do not replace a test against the live server.

## Disclaimer

Bubble® is a trademark of Bubble Group, Inc. This is an independent community
project, not affiliated with, endorsed by, or sponsored by Bubble. Not official
documentation.

## License

[MIT](LICENSE)

## Who made this

I'm Julia Tarasenko. I've built on Bubble since 2021, mostly the layer users don't see: data model, privacy rules, backend workflows, API integrations. This skill grew out of my own tests of the Bubble MCP.

## Need help with a Bubble app?

I take freelance projects and subcontract work for agencies, mostly the backend of Bubble apps that handle money or sensitive data.

Write to me: [projects.julia@tarasen.co](mailto:projects.julia@tarasen.co?subject=Bubble%20MCP%20skill). Email works best: I work async, in writing.
Case studies: [julia.tarasen.co](https://julia.tarasen.co) · [LinkedIn](https://www.linkedin.com/in/julia-tarasenko)
