---
name: ai-usage-report
description: >-
  Calculate local Claude token usage, raw API cost, enterprise-adjusted cost,
  and efficiency rating (value per $1 spent) from session JSONL files on any platform.
---

# /ai-usage-report

## USER PLAN CONFIGURATION
<!-- PLAN_CONFIG_START -->
NOT_CONFIGURED
<!-- PLAN_CONFIG_END -->

---

## Execution logic

### Check configuration first

Read the block between `PLAN_CONFIG_START` and `PLAN_CONFIG_END` above.

- If it says `NOT_CONFIGURED` — run **Setup Mode** below.
- If it contains saved plan data — skip to **Report Mode**.
- If the user passed `--reset` as an argument — run **Setup Mode** regardless.

---

### Setup Mode

Call `EnterPlanMode` first. Then ask the user these questions:

1. Which Claude plan(s) do you have, and which config directory does each cover?
   - Known fixed prices (do not ask the user for these): Pro = $20/mo, Max = $100/mo, Max 5x = $200/mo
   - For **Teams or Enterprise**: the monthly budget varies — ask the user for the exact dollar amount. Do not assume any default.
2. If you have a Teams or Enterprise plan: what is your exact monthly budget in USD?

After the user answers, call `ExitPlanMode` to approve the plan, then save their plan configuration by editing this SKILL.md file — replace the contents between `PLAN_CONFIG_START` and `PLAN_CONFIG_END` with a structured summary of their answers, for example:

```
Plans:
- .claude-work + .claude: $NNN/mo Teams/Enterprise budget (user-provided)
- .claude-eco: $200/mo Max 5x
```

Then immediately proceed to **Report Mode** using those answers — do not make the user invoke the skill again.

---

### Report Mode

Run all steps below in one response.

#### Step 1 — Find session files

Search for Claude session files on this device. Do not assume a fixed path:
- macOS/Linux: look for `.claude`, `.claude-*` dirs under home with a `projects/` subfolder containing `.jsonl` files
- Windows: check `%APPDATA%\Claude`, `%USERPROFILE%\.claude`, and any custom path
- Mobile/tablet: likely not accessible — say so and stop

List every directory found.

#### Step 2 — Token totals (last 30 days)

Parse `usage` fields from all assistant messages. Show per-fork and combined:

| Fork | Input | Output | Cache-W | Cache-R | Total | Msgs |
|---|---|---|---|---|---|---|

Express in M (millions) or B (billions).

#### Step 3 — Raw API cost

Pricing: Input 3 USD/MTok · Output 15 USD/MTok · Cache-write 3.75 USD/MTok · Cache-read 0.30 USD/MTok

| Fork | Input | Output | Cache-W | Cache-R | Total |
|---|---|---|---|---|---|

#### Step 4 — Efficiency rating

Using the saved plan configuration:
- For each plan: discount factor = plan monthly cost / API cost of directories it covers
- Total actual spend = sum of all plan monthly costs
- Value per dollar = total API value / total actual spend
- Efficiency rating = value per dollar as a percentage (1.59 = 159%; 100% = break-even; never show as a delta)

| Metric | Value |
|---|---|
| Total API value | $X |
| Total actual spend | $X |
| Discount vs API | X% cheaper |
| Value per dollar | $X.XX |
| **Efficiency rating** | **X%** |

#### Step 5 — Model breakdown

| Model | Responses |
|---|---|

Sorted by count descending.

#### Final summary (required)

> "You consumed [X]B tokens over 30 days, with an API value of $[X], at an actual cost of $[X] — an efficiency rating of [X]%: $[X.XX] in value per $1 spent."
