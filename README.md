# catai-skill

Agent skill for **CATAI 智猫** — 5,000 on-chain cat NFTs on BNB Smart Chain.
Agent-ID holders only. Each mint costs 0.001 BNB and entitles the minter to claim
160 CAT AI (CATAIT).

## Install

```sh
git clone https://github.com/<you>/catai-skill
# then copy skills/catai-mint-claim into your agent's skills directory
```

The skill drives the Binance Agentic Wallet CLI (`baw`, developer mode) on BSC.

## What the agent can do with it

- mint CATAI (`mint(uint256)`), 0.001 BNB each, 1–50 per transaction
- check a wallet's cats, mints remaining, total minted
- claim CAT AI (`claim()`, 0.002 BNB fee per claim call, 160 CAT AI per cat)
- read the current price/state from chain before every transaction

## Files

| Path | What |
|---|---|
| `skills/catai-mint-claim/SKILL.md` | the skill itself (preflight, contracts, read-only checks) |

## Contracts (BNB Smart Chain, chainId 56)

- CATAI NFT (ERC-721): `0x4aB809dc6e0C62a82d5Bff2F128C9FA30D752297`
- CAT AI token (CATAIT): `0xb365da49715436782b0B80B9fe4CEAA7683029b6`
- Agent ID registry (SPACE ID): `0x60B9262456dE011F7d15a54CdD8c48bAaa8B022F`

Site: https://cataiagent.xyz · Spec: https://cataiagent.xyz/api/mint-manifest.json

Source-verified on Sourcify (exact match) and BscScan. **Not third-party audited.**
Nothing here is financial advice; verify price and supply on chain before acting.
