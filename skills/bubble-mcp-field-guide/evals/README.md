# Evals

Offline checks for this skill. They test what the agent knows and decides, not live
tool calls: the agent under test has no Bubble tools and describes what it would do and
what it would tell the user.

- `evals.json`: seven scenarios in the skill-creator format, each with checkable
  assertions. Each is run twice, with the skill and without, and the two answers are
  graded against the same assertions.
- `files/`: synthetic fixtures (an invented app read-back and invented tool output).
  They contain no real accounts, emails or ids.
- `trigger-evals.json`: 16 user messages, 10 where the skill should load and 6 where it
  should not (near misses such as a Go terminal UI library, Bubble's built-in editor
  agent, plugin development, Supabase policies, the Data API from a script, CSV export).

What these checks do not cover: the behaviour of the real server, which changes while it
is in beta. Run the scenarios again when the tool list changes, and add fixtures from
real (anonymized) server responses when you have them.
