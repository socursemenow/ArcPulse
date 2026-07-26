# ArcPulse

**Real-data dashboard for [Arc](https://arc.network), Circle's stablecoin-native L1.**
One file, no build step, everything live from Arc's own chain.

Arc is unusual: **gas is paid in USDC**, EURC is first-class, and it has a **native on-chain FX engine (StableFX)**. ArcPulse turns those unique properties into things you can actually see.

Built for the **Build on Arc** hackathon.

---

## What it shows (4 tabs)

| Tab | What it is | Data source |
|---|---|---|
| **Peg Watch** | Bubble map of top stablecoins, positioned by live drift from $1.00 | CoinGecko (mainnet prices) |
| **Arc Whale Map** | Live USDC/EURC holder concentration on Arc, ranked & paginated | Arc Blockscout API |
| **Gas $ (Arc)** | Real-dollar gas spend, a live fee ticker, and a paginated full tx history | Arc Blockscout API |
| **StableFX Tape** | Every on-chain USDC↔local-stablecoin FX trade, with its **executed rate vs the ECB interbank mid** (the real spread paid) | Arc RPC (Multicall3) + Frankfurter/ECB |

The **StableFX Best-Execution Tape** is the standout: because Arc settles FX atomically on-chain, the executed rate of every trade is public. ArcPulse reconstructs it and benchmarks it against the bank mid, a provable best-execution view that **can't exist on any other chain**.

### Why gas-in-dollars matters
On every other L1, showing a fee in fiat needs a price oracle and is an estimate. On Arc, gas *is* USDC, so `fee.value` is already a dollar figure. ArcPulse quotes network cost in real dollars with no conversion.

---

## Run it

No install, no `npm`:

```bash
npx serve .
# then open index.html (served at the root URL)
```

Or open the file with the **Live Server** VS Code extension. Every tab has a labeled demo-data fallback if an API is unreachable, so it always renders.

---

## How it works (tech)

- **Single-file app**: plain HTML + CSS + vanilla JS in `index.html`, scoped under `#peg-watch-root`, one IIFE, zero dependencies.
- **Reads Arc directly**: Blockscout REST API for holders/txs/stats, and JSON-RPC for contract reads.
- **StableFX tape** decodes `getTradeDetails()` from the `FxEscrow` contract. All trades on a page are fetched in **one `eth_call` via Multicall3** (canonical address, deployed on Arc) so it never trips the public RPC's rate limit, at any page size.
- **BigInt math** for holder balances (they exceed JS's safe-integer range); percentages are decimal-agnostic.
- **Decimals handled correctly**: Arc's native gas is 18-decimal; the USDC/EURC ERC-20 interface is 6-decimal.
- **Configurable pagination** (10/20/30/40/50 per page) on every list, with a buffered cursor paginator for Blockscout's fixed-size pages.
- Accessible (ARIA tabs, keyboard nav, reduced-motion), theme is a gold-on-navy terminal aesthetic.

---

## Roadmap: from dashboard to product

ArcPulse today is the **observability layer** (the "eyes"). Next is the **ArcPulse Treasury Agent** (the "hands"):

> an autonomous agent that holds a Circle Wallet and acts on ArcPulse's live signals: rebalances USDC/EURC, pays invoices, and settles **only when the FX spread is tight, gas is cheap, and the peg is healthy**, with a visible, human-readable decision log.

- **CP2**: agent runtime (Node + TypeScript + viem) + one real testnet USDC action triggered by a live signal
- **Final**: full watch → decide → act → log loop, 2-3 policies, deployed MVP + demo

Planned stack: **Circle Wallets**, **App Kits (Send)**, USDC/EURC, optional **Paymaster/Nanopayments**. Fits both the **DeFi** and **Agentic Economy** tracks.

---

## Arc Testnet reference

| | |
|---|---|
| Chain ID | `5042002` |
| RPC | `https://rpc.testnet.arc.network` |
| Explorer | `https://testnet.arcscan.app` (Blockscout) |
| USDC (native gas + ERC-20) | `0x3600000000000000000000000000000000000000` |
| EURC | `0x89B50855Aa3bE2F677cD6303Cec089B5F319D72a` |
| StableFX escrow | `0x867650F5eAe8df91445971f14d89fd84F0C9a9f8` (proxy) |
| Multicall3 | `0xcA11bde05977b3631167028862bE2a173976CA11` |
| Consensus | Malachite (~350ms deterministic finality) |

> Testnet data reflects faucet/dev activity: real *data*, not yet real *economic* activity. StableFX settlement is permissioned; ArcPulse **measures** it from public on-chain state.
