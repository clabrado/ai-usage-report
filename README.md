# ai-usage-report

A [Claude Code](https://claude.ai/code) skill that calculates your local token usage, raw API cost, and efficiency rating from session JSONL files — on any platform.

## What it does

- Finds Claude session files on your machine (macOS, Linux, Windows)
- Tallies tokens across all config directories (per-fork and combined)
- Calculates raw API cost at current Sonnet pricing
- Compares actual plan spend to API value to produce an **efficiency rating**
- Self-configures on first run via plan mode — saves your plan info so you never have to re-enter it

## Install

Copy `SKILL.md` into your Claude skills directory:

```bash
# macOS/Linux
mkdir -p ~/.claude/skills/ai-usage-report
curl -o ~/.claude/skills/ai-usage-report/SKILL.md \
  https://raw.githubusercontent.com/clabrado/ai-usage-report/main/SKILL.md
```

Then restart Claude Code.

## Usage

```
/ai-usage-report
```

First run: enters plan mode to ask about your subscription(s), saves config into the skill file, then runs the full report.

Subsequent runs: skips setup and goes straight to the report.

To reconfigure:
```
/ai-usage-report --reset
```

## Output

- Token totals by directory and combined (input / output / cache-write / cache-read)
- Raw API cost per directory
- Efficiency rating: what % of API value you're capturing per dollar spent (700% = $7 of API value per $1 paid)
- Model breakdown
- One-line summary

## Pricing used

Current Claude Sonnet rates:
- Input: $3 / MTok
- Output: $15 / MTok
- Cache-write: $3.75 / MTok
- Cache-read: $0.30 / MTok

## Notes

- Reads only local `.jsonl` session files — no data leaves your machine
- Works with any number of config forks (`.claude`, `.claude-eco`, `.claude-work`, etc.)
- Plan config is stored in `SKILL.md` itself between `PLAN_CONFIG_START` / `PLAN_CONFIG_END` markers
