# VaultBags agent skills

Skills for AI agents to read the [VaultBags](https://vaultbags.app) autonomous RWA treasury on Solana: today's frozen allocation and its on-chain receipt, live treasury stats, the market signals behind each decision, the certified Solana RWA registry, the Autonomy Score, and more. Read-only public data over HTTPS; no key, no auth, nothing here can move funds.

## Install

```bash
npx skills add lshades/vaultbags-skills
```

Works with Claude Code, Cursor, Codex and any `skills`-compatible agent. The same skill is also served from the protocol's own domain at [vaultbags.app/skill.md](https://vaultbags.app/skill.md), and the machine surface is discoverable at [vaultbags.app/llms.txt](https://vaultbags.app/llms.txt).

## What's inside

- `skills/vaultbags`: the vault-data skill. REST + MCP endpoints, the ask lane (natural-language questions, x402 paid lane), usage notes and examples.

## More machine surfaces

- MCP server: `POST https://vaultbags.app/api/mcp` (JSON-RPC 2.0, Streamable HTTP)
- OpenAPI 3.1 spec: `GET https://vaultbags.app/api/openapi`
- Solana Agent Kit v2 plugin: `solana-agent-kit-vaultbags` on npm
- The Meter (live protocol numbers): `GET https://vaultbags.app/api/meter`

All data is informational. Nothing in these surfaces predicts returns, is financial advice, or can move funds.
