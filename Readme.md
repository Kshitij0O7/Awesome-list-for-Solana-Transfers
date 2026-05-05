# Awesome Bitquery Solana Transfers (V1) [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> A curated list of resources, queries, and tutorials for tracking **Solana SPL token & SOL transfers** with the **Bitquery V1 GraphQL API**.

[Bitquery](https://bitquery.io) is the most comprehensive blockchain data platform for Solana — indexing every block, transaction, instruction, and transfer since the genesis block. The **V1 Solana Transfers API** lets you query historical SOL and SPL token movements with a single GraphQL endpoint — no node infrastructure, no parsing, no headaches.

> **Note:** Bitquery V1 (`graphql.bitquery.io`) is the legacy API. For new projects (real-time WebSocket subscriptions, Pump.fun, Raydium, gRPC CoreCast streams), see the [V2 Solana API](https://docs.bitquery.io/docs/blockchain/Solana/). V1 remains the easiest way to query historical, aggregated transfer data with simple `options`/`date` filters.

---

## Contents

- [Why Bitquery for Solana Transfers](#why-bitquery-for-solana-transfers)
- [Quick Start](#quick-start)
- [Authentication](#authentication)
- [Core Query Examples](#core-query-examples)
  - [Latest SOL & SPL transfers](#latest-sol--spl-transfers)
  - [Transfers for a specific wallet](#transfers-for-a-specific-wallet)
  - [Transfers for a specific token (mint)](#transfers-for-a-specific-token-mint)
  - [Aggregate volume over time](#aggregate-volume-over-time)
  - [Top senders / receivers](#top-senders--receivers)
  - [Inbound vs outbound for an address](#inbound-vs-outbound-for-an-address)
  - [Transfers for a transaction signature](#transfers-for-a-transaction-signature)
- [Migration Path: V1 → V2](#migration-path-v1--v2)
- [Tools, IDEs & SDKs](#tools-ides--sdks)
- [Use Cases](#use-cases)
- [Pricing](#pricing)
- [Community & Support](#community--support)
- [License](#license)

---

## Why Bitquery for Solana Transfers

- **Complete Solana history** — Every SOL and SPL transfer indexed since genesis. No 1,000-tx RPC ceiling, no `getSignaturesForAddress` pagination loops.
- **Decoded & enriched** — Transfers come pre-decoded with token metadata (`name`, `symbol`, `decimals`), USD value (`AmountInUSD`), and sender/receiver context.
- **Single GraphQL endpoint** — Replaces dozens of RPC calls. Filter, sort, aggregate, and join in one round-trip.
- **Aggregations built in** — `count`, `sum`, `uniq` over any field, with date bucketing, without dragging raw rows into your app.
- **Multi-chain** — Same schema patterns across Ethereum, BSC, Tron, Cardano, and 40+ other networks.

---

## Quick Start

**V1 GraphQL endpoint:** `https://graphql.bitquery.io`

```bash
curl -X POST https://graphql.bitquery.io \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_BITQUERY_TOKEN" \
  -d '{"query":"{ solana(network: solana) { transfers(options: {limit: 5, desc: \"block.timestamp.unixtime\"}) { transaction { signature } amount currency { symbol address } sender { address } receiver { address } } } }"}'
```

## Authentication

1. Sign up at [account.bitquery.io](https://account.bitquery.io/auth/signup).
2. Generate an access token at [account.bitquery.io/user/api_v2/access_tokens](https://account.bitquery.io/user/api_v2/access_tokens).
3. Send it as a Bearer token in the `Authorization` header.

Full auth guide: [docs.bitquery.io/docs/authorisation/how-to-generate](https://docs.bitquery.io/docs/authorisation/how-to-generate/)

---

## Core Query Examples

Open any of these in the [Bitquery IDE](https://ide.bitquery.io/) to run instantly.

### Latest SOL & SPL transfers

Returns the 10 most recent transfers across the Solana blockchain — both native SOL and SPL tokens.

```graphql
{
  solana(network: solana) {
    transfers(
      options: {limit: 10, desc: "block.timestamp.unixtime"}
    ) {
      block {
        timestamp { time(format: "%Y-%m-%d %H:%M:%S") }
        height
      }
      transaction { signature }
      sender { address }
      receiver { address }
      amount
      currency { symbol address name decimals }
    }
  }
}
```

### Transfers for a specific wallet

Get inflows AND outflows for a single Solana wallet over a date range.

```graphql
{
  solana(network: solana) {
    transfers(
      options: {limit: 100, desc: "block.timestamp.unixtime"}
      date: {since: "2024-01-01", till: "2024-12-31"}
      any: [
        {senderAddress: {is: "9nnLbotNTcUhvbrsA6Mdkx45Sm82G35zo28AqUvjExn8"}}
        {receiverAddress: {is: "9nnLbotNTcUhvbrsA6Mdkx45Sm82G35zo28AqUvjExn8"}}
      ]
    ) {
      block { timestamp { time } }
      transaction { signature }
      sender { address }
      receiver { address }
      amount
      amount_usd: amount(in: USD)
      currency { symbol address }
    }
  }
}
```

### Transfers for a specific token (mint)

Track every transfer of a specific SPL token — useful for memecoin analytics, holder distribution, and circulating-supply checks.

```graphql
{
  solana(network: solana) {
    transfers(
      options: {limit: 100, desc: "block.timestamp.unixtime"}
      currency: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}  # USDC mint
    ) {
      block { timestamp { time } }
      transaction { signature }
      sender { address }
      receiver { address }
      amount
      currency { symbol address }
    }
  }
}
```

### Aggregate volume over time

Bucket SPL token transfer volume into daily intervals — perfect for dashboards and analytics.

```graphql
{
  solana(network: solana) {
    transfers(
      options: {asc: "date.date"}
      date: {since: "2024-01-01"}
      currency: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}
    ) {
      date { date(format: "%Y-%m-%d") }
      count
      transferred: amount
      transferred_usd: amount(in: USD)
      unique_senders: count(uniq: senders)
      unique_receivers: count(uniq: receivers)
    }
  }
}
```

### Top senders / receivers

Rank wallets by total tokens transferred — great for whale-leaderboard widgets.

```graphql
{
  solana(network: solana) {
    transfers(
      options: {limit: 50, desc: "amount"}
      currency: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}
      date: {since: "2024-01-01"}
    ) {
      sender { address }
      total_sent: amount
      txn_count: count
    }
  }
}
```

### Inbound vs outbound for an address

Compute net flow for a wallet — the foundation of any portfolio tracker or accounting tool.

```graphql
{
  solana(network: solana) {
    inflow: transfers(
      receiverAddress: {is: "9nnLbotNTcUhvbrsA6Mdkx45Sm82G35zo28AqUvjExn8"}
      currency: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}
    ) {
      total_in: amount
      count
    }
    outflow: transfers(
      senderAddress: {is: "9nnLbotNTcUhvbrsA6Mdkx45Sm82G35zo28AqUvjExn8"}
      currency: {is: "EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v"}
    ) {
      total_out: amount
      count
    }
  }
}
```

### Transfers for a transaction signature

Decompose a single Solana transaction into its underlying transfers — useful for explorers and forensic tooling.

```graphql
{
  solana(network: solana) {
    transfers(
      txHash: {is: "3x3fbg3zfTvcfwEiDee5Z5NnQP2Hr7cgZZ9bMxNYtYKi6fMN9gT6xpdUzRb2FjfCkGXMPvhkt3bW61CHCNaWwdQi"}
    ) {
      sender { address }
      receiver { address }
      amount
      currency { symbol address }
    }
  }
}
```

---

## Migration Path: V1 → V2

V1 is great for historical batch queries but does not support real-time WebSocket subscriptions, modern DEX cubes (Pump.fun, Raydium, Orca), or Solana Kafka/gRPC streams. For those, migrate to **V2**.

| Feature | V1 | V2 |
|---|---|---|
| Endpoint | `graphql.bitquery.io` | `streaming.bitquery.io` |
| Schema | `solana(network: solana) { transfers(...) }` | `Solana { Transfers(...) }` |
| Filters | `options`, `date`, lower-case args | `where`, `orderBy`, `limit` (PascalCase) |
| Real-time WebSocket | ❌ | ✅ via `subscription` |
| Pump.fun / Raydium | ❌ | ✅ |
| Kafka topics | ❌ | ✅ `solana.tokens.proto` |
| gRPC (CoreCast) | ❌ | ✅ |

[V2 Solana Transfers Docs ➜](https://docs.bitquery.io/docs/blockchain/Solana/solana-transfers/)

---

## Tools, IDEs & SDKs

- 🛠️ **[Bitquery GraphQL IDE](https://ide.bitquery.io/)** — Interactive query editor with auto-complete, schema explorer, and sharing.
- 🌐 **[Bitquery Solana Explorer](https://explorer.bitquery.io/solana)** — Browse blocks, transactions, transfers, DEX trades. Click "Get API" on any widget to copy the underlying GraphQL.
- 🐙 **[bitquery/graphql-ide on GitHub](https://github.com/bitquery/graphql-ide)** — Self-host the IDE.
- 📦 **[bitquery-crypto-price (npm)](https://github.com/bitquery/crypto-price-api)** — Convenience SDK with helpers for prices and aggregations.
- 📓 **[Code samples (Python, JS, Go)](https://github.com/bitquery/grpc-code-samples)** — Streaming and batch examples.

---

## Use Cases

- **Wallet analytics & portfolio trackers** — Inbound/outbound, net flow, P&L by token.
- **Tax reporting** — Reconstruct full transfer history for any wallet across any date range.
- **AML & compliance investigations** — Trace stolen funds, sanctioned-address screening, suspicious-activity reporting.
- **Memecoin holder analytics** — Top holders, distribution heatmaps, top wallets by inflow.
- **Stablecoin payment monitoring** — Detect on-chain settlements of USDC, USDT, EURC, PYUSD.
- **Treasury reconciliation** — Match on-chain transfers to off-chain accounting.

---

## Pricing

Bitquery offers a free Developer plan plus paid tiers based on points (per-query cost). See [bitquery.io/pricing](https://bitquery.io/pricing).

---

## Community & Support

- 📚 **V1 Docs:** [docs.bitquery.io/v1](https://docs.bitquery.io/v1/)
- 📚 **V2 Docs:** [docs.bitquery.io](https://docs.bitquery.io/)
- 💬 **Telegram:** [t.me/Bloxy_info](https://t.me/Bloxy_info)
- 💬 **Forum:** [community.bitquery.io](https://community.bitquery.io/)
- 🐦 **Twitter:** [@Bitquery_io](https://twitter.com/Bitquery_io)
- ✉️ **Email:** support@bitquery.io
- 🐙 **GitHub:** [github.com/bitquery](https://github.com/bitquery)

---

## License

[CC0](https://creativecommons.org/publicdomain/zero/1.0/) — Free for any use. Contributions welcome via PR.
