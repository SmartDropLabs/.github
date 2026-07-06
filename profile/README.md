<div align="center">

# SmartDrop <img width="20" height="20" alt="smartdrop-logo" src="https://github.com/user-attachments/assets/a19380bb-3241-4611-971d-a1f2b197745e" />


**A liquidity-oriented airdrop mechanism on Stellar.**

Participants lock Stellar assets in Soroban farming pools and accrue airdrop credits over time — instead of a one-off "click to claim" snapshot.

[Frontend](https://github.com/SmartDropLabs/smartdrop-frontend) · [Backend](https://github.com/SmartDropLabs/smartdrop-backend) · [Contracts](https://github.com/SmartDropLabs/smartdrop-contracts)

</div>

---

## What this is

Most airdrops optimize for reach: snapshot every wallet that ever touched a token, distribute proportionally, done. That rewards drive-by activity as much as real commitment, and it's a one-time event with no ongoing incentive to stick around.

SmartDrop reframes distribution around **skin in the game**. Instead of a snapshot, participants **lock** Stellar assets into a **farming pool** for a minimum period. While locked, they accrue **credits** as a function of amount × time × a configurable rate (plus optional boosts). Credits determine the eventual airdrop allocation. The result: the people who get rewarded are the people who actually committed liquidity and time, not the people who happened to hold a token on a specific block.

This is a **mechanism**, not a finished product — a pattern teams can study, fork, deploy on testnet, and adapt to their own token design. It is explicitly **not** presented as audited, production-hardened infrastructure. See each repo's README for its own security notes.

### How it works, concretely

1. A **factory** contract (Soroban/Rust) deploys or registers isolated **farming pool** instances, one per campaign.
2. Each pool accepts a configured Stellar asset. Participants call **lock**, and the pool tracks their position.
3. Credits accrue continuously based on locked amount, elapsed time, the pool's rate, and any active boost multiplier.
4. Once the minimum lock period has elapsed, participants can **partially or fully unlock** — withdrawing their original assets while keeping the credits already earned.
5. A backend service indexes pool and position state, serves it to the frontend, and handles off-chain concerns (price data, webhooks, alerting) that don't belong on-chain.
6. A Next.js frontend lets participants connect a Freighter wallet, browse pools, lock/unlock assets, and track their position, history, and rank on a leaderboard.

---

## Architecture

SmartDrop is split into three repositories, each independently deployable:

| Repo | Role | Stack |
|---|---|---|
| [**smartdrop-contracts**](https://github.com/SmartDropLabs/smartdrop-contracts) | On-chain logic: the `factory` and `farming-pool` Soroban contracts — locking, credit accrual, boosts, pausing, admin controls | Rust, Soroban SDK |
| [**smartdrop-backend**](https://github.com/SmartDropLabs/smartdrop-backend) | Off-chain services: multi-source price oracle (Stellar DEX → CoinGecko → CoinMarketCap fallback chain), signed webhook delivery for lifecycle events, API-key auth, rate limiting | Node.js, Express, Redis |
| [**smartdrop-frontend**](https://github.com/SmartDropLabs/smartdrop-frontend) | The web app: dashboard, farm pools, lock/unlock flows, history, leaderboard, live contributor data — talks to Soroban RPC directly and to the backend for off-chain data | Next.js, TypeScript, Chakra UI, Tailwind CSS |

Data flows roughly like this: the **frontend** reads pool and position state directly from **Soroban RPC** (via Freighter-signed transactions for writes), while the **backend** independently indexes chain events, serves price data the contracts don't have, and pushes webhook notifications when pool events fire. The three repos share no code — they integrate over public interfaces (RPC, REST, webhooks) so each can evolve independently.

---

## Getting started

Each repo has its own setup instructions, but the short version:

```bash
# Contracts — build and test the Soroban pools
git clone https://github.com/SmartDropLabs/smartdrop-contracts
cd smartdrop-contracts/soroban && cargo test --workspace

# Backend — off-chain services
git clone https://github.com/SmartDropLabs/smartdrop-backend
cd smartdrop-backend && npm ci && npm run dev

# Frontend — the web app
git clone https://github.com/SmartDropLabs/smartdrop-frontend
cd smartdrop-frontend && npm ci && npm run dev
```

The frontend runs standalone against Stellar Testnet defaults with no configuration. To see real pool data end-to-end, deploy the factory contract (`smartdrop-contracts`) and point the frontend at it via `NEXT_PUBLIC_FACTORY_CONTRACT_ID`.

---

## Status

SmartDrop is an active work in progress, built in the open. Contract logic, pool economics, and admin/privileged operations have **not** been professionally audited — anyone deploying beyond testnet should run their own review first and treat functions like `pause`, rate changes, and rescues as governance-sensitive.

## Contributing

Each repo takes its own PRs — see the repo's README for setup and contribution guidelines. In general: fork, branch, keep PRs focused, add tests for contract or backend logic changes, and update docs when env vars or deployment steps change.

## Contributors

See each repo's `CONTRIBUTORS.md` (or `/contributors` on the live frontend) for a live, GitHub-API-sourced list of everyone who has actually committed to that repository.
