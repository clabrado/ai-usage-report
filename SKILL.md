---
name: ai-usage-report
description: >-
  Calculate local Claude token usage, raw API cost, enterprise-adjusted cost,
  and efficiency rating (value per dollar spent) from session JSONL files on any platform.
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

Call `EnterPlanMode` first. Then ask BOTH questions in a SINGLE `AskUserQuestion` call (so the user answers everything at once):

**Question 1:** "Which personal Claude plan do you have?"
Options (max 4):
- Max 20x (200 USD/mo)
- Max 5x (100 USD/mo)
- Pro (20 USD/mo)
- No personal plan

**Question 2:** "Do you have a Teams or Enterprise plan for work — and if so, what is the monthly budget?"
Options (max 4):
- Yes — 100 USD/mo
- Yes — 200 USD/mo
- Yes — 500 USD/mo
- No Teams/Enterprise plan

After both answers come back, call `ExitPlanMode`, then save config and run Report Mode immediately.

Save the config between `PLAN_CONFIG_START` and `PLAN_CONFIG_END`. Use "USD" not dollar signs. Example:

```
Plans:
- .claude-eco: Max 20x (200 USD/mo)
- .claude + .claude-work: Teams/Enterprise (100 USD/mo)
```

If the user selected "No personal plan" or "No Teams/Enterprise plan", omit that line.

Then immediately proceed to **Report Mode** — do not make the user invoke the skill again.

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
| Total API value | X USD |
| Total actual spend | X USD |
| Discount vs API | X% cheaper |
| Value per dollar | X.XX USD |
| **Efficiency rating** | **X%** |

#### Step 5 — Model breakdown

| Model | Responses |
|---|---|

Sorted by count descending.

#### Final summary (required)

End with this exact line, filled in with real numbers:

> "You consumed XB tokens over 30 days, with an API value of X USD, at an actual cost of X USD/mo — an efficiency rating of X%: X.XX USD in value per 1 USD spent."
