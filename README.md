# Convallax Testnet Bot

Options trading automation for [Convallax](https://www.convallax.com/predict) — trade calls and puts on Polymarket prediction market outcomes.

## 🌐 Website
https://www.convallax.com/predict

## 🔗 Referral
https://www.convallax.com/predict

## Features
- 🚰 **Auto Faucet** — Claim 1000 test USDC + POL gas per wallet
- 📈 **Options Trading** — Request quotes → commit → fill on-chain
- 🔄 **Volume Farming** — Multi-round trading for Zealy quest completion
- 👛 **Multi-Wallet** — Process unlimited wallets from privateKey.txt
- ⚡ **CLI Menu** — Interactive menu + `--daily` / `--faucet` flags

## Chain
- **Network:** Polygon Amoy Testnet (Chain ID 80002)
- **RPC:** https://rpc-amoy.polygon.technology
- **Explorer:** https://amoy.polygonscan.com

## Contracts
| Contract | Address |
|----------|---------|
| ConvallaxCore | `0xc760F8f6B8830463822be9F68eB10e1b5Dace378` |
| ConvallaxRFQSettlement | `0xC6Eb814Cc01189e20B2DB2D2a22Ed2DcAC404992` |
| ConvallaxOptionToken | `0xd6F69CecCdf846563bE20A860F0A38D3e9324959` |
| USDC (test) | `0xCc544B615B244Cf59A7cB6Cb7d7119ef73A485E5` |

## Setup
```bash
npm install
```

Add your private key(s) to `privateKey.txt` (one per line):
```
0xYOUR_PRIVATE_KEY_HERE
```

## Usage
```bash
# Interactive menu
node index.js

# Auto daily (faucet + farm volume all wallets)
node index.js --daily

# Faucet only
node index.js --faucet
```

### Menu Options
1. **Claim Faucet** — 1000 USDC per wallet
2. **Single Trade** — One trade with first wallet
3. **Farm Volume** — Multiple rounds with first wallet
4. **Farm Volume All** — Multiple rounds all wallets
5. **Check Balance** — USDC + POL for all wallets
6. **Full Daily** — Faucet + farm all wallets

## Trade Flow
1. Fetch active markets from `/v1/markets`
2. Create quote request (off-chain, no gas)
3. Poll for market maker quotes
4. Commit to best quote
5. Approve USDC to Settlement contract
6. Call `fill()` on-chain (atomic: pay premium + mint options)

## Zealy Quests
- 5/25/50/100/500 testnet trades
- $1K/$10K/$20K/$50K/$100K volume
- $2,500 prize pool (top 100 leaderboard)

## Backed by
Alliance DAO

## Contact
@rmndkyl
