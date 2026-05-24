# CLAUDE.md

## Repository Overview

This is a Python CLI project — an options trading screening agent powered by the Anthropic Claude API. The single application file (`options_agent.py`) provides an interactive terminal-based chatbot that walks users through a structured 5-filter framework for evaluating call options setups.

## File Structure

```
mattbaranello/
├── options_agent.py   # Entire application — CLI loop, streaming, system prompt
├── requirements.txt   # anthropic>=0.40.0
└── README.md          # GitHub profile README (not project docs)
```

## Running the Agent

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...
python options_agent.py
```

Type a ticker or question at the `You:` prompt. Type `quit`, `exit`, or `q` to exit.

## Key Code Patterns

### Model and API Usage

- **Model**: `claude-opus-4-6`
- **Streaming**: Uses `client.messages.stream()` — output is printed chunk-by-chunk as it arrives
- **Extended thinking**: `thinking={"type": "adaptive"}` is passed on every request, letting the model reason before responding
- **Max tokens**: 8192

### Prompt Caching

The system prompt is cached using Anthropic's prompt caching feature:

```python
system=[{
    "type": "text",
    "text": SYSTEM_PROMPT,
    "cache_control": {"type": "ephemeral"},
}]
```

This reduces latency and cost on every turn after the first. Do not remove `cache_control` — the system prompt is large (~3KB) and caching it matters.

### Conversation History

Conversation turns are stored in `conversation: list[dict]` with `{"role": "user"|"assistant", "content": str}` entries. Standard multi-turn format — append user message, stream response, append assistant message.

### First-Turn Disclaimer Injection

On the first user turn, a `[Note: ...]` is appended to the prompt to ensure the agent outputs the disclaimer. After streaming, the stored user message is replaced with the original (clean) text so the injected note doesn't pollute conversation history:

```python
conversation.append({"role": "user", "content": prompt})   # injected
...
conversation[-1] = {"role": "user", "content": user_input} # restore clean
```

## The 5-Filter Screening Framework

The system prompt encodes a structured analysis workflow the agent runs for every trade evaluation:

1. **IV Assessment** — IV Rank vs 52-week range; flags expensive options above 50%
2. **Catalyst Identification** — Earnings, FDA dates, macro events within 2–6 weeks
3. **Technical Analysis** — Support/resistance, RSI, MAs, trend context
4. **Options Chain Quality** — OI, bid-ask spread, volume, Greeks (delta/theta/vega)
5. **Risk/Reward & Position Sizing** — Breakeven, profit targets, max loss, contract quantities

The agent outputs a formatted block with emoji headers and an overall letter grade (A–F).

## Development Conventions

- No test suite — validate changes by running the CLI manually
- No linter config — standard Python style (PEP 8)
- The entire application is self-contained in `options_agent.py`; keep it that way unless scope grows significantly
- The system prompt (`SYSTEM_PROMPT`) and disclaimer (`DISCLAIMER`) are module-level constants — edit them directly for behavioral changes
- `BANNER` is a cosmetic ASCII header printed at startup; update it if the model name changes

## Changing the Model

The model is referenced in three places — update all three together:

1. `stream_response()` → `model="claude-opus-4-6"` argument
2. `BANNER` string (cosmetic label)
3. Module docstring at the top of the file

## Dependencies

Only one runtime dependency: `anthropic>=0.40.0`. The SDK must support `thinking` parameter and prompt caching (both available since SDK v0.40).
