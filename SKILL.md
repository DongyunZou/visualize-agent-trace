---
name: visualize-agent-trace
description: Generate best-so-far score curves for agent optimization runs found in a workspace directory. Each run's score trajectory is plotted against cost (USD), output tokens, and elapsed time, with one row per task and one color per method variant. Cost and token usage are recovered from Codex CLI and Claude Code session logs. Use when the user asks to visualize, plot, or compare agent runs, traces, or optimization curves.
---

# Visualize Agent Trace

## Input

- Workspace directory: the user provides a target directory that contains the
  run directories described under Context.
- Optionally, the user names the file and columns that record the score
  trajectory of a run. If they do not, discover it (see Context).

## Output

- Curve graph: a PDF file named `curves.pdf` written to the workspace
  directory.
  - The graph contains multiple rows, one per task.
  - Each row has three subgraphs:
    1. Cost (in USD) vs. Score
    2. Output Tokens vs. Score
    3. Time Elapsed (relative to the first session start) vs. Score
  - Different methods and method variants are plotted in the same row with
    different colors and legend entries.
  - Repeated runs (seeds) of the same variant share one curve style and are not
    differentiated by color or legend.
  - The x-axis is the respective metric (Cost, Output Tokens, Time Elapsed) and
    the y-axis is the Score. All axes are linearly scaled.
  - Curves are best-so-far: for each x value, the y value is the best score
    achieved up to that point, not the score at that specific point.
  - If the first few points are far from the rest along the y-axis, crop the
    y-range to the region that holds most optimization points instead of
    showing every point, so later readers are not misled by outliers.
  - Each subgraph has a resolution of at least 500x500 pixels.

## Context

- The workspace directory contains run directories laid out as
  `<task>/<method>/<variant>/<seed>/`. If the user describes a different
  layout, adapt to it. Treat every run directory as read-only; the only file
  written is `curves.pdf` in the workspace directory.
- Each run directory may contain an optimization record: a file such as a CSV
  or JSONL that logs one row per evaluated candidate with a timestamp and a
  score. Its name and columns vary by project, so ask the user or inspect the
  run directory. If no such record exists, reconstruct the trajectory from the
  agent session logs (for example, from tool calls that ran the evaluation and
  reported a score).
- Use Seaborn + Matplotlib to generate graphs.

### Agent session logs (per CLI)

Detect which CLI produced a run from which log layout exists inside the run
directory (variant names often hint at it as well, e.g. `codex*` vs
`claude*`). Harnesses usually redirect each CLI's home directory into the run
directory; look there first, then fall back to the CLI's default location.

- **Codex CLI**: sessions live under `$CODEX_HOME/sessions/YYYY/MM/DD/rollout-*.jsonl`
  (`CODEX_HOME` defaults to `~/.codex`). Token usage: `event_msg` lines with
  `payload.type == "token_count"`; `payload.info.total_token_usage` is
  CUMULATIVE within one session file (fields: `input_tokens`,
  `cached_input_tokens`, `output_tokens`, `reasoning_output_tokens`,
  `total_tokens`), timestamp in the top-level `timestamp`. Accumulate across
  session files ordered by start time. The model is in the `turn_context`
  line (`payload.model`). Cost is NOT recorded: obtain token prices from the
  official pricing page on the Internet for the model used (a run may involve
  multiple models; handle each model's prices).
- **Claude Code**: sessions live under `<config dir>/projects/*/*.jsonl`
  (the config dir defaults to `~/.claude` and is often redirected per run via
  `HOME` or `CLAUDE_CONFIG_DIR`). Token usage: lines whose `message.usage`
  exists are PER-MESSAGE DELTAS (fields: `input_tokens`,
  `cache_creation_input_tokens`, `cache_read_input_tokens`, `output_tokens`),
  timestamp in the top-level `timestamp`, model in `message.model`; sum deltas
  in timestamp order. If the harness captured the stdout of
  `claude -p --output-format json` invocations (one JSON object per completed
  invocation), those objects carry authoritative totals: prefer their
  `total_cost_usd` (and `modelUsage.<model>.costUSD`) over Internet pricing.
  Per-message cost can then be interpolated by prorating a session's cost
  over its cumulative output tokens.

## Steps

1. Analyze the user-provided target directory to identify all run directories
   following the layout above.
2. For each run directory, locate the optimization record and the agent
   session logs.
3. Plot the curve graph.
4. Double-check the generated graph to ensure it meets the specified
   requirements and looks visually clear and informative.
