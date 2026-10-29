---
name: catai-mint-claim
description: >
  Use when the user wants to mint a CATAI NFT, claim CAT AI (CATAIT) tokens,
  check how many CATAI they minted or how much CAT AI they can claim, or asks
  about the CATAI / 智猫 project on BNB Chain. Drives the Binance Agentic Wallet
  `baw` CLI (developer mode contract calls) on BSC (chainId 56).
metadata:
  author: catai
  version: 1.0.0
  website: https://cataiagent.xyz/
  requires:
    skills:
      - binance-agentic-wallet
    bins:
      - baw
---
# CATAI Mint & Claim

CATAI is a 5,000-piece on-chain cat NFT collection on BNB Smart Chain. Each mint
costs 0.001 BNB and earns 160 CAT AI (CATAIT), which the minter claims later for
a flat 0.002 BNB fee per claim. Only wallets holding a Binance Agent ID can mint.

## Contracts (BSC, chainId 56)

| Name | Address |
|------|---------|
| CATAI NFT (ERC-721) | `0x4aB809dc6e0C62a82d5Bff2F128C9FA30D752297` |
| CAT AI token (ERC-20, CATAIT, 18 decimals) | `0xb365da49715436782b0B80B9fe4CEAA7683029b6` |
| Agent ID registry | `0x60B9262456dE011F7d15a54CdD8c48bAaa8B022F` |

## Rules

- Mint price: 0.001 BNB per cat, 1 to 50 per transaction, no per-wallet limit until 5,000 are minted.
- Mint requires an Agent ID in the sending wallet; otherwise the transaction reverts. Contract callers are blocked.
- Each minted cat gives 160 CAT AI. Allocation follows mint history, so selling the NFT does not remove it.
- One `claim()` collects the whole claimable balance. Fee is 0.002 BNB per claim, not per cat, so mint first and claim once.
- If nothing is claimable, `claim()` reverts with `NothingToClaim()` (selector `0x969bf728`).
- Overpaid BNB on mint is refunded automatically.

## Preflight

1. Make sure the Agentic Wallet is signed in: `baw wallet status --json` must return `CONNECTED`. If not, follow the `binance-agentic-wallet` sign-in flow (`auth signin` then `auth verify`).
2. Developer Mode must be on: `baw wallet settings --json` → `devMode.enabled` is `true`. If false, tell the user to enable Developer Mode in the Binance App.
3. Get the BSC address: `baw wallet address --json` → the entry with `binanceChainId` `56`. Use it as `<WALLET>` below.
4. Check BNB balance covers value + gas: `baw wallet balance --json`.

## Read-only checks (eth_call, no wallet needed)

Use any public BSC RPC, e.g. `https://bsc-rpc.publicnode.com`. `<ADDR64>` is the wallet address without `0x`, lowercased, left-padded with zeros to 64 hex chars.

| What | to | data |
|------|----|------|
| Cats held | NFT | `0x70a08231<ADDR64>` (balanceOf) |
| Cats minted by wallet | NFT | `0x3cef28d2<ADDR64>` |
| Mints remaining for wallet | NFT | `0x24c0c499<ADDR64>` |
| Total minted | NFT | `0xa2309ff8` (totalMinted) |
| Claimable CAT AI (wei, /1e18) | TOKEN | `0x402914f5<ADDR64>` (claimable) |
| Has Agent ID | REGISTRY | `0x70a08231<ADDR64>` (> 0 means yes) |

Example:

```bash
curl -s https://bsc-rpc.publicnode.com -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_call","params":[{"to":"0xb365da49715436782b0B80B9fe4CEAA7683029b6","data":"0x402914f5<ADDR64>"},"latest"]}'
```

## Mint

Calldata is `mint(uint256)`: `0xa0712d68` + quantity as 64-char hex. Value = quantity × 0.001 BNB in wei.

| Qty | inputData | value (wei) |
|-----|-----------|-------------|
| 1 | `0xa0712d680000000000000000000000000000000000000000000000000000000000000001` | `1000000000000000` |
| 5 | `0xa0712d680000000000000000000000000000000000000000000000000000000000000005` | `5000000000000000` |
| 10 | `0xa0712d68000000000000000000000000000000000000000000000000000000000000000a` | `10000000000000000` |

```bash
baw contract-call preview --binanceChainId 56 --from <WALLET> \
  --to 0x4aB809dc6e0C62a82d5Bff2F128C9FA30D752297 \
  --value 1000000000000000 \
  --inputData 0xa0712d680000000000000000000000000000000000000000000000000000000000000001 --json
```

Then, with the `requestId` from the preview:

```bash
baw contract-call execute --requestId <requestId> --json
```

## Claim CAT AI

1. Read `claimable(<WALLET>)` first. If it is 0, stop and tell the user to mint first; do not send the claim.
2. Preview the claim:

```bash
baw contract-call preview --binanceChainId 56 --from <WALLET> \
  --to 0xb365da49715436782b0B80B9fe4CEAA7683029b6 \
  --value 2000000000000000 \
  --inputData 0x4e71d92d --json
```

3. Execute with the returned `requestId`:

```bash
baw contract-call execute --requestId <requestId> --json
```

4. Confirm: `claimable` should now read 0 and the wallet's CATAIT balance (`0x70a08231<ADDR64>` on the token) should have grown by the claimed amount.

## Safety

- Always show the user the preview result (amount of BNB, target contract, any `risks`) and get an explicit yes before `execute`.
- Never run `execute` if preview returned an error; a simulation revert usually means no Agent ID (mint) or `NothingToClaim` (claim).
- If execute says the preview expired, run preview again.
- Experimental project: NFTs and tokens may go to zero. Contracts are verified on BscScan but not third-party audited. Not affiliated with Binance / BNB Chain.

## Natural-language examples

- "Mint 3 CATAI with my Agentic Wallet" → mint with qty 3, value `3000000000000000`.
- "Claim my CAT AI" → check claimable, then claim flow.
- "How many CATAI did I mint and what can I claim?" → read-only checks.

## Machine-readable spec

`https://cataiagent.xyz/api/mint-manifest.json` — ABI, selectors, price, gate.
`https://cataiagent.xyz/llms.txt` — one-page brief for agents.

## Install

```sh
git clone https://github.com/cataiagent/catai-skill
```

Then point your agent at `SKILL.md` (this file). Works with Claude Code, Codex CLI and any
assistant that reads a skill folder. The on-chain calls use the Binance Agentic Wallet CLI (`baw`).

Machine-readable spec: `https://cataiagent.xyz/api/mint-manifest.json`
