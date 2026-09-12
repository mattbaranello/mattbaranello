# Options Call Screening Agent

A five-filter framework for evaluating long call setups: implied volatility, catalysts,
technicals, chain liquidity, and risk/reward.

Available two ways. **The skill is the recommended one** — it costs nothing beyond a
Claude subscription and it can pull live market data.

## Option A — Claude Code skill (free, recommended)

Runs on your Claude Pro/Max subscription. No API credits, no Python, no `pip`.

```bash
# from anywhere on your Mac
cd ~/mattbaranello
claude
```

Then just ask:

```
/screen-calls AAPL
```

or in plain language — the skill triggers on its own:

```
screen NVDA for calls 30 days out, I have a $25k portfolio
should I buy calls on SOFI?
run me a weekly scan on semis
```

### Use it from any directory

The skill currently lives in this repo, so it only loads when you run `claude` from here.
To make it available everywhere:

```bash
mkdir -p ~/.claude/skills
cp -r ~/mattbaranello/.claude/skills/screen-calls ~/.claude/skills/
```

Now `/screen-calls` works from any folder.

### Data

The skill fetches live figures via web search before analyzing — IV Rank and Greeks from
Barchart, price and earnings dates from Yahoo Finance, technicals from Finviz. Anything it
can't retrieve gets marked `unavailable` rather than guessed.

Scraped options chains are less reliable than your broker's. For a setup you're actually
going to trade, paste the chain from your broker and ask it to run filters 4 and 5 against
those numbers.

## Option B — Python CLI (needs API credits)

`options_agent.py` runs the same framework through the Anthropic API, which bills
per-token separately from your Claude subscription.

```bash
python3 -m pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...
python3 options_agent.py
```

Note this version has **no market data access** — it can't look anything up, so it will
either decline to fill in the numbers or produce figures that aren't real. Use Option A
unless you're planning to wire in a data source.

## A caveat worth repeating

This is an analysis framework, not a signal service and not advice. Long calls expire
worthless routinely — that's the base case, not the tail. Size accordingly, and verify
every number against your broker before putting money behind it.
