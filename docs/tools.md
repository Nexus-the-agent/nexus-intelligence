# MCP Tools Reference

## Connection

**Endpoint:** `https://api.intelrefined.com/mcp/`  
**Protocol:** MCP (JSON-RPC 2.0)  
**Auth:** `X-Nexus-API-Key` header  
**Rate Limit:** 100 requests/hour (research tools: 5/hour)

## Tools

### get_market_regime

Current market regime assessment.

**Input:** None required  
**Output:**
```json
{
  "regime": "transition",
  "momentum": "bearish",
  "stress": 42,
  "stress_label": "elevated",
  "drivers": ["credit-equity divergence", "VIX term structure inverted"],
  "confidence": 0.72
}
```

### get_signal_to_noise

Six calibrated market indicators updated daily.

**Input:** None required  
**Output:**
```json
{
  "nci_score": -12,
  "stress_index": 42,
  "price_narrative_divergence": 0.67,
  "narrative_quality": 0.54,
  "capitulation_score": 0.23,
  "thrust_score": 0.31,
  "regime": "cautious"
}
```

### get_ta_snapshot

Technical analysis for any of 79 covered tickers.

**Input:**
```json
{"ticker": "SPY"}
```

**Output:**
```json
{
  "ticker": "SPY",
  "price": 658.42,
  "trend": "bearish",
  "fibonacci_levels": {"support_1": 642, "support_2": 618, "resistance_1": 672},
  "moving_averages": {"sma_50": 651, "sma_200": 612},
  "rsi": 44.2,
  "bull_trigger": "Close above 672 with volume",
  "bear_trigger": "Close below 642"
}
```

### get_nexus_view

Today's market synthesis — the single most important thing in markets today.

**Input:** None required  
**Output:**
```json
{
  "date": "2026-04-09",
  "sharpest_insight": "Credit-equity divergence at 4 standard deviations...",
  "regime": "transition",
  "key_levels": {"SPY": 658, "VIX": 23.87},
  "thesis": "Bear market rally inside contractionary regime"
}
```

### run_research

Launch autonomous multi-model research. Returns immediately with project ID.

**Input:**
```json
{
  "topic": "AI infrastructure capex demand outlook 2026-2027",
  "depth": "standard"
}
```

**Depth options:**
- `quick` (~2 min, ~$0.02) — Existing knowledge + single cloud model
- `standard` (~10 min, ~$0.15) — Web search + 3-pass synthesis
- `deep` (~25 min, ~$2.00) — Council of Experts debate + RAIM validation

**Output:**
```json
{
  "project_id": 170,
  "status": "queued",
  "depth": "standard",
  "estimated_time_seconds": 600
}
```

### get_research_result

Poll for research results. Returns structured findings when complete.

**Input:**
```json
{"project_id": 170}
```

**Output (in progress):**
```json
{
  "project_id": 170,
  "status": "in_progress",
  "phase": "synthesizing",
  "estimated_seconds_remaining": 120
}
```

**Output (complete):**
```json
{
  "project_id": 170,
  "status": "complete",
  "structured_output": {
    "schema_version": "1.0",
    "thesis": "NVDA AI infrastructure demand strengthened with FY2027 consensus at $369B",
    "confidence": 0.75,
    "invalidation": "Hyperscaler capex guidance cut >20% in Q2 2026",
    "time_horizon": "6-12m",
    "findings": [
      {
        "claim": "FY2027 consensus revenue at $369B, +71% YoY",
        "so_what": "Confirms bullish infrastructure thesis",
        "source_trust": "medium",
        "time_bound": "Q2 2026 earnings"
      }
    ],
    "tickers_affected": ["NVDA"],
    "actions": ["Hold existing exposure, avoid adding ahead of May earnings"],
    "risks": ["Custom silicon acceleration from Google TPU v5 and AWS Trainium2"],
    "regime_impact": "Confirms sustained AI infrastructure buildout regime",
    "contradictions": [
      {
        "source_a": "Executive Summary states 85% confidence",
        "source_b": "Detailed assessment shows 60% on earnings execution",
        "resolution": "Use differentiated figures; 60% is operative for near-term"
      }
    ]
  }
}
```

## Example: Full Research Flow

```python
import httpx
import time

BASE = "https://api.intelrefined.com/mcp/"
HEADERS = {
    "Content-Type": "application/json",
    "X-Nexus-API-Key": "your-api-key"
}

# 1. Start research
resp = httpx.post(BASE, headers=HEADERS, json={
    "jsonrpc": "2.0", "id": 1,
    "method": "tools/call",
    "params": {
        "name": "run_research",
        "arguments": {"topic": "copper demand from AI data centers", "depth": "standard"}
    }
})
project_id = resp.json()["result"]["content"][0]["text"]["project_id"]

# 2. Poll for results
while True:
    resp = httpx.post(BASE, headers=HEADERS, json={
        "jsonrpc": "2.0", "id": 2,
        "method": "tools/call",
        "params": {
            "name": "get_research_result",
            "arguments": {"project_id": project_id}
        }
    })
    result = resp.json()["result"]["content"][0]["text"]
    if result["status"] == "complete":
        print(result["structured_output"])
        break
    time.sleep(30)
```
