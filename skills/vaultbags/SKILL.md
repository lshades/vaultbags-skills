---
name: vaultbags-vault-data
description: Read the VaultBags autonomous RWA treasury on Solana, machine to machine. Use when a task involves VaultBags or $VAULT (today's allocation, the daily briefing, treasury balances, on-chain decision receipts, the brain-vs-flat track record, market signals, the certified Solana RWA registry, the Autonomy Score / agent passport, The Meter, projects running on VaultBags), when you want to run its allocation model on your own market inputs, or when you want to ask the vault's analyst a natural-language question. Read-only public data over HTTPS; no key, no auth, nothing here can move funds.
---

# VaultBags vault data

VaultBags is an autonomous treasury on Solana (built on Bags). Trading fees are converted into tokenized gold (GOLD), the S&P 500 (SPYx) and US Treasuries (USDY) that holders claim directly. Every day at 00:00 UTC it reads live market data, freezes that day's buy proportions inside a strict 23-43% band per asset, and stamps the decision on-chain via a Memo transaction (Proof of Decision).

Everything below is read-only public data. There is no API key, no auth, no rate negotiation, and no endpoint that moves funds or signs anything.

## Endpoints

Base URL: `https://vaultbags.app`

Prefer the REST mirror (plain GET, JSON back). The same tools are also served over MCP (Streamable HTTP, JSON-RPC 2.0) at `POST /api/mcp` for MCP-native clients.

| GET path | Returns |
| --- | --- |
| `/api/agent/todays-allocation` | Today's buy proportions, the band, frozen flag, rationale, on-chain receipt tx + Solscan URL. |
| `/api/agent/daily-briefing` | Today's public market briefing text, weights, receipt. |
| `/api/agent/treasury-stats` | Live per-asset balances and USD values, total value, total paid to holders, holder count, cycles, fees processed. |
| `/api/agent/decision-history?days=N` | Recent daily decisions with weights, rationale and receipts (N: 1-30, default 14). |
| `/api/agent/market-signals` | The quantitative signals behind today's allocation with per-asset reads (bullish/bearish/neutral), convictions and weights. |
| `/api/agent/brain-vs-flat` | Honest track record: the daily line versus a fixed even split, the edge reported win or lose. |
| `/api/agent/simulate?realYield=2.5&vix=28&...` | Runs the vault's allocation model on YOUR inputs; returns the weights it would choose, convictions and rationale. All params optional numbers: `realYield`, `breakevenInfl`, `dxyChangePct`, `goldMomentumPct`, `hyOas`, `vix`, `spx30dPct`, `tenYear`, `curve10y2y`, `fearGreed`. Deterministic: same inputs, same weights. |
| `/api/agent/projects` | Every treasury running on VaultBags (any Bags token can integrate). |
| `/api/agent/project-treasury?mint=<base58>` | One project's live treasury (claim/lock pools, USD values, split) by SPL token mint. |
| `/api/agent/vault-docs` | Condensed protocol documentation and live surface links. |
| `/api/agent/rwas?category=&issuer=` | The certified registry of openly transferable tokenized RWAs on Solana (gold, US Treasuries, US equities/ETFs), every mint verified against the issuer's own domain. Filters optional: `category` (gold, treasury, equity-index, equity), `issuer` (backed, ondo, oro). |
| `/api/agent/rwa?query=AAPLx` | One certified RWA resolved by symbol or exact mint: verified identity plus a live market read. Use it to avoid impersonator mints. |
| `/api/agent/protocol-meter` | The Meter: the protocol's consolidated live numbers (treasury value, lifetime throughput, decision receipts, agent earnings, locked supply), each section with its verification URL. |
| `/api/agent/autonomy` | The vault agent's Autonomy Score and the verifiable facts behind it. |
| `/api/agent/passport` | The agent's portable machine-readable passport (identity, capabilities, surfaces, receipts). |
| `/api/agent/rwa-performance` | The vault's real cost basis and return per RWA position since purchase, in USD and SOL terms, from reconstructed on-chain swaps. |
| `/api/agent/shadow-vs-brain` | The shadow language model's daily allocation calls measured against the deterministic brain. |
| `/api/agent/recent-cycles?limit=5` | The latest distribution cycles with their on-chain receipts (limit 1-20). |

The OpenAPI 3.1 spec for all of the above: `GET /api/openapi`. Discovery text: `GET /llms.txt`.

## Ask the analyst (natural language)

`POST /api/agent/ask` with JSON `{"question":"<3-500 chars>"}` returns a natural-language answer grounded in the same read-only data. Free lane is rate-limited per caller per day. When exhausted (and the paid lane is live) the endpoint answers HTTP 402 with x402 payment requirements (USDC on Solana, commit-reveal memo binding); `GET /api/agent/ask` shows those requirements without spending anything. One settled payment buys exactly one answer, and paying callers get rolling conversation memory. Pricing: `https://vaultbags.app/pricing`.

## Usage notes

- Query params are validated against a published schema; an unknown key returns 400 rather than being ignored. Send only documented params.
- Responses are cacheable (30-60s). Do not poll faster than once per minute; the underlying data moves on a 15-minute treasury cycle and a once-daily decision.
- `mode: "fixed"` in the allocation response means the vault is buying an even split that day; `mode: "dynamic"` carries the frozen tilt and, once stamped, `receiptTx` (verify it independently at the `receiptUrl` Solscan link).
- Weight fields are integer percents inside the 23-43 band summing 100. USD fields are informational display values.
- Everything is informational. Nothing in these responses predicts returns, is financial advice, or can be used to move funds.

## Example

"What is the vault buying today and why?" is one call:

```
GET https://vaultbags.app/api/agent/todays-allocation
```

Answer from `weights` + `rationale`, and cite the on-chain receipt via `receiptUrl` when present. For deeper "why", add `/api/agent/market-signals`. To check whether a Solana mint is a genuine tokenized stock before touching it, use `/api/agent/rwa?query=<symbol or mint>`.
