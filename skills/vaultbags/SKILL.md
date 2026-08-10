---
name: vaultbags-vault-data
description: Read the VaultBags autonomous RWA treasury on Solana, machine to machine. Use when a task involves VaultBags or $VAULT (today's allocation, the daily briefing, treasury balances, on-chain decision receipts, the brain-vs-flat track record, market signals, the certified Solana RWA registry, the Autonomy Score / agent passport, The Meter, projects running on VaultBags), when you want to run its allocation model on your own market inputs, or when you want to ask the vault's analyst a natural-language question. Read-only public data over HTTPS; no key, no auth, nothing here can move funds.
---

# VaultBags vault data

VaultBags is an autonomous treasury on Solana (built on Bags). Trading fees are converted into tokenized gold (GOLD), the S&P 500 (SPYx) and US Treasuries (USDY) that holders claim directly. Every day at 00:00 UTC it reads live market data, freezes that day's buy proportions inside a strict 23-43% band per asset, and stamps the decision on-chain via a Memo transaction (Proof of Decision).

Everything below is read-only public data. There is no API key, no auth, no rate negotiation, and no endpoint that moves funds or signs anything.

## Endpoints

Base URL: `https://vaultbags.app`

Prefer the REST mirror (plain GET, JSON back). The same tools are also served over MCP (Streamable HTTP, JSON-RPC 2.0) at `POST /api/mcp` for MCP-native clients. The MCP server also exposes prompts (one-click actions like `verify_payout` and `audit_agent`) and resources (addressable read-only snapshots like `vaultbags://proof/reserves`), so an MCP client shows a menu, not just a tool list.

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
| `/api/agent/autonomy-spec` | The Autonomy Score's own rules: dimensions, weights, formulas, input sources and how verifiable each one is. Versioned. |
| `/api/agent/passport` | The agent's portable machine-readable passport (identity, capabilities, surfaces, receipts). |
| `/api/agent/rwa-performance` | The vault's real cost basis and return per RWA position since purchase, in USD and SOL terms, from reconstructed on-chain swaps. |
| `/api/agent/shadow-vs-brain` | The shadow language model's daily allocation calls measured against the deterministic brain. |
| `/api/agent/recent-cycles?limit=5` | The latest distribution cycles with their on-chain receipts (limit 1-20). |
| `/api/agent/monthly-reports?months=12` | The agent's closed books, one per calendar month, each committed on-chain (limit 1-24). |
| `/api/agent/proof-of-reserves` | Proof of Reserves: the reserve wallets, their certified issuers and their on-chain balances, plus the decision receipts and value paid to holders. Balances are read in one request and returned with the Solana `slot` they were read at, so the total is reproducible digit for digit rather than approximately; `reconciliation` reports any divergence instead of smoothing it. |
| `/api/agent/treasury-history?days=<N>&points=<P>` | The treasury's value and per-asset balances over time. Samples are spread across the WHOLE window, not taken from its newest end, so a 90-day request describes 90 days. Descriptive history, never a projection. |
| `/api/agent/holder-distribution` | How many distinct wallets hold the token (not token accounts; protocol wallets and pools excluded), how that changed over a day and a week, how many are locking, and how concentrated supply is across the largest wallets. Aggregates only, no addresses. Concentration is null rather than approximated when the scan behind it could not complete. |
| `/api/agent/recent-activity` | Recent trading in aggregate for the last day and week: trades, buys, sells, and distinct buyers, sellers and traders. `truncated` marks a window whose counts are a floor rather than a total. |
| `/api/agent/payout-integrity` | Every holder payout ever recorded, looked up on Solana and counted: confirmed by the chain, rejected by it, or no longer carried in the answering node's history. Recomputed on read, so it never goes stale. `allLanded` is true only when every payout was found AND accepted, because "not found" is a fact about the node and never counts as success. If the ledger or the chain cannot be read in full, no partial figure is served. |
| `/api/agent/lock-tier?days=<N>&wallet=<addr>` | The tier a lock term reaches, and the tier a wallet currently holds. Both parameters optional and independent; with neither, the tier table. The tier comes from the TERM signed for, measured from the lock's on-chain creation to its unlock. It is derived on read, never stored, and grants nothing: the boost multiplier does not read it, and the same term reaches the same tier at any balance. |
| `/api/agent/verify-claim?tx=<sig>` | Verify one holder claim against the on-chain Merkle root: the committed record, its proof, the day's root, and the on-chain memo that stamped it. |
| `/api/agent/lock-boost?amount=<N>` | Lock boost estimate with live data: current % of circulating locked, the shared multiplier (the payout formula, capped 1.5x), and both values after locking N more tokens. Omit `amount` for the current state. |
| `/api/agent/verify-day?period=<YYYY-MM-DD>` | Verify a whole day of the claim ledger: every committed claim, the root rebuilt live, the root stamped on Solana, whether they match, and the treasury signer to check. Omit `period` for the latest stamped day. |
| `/api/agent/verify-decision?date=<YYYY-MM-DD>` | Verify one daily allocation decision against the hash stamped on-chain that day: the payload the receipt commits to, both hashes, the anchoring transaction, and the wallet that must have signed it. Omit `date` for today. |

| `/api/agent/liquidity` | The liquidity the protocol has built into its own pool, and whether any of it can be withdrawn. `lock.allLocked` is read live from the pool position and is true only when it reports zero unlocked liquidity, so nobody can take any of it out, including the protocol. Two money figures that are not interchangeable: `builtIntoLiquidity` is a cost basis, each deposit priced at the moment it happened, which does not move with the market; `positionNow` is what that liquidity represents at current prices, which does. `feesCompounded` is what the locked position earned from trades and had put straight back in. Rebuilt from one public wallet's history on every read, so all of it can be recomputed from Solana. |
| `/api/agent/supply` | Both figures that go by "supply", because using the wrong one silently produces a wrong percentage. `marketSupply` is the total minted, which is what market caps are computed against and what the rest of the ecosystem reports, since tokens in a pool are still tradeable by anyone. `distributedSupply` subtracts the pool authorities' balances and is what holder shares, the leaderboard and the lock boost use, because tokens inside a pool belong to no wallet that can claim. |
| `/api/agent/raffle` | The holder raffle's public state: window, weighted ticket total, participant count, prizes and claim grace. Tickets are weighted by how long a holder actually held across the window rather than by a balance at one instant, so buying just before the close buys none, and locked tokens count. The draw's randomness comes from a public beacon whose round is fixed by the window's end date, before the entrant list can be known. Aggregate only: entrants are never listed. |

The OpenAPI 3.1 spec for all of the above: `GET /api/openapi`. Discovery text: `GET /llms.txt`.

## Verify a payout yourself (no trust required)

Every holder claim is a leaf in a daily Merkle tree whose root is stamped on-chain. You verify a payout without trusting this API: `GET /api/proof/claim/<tx>` returns the claim's committed record, its Merkle proof, the day's root and the on-chain memo; `GET /api/proof/claims/<YYYY-MM-DD>` returns the full committed set so you can rebuild the root yourself and confirm nothing was hidden or altered. A self-contained script (no dependencies) does all three checks and points you at the on-chain memo: `curl -s https://vaultbags.app/verify-claim.mjs > verify-claim.mjs && node verify-claim.mjs <claim_tx>`.

The monthly closed books verify the same way: `GET /api/reports/<YYYY-MM-01>` returns the exact stored payload plus its receipt, and its sha256 is committed in a TREASURY-signed memo. Recompute it yourself with `curl -s https://vaultbags.app/verify-report.mjs > verify-report.mjs && node verify-report.mjs <YYYY-MM-01>`.

## Verify the decision itself (no trust required)

Payouts describe what already happened; the decision receipt covers what the agent chose to do, before it acted. Each UTC day the frozen allocation is hashed and stamped on-chain in a treasury-signed memo. `GET /api/proof/decision/<YYYY-MM-DD>` returns the exact payload that hash commits to (the date, the three weights, and the market signals the model read), the hash recomputed live from the stored record, the hash written at stamping time, the anchoring transaction, and the wallet that must have signed it; `GET /api/proof/decisions` indexes the days on record and marks which carry an anchor. Serialize the payload canonically (sort object keys recursively, keep arrays in order, skip undefined, serialize primitives with `JSON.stringify`) and sha256 the UTF-8 string: it must equal the stamped hash, and the memo read from Solana must end in that same digest and be signed by the published treasury wallet. A memo signed by any other wallet proves nothing. A self-contained script does it end to end and reads the anchor straight from an RPC of your choosing: `curl -s https://vaultbags.app/verify-decision.mjs > verify-decision.mjs && node verify-decision.mjs <YYYY-MM-DD>`.

## Act for a holder, without ever touching their keys

Everything above is public and needs no credentials. To answer questions about a
SPECIFIC holder ("what can I claim right now", "when does my lock end"), you do
not ask for their wallet and you never receive their keys. The holder delegates
to a burner wallet you control.

1. The holder signs a delegation naming your burner as their delegate. Nothing
   about it grants authority over their tokens: it is an off-chain signature (or
   a dust transfer they send themselves), recorded and revocable.
2. Your burner signs in normally (Sign-In With Solana) and gets a session token.
3. You call the holder endpoints with that token. The server resolves WHOSE data
   this is from the stored delegation and never from anything you send it, so a
   wallet address in your request cannot redirect the answer.

What a delegation carries, and what it refuses:

- **claim** is always granted, because claiming is what delegation is for.
- **intelligence** (the holder's own dashboard) is granted by default.
- **governance** and **raffle** are OFF unless the holder signed for them, so an
  agent does not vote or enter draws on somebody's behalf by accident.
- Creating governance proposals and any creator or admin operation are NOT
  delegable at all, by any means.

If you are an agent rather than a person, say so when you request the
delegation. An agent delegation is signed scope by scope: the holder reads a
line per permission and signs that exact text, and the server stores what they
signed. When the delegation is confirmed, the permissions are copied from that
stored record and never from anything sent at confirmation time, so what was
signed is what is enforced. Agents get claim and intelligence; governance and
raffle require the holder to have signed for them explicitly. Only the signed
message flow supports this, because a payment cannot express a list of scopes.

The guard that matters is on the money, not on the paperwork: for a claim, the
destination is always the holder's own wallet, derived on the server and bound
to the exact transaction they approved. Your burner pays the gas and the account
rent, and cannot receive the assets even if it builds the request itself.

The holder is never locked in. They can revoke with their own signature at any
time, without your cooperation and without connecting to anything you control,
and delegating to somebody else revokes you automatically.

## Ask the analyst (natural language)

`POST /api/agent/ask` with JSON `{"question":"<3-500 chars>"}` returns a natural-language answer grounded in the same read-only data. Free lane is rate-limited per caller per day. When exhausted (and the paid lane is live) the endpoint answers HTTP 402 with x402 payment requirements (USDC on Solana, commit-reveal memo binding); `GET /api/agent/ask` shows those requirements without spending anything. The 402 `accepts[]` lists every option currently offered, including a prepaid credit pack when available: one payment mints a one-time credit token, spent on later calls via the `X-CREDIT-TOKEN` header with no further transactions. One settled payment buys exactly one answer (or one pack), and paying callers get rolling conversation memory. Pricing: `https://vaultbags.app/pricing`.

## Score an agent without us

`GET /api/agent/autonomy-spec` publishes the Autonomy Score's own rules: every dimension with its weight and scope, the formula behind each 0-100 sub-score, which endpoint each input comes from, and how independently verifiable that input is. Some inputs are anchored on-chain; one is our own assertion and says so.

Fetch `GET /api/agent/autonomy` for the raw facts, apply the published formulas, take the weighted mean over the active weights, and compare with the score in that same response. If the two disagree, we are wrong and that is the finding. The spec is versioned, and scores computed under different versions are not comparable.

## Install it in one line

`claude mcp add --transport http vaultbags https://vaultbags.app/api/mcp`

For a client that takes a config file, the same server is two fields. The `type` is required: an entry with a `url` and no `type` is read as a local command and skipped.

```json
{ "mcpServers": { "vaultbags": { "type": "http", "url": "https://vaultbags.app/api/mcp" } } }
```

## Use it from an OpenAI-compatible client

Already speak the OpenAI chat API? Point any client at base URL `https://vaultbags.app/api/v1` with model `vaultbags-agent` (Cursor, Cline, Continue, the `openai` SDK). `GET /api/v1/models` lists it.

It is a facade, not a second door: every request forwards to `/api/agent/ask`, so the lanes, the daily limits, the x402 settlement and the prompt-injection filter are the same ones described above. The API key field is optional and is passed through, so a VaultBags session token there buys a holder's higher allowance; a placeholder key is harmless. Streaming is not supported yet and is refused explicitly rather than faked, and `usage` is reported only when the agent actually returned token counts.

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
