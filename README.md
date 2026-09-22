# visualize-agent-trace

An agent skill that turns a directory of agent optimization runs into
best-so-far score curves (score vs. cost, output tokens, and elapsed time),
recovering cost and token usage from Codex CLI and Claude Code session logs.

The skill itself is [`SKILL.md`](SKILL.md).

## Install

Clone this repository into a skills directory under the name of the skill,
for example for Claude Code:

```bash
git clone https://github.com/DongyunZou/visualize-agent-trace.git \
  .claude/skills/visualize-agent-trace
```

Then invoke it with `/visualize-agent-trace <workspace directory>` or simply
ask to visualize the runs in a directory.

## Expected workspace layout

```
<workspace>/
  <task>/<method>/<variant>/<seed>/   # one run directory per seed
```

Each run directory holds the agent's session logs (redirected `CODEX_HOME`
or Claude Code config dir) and, optionally, a record of evaluated candidates
with timestamps and scores. See `SKILL.md` for the log formats that are read.

## Output

`curves.pdf` in the workspace directory: one row per task, three subgraphs
per row (cost, output tokens, elapsed time on the x-axis; best-so-far score on
the y-axis), one color per method variant, seeds sharing a style.
