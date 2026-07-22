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
| `/api/agent/treasury-stats` | Live per-asset balances and USD values, total value, total paid to holders, fees processed. `holdersCount` and `cyclesCount` are $VAULT's own; `protocolHoldersTracked` and `protocolCyclesExecuted` span every project integrated with the protocol. Locking is reported as two separate numbers on purpose: `activeLocks` counts lock contracts and `uniqueLockers` counts the wallets holding them, since one wallet can hold several contracts. |
| `/api/agent/decision-history?days=N` | Recent daily decisions with weights, rationale and receipts (N: 1-30, default 14). |
| `/api/agent/market-signals` | The quantitative signals behind today's allocation with per-asset reads (bullish/bearish/neutral), convictions and weights. |
| `/api/agent/brain-vs-flat` | Honest track record: the daily line versus a fixed even split, the edge reported win or lose. |
| `/api/agent/simulate?realYield=2.5&vix=28&...` | Runs the vault's allocation model on YOUR inputs; returns the weights it would choose, convictions and rationale. All params optional numbers: `realYield`, `breakevenInfl`, `dxyChangePct`, `goldMomentumPct`, `hyOas`, `vix`, `spx30dPct`, `tenYear`, `curve10y2y`, `fearGreed`. Deterministic: same inputs, same weights. |
| `/api/agent/projects` | Every treasury running on VaultBags (any Bags token can integrate). |
| `/api/agent/project-treasury?mint=<base58>` | One project's live treasury (claim/lock pools, USD values, split) by SPL token mint. |
| `/api/agent/evaluate?mint=<base58>` | Evaluate ANY token running on VaultBags as an agent: Autonomy Score, passport and treasury. The score splits into `protocolScore` (guarantees every VaultBags agent inherits) and `projectScore` (what that token earned itself). A fresh launch reads high on the protocol side and 0 on its own, so never read the composite alone as a track record. |
| `/api/agent/vault-docs` | Condensed protocol documentation and live surface links. |
| `/api/agent/rwas?category=&issuer=` | The certified registry of openly transferable tokenized RWAs on Solana (gold, US Treasuries, US equities/ETFs), every mint verified against the issuer's own domain. Filters optional: `category` (gold, treasury, equity-index, equity), `issuer` (backed, ondo, oro). |
| `/api/agent/rwa?query=HOODx` | One certified RWA resolved by symbol or exact mint: verified identity, a live market read, and for tokenized equities the underlying's oracle price plus the token's premium/discount (`premiumPct`). Use it to avoid impersonator mints. |
| `/api/agent/protocol-meter` | The Meter: the protocol's consolidated live numbers (treasury value, lifetime throughput, decision receipts, agent earnings, locked supply), each section with its verification URL. |
| `/api/agent/autonomy` | The vault agent's Autonomy Score and the verifiable facts behind it. |
| `/api/agent/passport` | The agent's portable machine-readable passport (identity, capabilities, surfaces, receipts). |
| `/api/agent/rwa-performance` | The vault's real cost basis and return per RWA position since purchase, in USD and SOL terms, from reconstructed on-chain swaps. |
| `/api/agent/shadow-vs-brain` | The shadow language model's daily allocation calls measured against the deterministic brain. |
| `/api/agent/recent-cycles?limit=5` | The latest distribution cycles with their on-chain receipts (limit 1-20). |
| `/api/agent/monthly-reports?months=12` | The agent's closed books, one per calendar month, each committed on-chain (limit 1-24). |
| `/api/agent/proof-of-reserves` | Proof of Reserves: the reserve wallets, their certified issuers and live on-chain balances, plus the decision receipts and value paid to holders. |
| `/api/agent/verify-claim?tx=<sig>` | Verify one holder claim against the on-chain Merkle root: the committed record, its proof, the day's root, and the on-chain memo that stamped it. |

The OpenAPI 3.1 spec for all of the above: `GET /api/openapi`. Discovery text: `GET /llms.txt`.

## Verify a payout yourself (no trust required)

Every holder claim is a leaf in a daily Merkle tree whose root is stamped on-chain. You verify a payout without trusting this API: `GET /api/proof/claim/<tx>` returns the claim's committed record, its Merkle proof, the day's root and the on-chain memo; `GET /api/proof/claims/<YYYY-MM-DD>` returns the full committed set so you can rebuild the root yourself and confirm nothing was hidden or altered. A self-contained script (no dependencies) does all three checks and points you at the on-chain memo: `curl -s https://vaultbags.app/verify-claim.mjs > verify-claim.mjs && node verify-claim.mjs <claim_tx>`.

## Ask the analyst (natural language)

`POST /api/agent/ask` with JSON `{"question":"<3-500 chars>"}` returns a natural-language answer grounded in the same read-only data. Free lane is rate-limited per caller per day. When exhausted (and the paid lane is live) the endpoint answers HTTP 402 with x402 payment requirements (USDC on Solana, commit-reveal memo binding); `GET /api/agent/ask` shows those requirements without spending anything. The 402 `accepts[]` lists every option currently offered, including a prepaid credit pack when available: one payment mints a one-time credit token, spent on later calls via the `X-CREDIT-TOKEN` header with no further transactions. One settled payment buys exactly one answer (or one pack), and paying callers get rolling conversation memory. Pricing: `https://vaultbags.app/pricing`.

## Paid data products (same x402 flow)

`GET /api/agent/ledger`: the ledger, the complete receipted decision history (every daily allocation since inception with raw signals, convictions, drivers, rationale and its on-chain receipt) plus the brain-vs-flat and shadow measurement series, one machine-readable export. `GET /api/agent/rwa-registry-live`: every certified RWA with a live market read in one call. Both answer 402 with their exact requirements when called bare, and accept `X-PAYMENT` or `X-CREDIT-TOKEN`.

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
