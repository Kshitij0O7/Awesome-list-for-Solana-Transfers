# Awesome Solana Transfers API

A list of resources for querying **Solana SPL token & SOL transfers** using the **Bitquery V1 GraphQL API**.

I ended up using Bitquery V1 for a side project (a wallet PnL tracker) and spent a while piecing this together from docs, the IDE, and Telegram threads. Sharing the queries that ended up being useful in case it saves someone else the time.

> **Heads up:** V1 is the legacy endpoint. It's still solid for historical/aggregated transfer queries, but if you need WebSocket subscriptions, Pump.fun/Raydium decoded data, or gRPC streams, you'll want V2 instead. See the migration table below.

---

## Contents

- [Endpoint & auth](#endpoint--auth)
- [Queries](#queries)
  - [Latest transfers](#latest-transfers)
  - [Transfers for a wallet](#transfers-for-a-wallet)
  - [Transfers for a specific Token](#transfers-for-a-specific-token)
  - [Aggregate volume over time](#aggregate-volume-over-time)
  - [Top senders / receivers](#top-senders--receivers)
  - [Inbound vs outbound](#inbound-vs-outbound)
  - [Transfers for a transaction signature](#transfers-for-a-transaction-signature)
- [V1 → V2 cheatsheet](#v1--v2-cheatsheet)
- [Tools](#tools)
- [Links](#links)

---

## Endpoint & auth

V1 endpoint: `https://graphql.bitquery.io`

You need an access token. Sign up at [account.bitquery.io](https://account.bitquery.io/auth/signup?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana) and generate one at [account.bitquery.io/user/api_v2/access_tokens](https://account.bitquery.io/user/api_v2/access_tokens?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana). Pass it as `Authorization: Bearer <token>`.

```bash
curl -X POST https://graphql.bitquery.io \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"query":"{ solana(network: solana) { transfers(options: {limit: 5, desc: \"block.timestamp.unixtime\"}) { transaction { signature } amount currency { symbol address } sender { address } receiver { address } } } }"}'
```

Auth walkthrough: [docs.bitquery.io/docs/authorisation/how-to-generate](https://docs.bitquery.io/docs/authorisation/how-to-generate/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana)

---

## Queries

All of these run as-is in the [Bitquery IDE](https://ide.bitquery.io/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Latest transfers

[Most recent SOL + SPL transfers across the chain](https://ide.bitquery.io/latest-Solana-transfers/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Transfers for a wallet

[Both sent and received transfers for a wallet within a date range](https://ide.bitquery.io/transfers-by-a-wallet/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Transfers for a specific Token

[Filter by SPL mint address. USDC mint shown](https://ide.bitquery.io/transfers-for-a-token/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Aggregate volume over time

[Daily buckets such as transfers count, total volume, USD volume, unique senders/receivers per day](https://ide.bitquery.io/daily-transfer-stats/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Top senders / receivers

[Rank wallets by total amount sent for a given mint](https://ide.bitquery.io/highest-senders/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Inbound vs outbound

[Two named sub-queries in one request to get inflow and outflow for a currency for a wallet, which can useful for net-flow / PnL math](https://ide.bitquery.io/currency-outflow-and-inflow-for-a-wallet/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

### Transfers for a transaction signature

[Decompose a signature into its underlying transfers](https://ide.bitquery.io/transfers-for-a-transaction_1/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana).

---

## V1 → V2 cheatsheet

Came up while migrating part of my project. Notes:

| | V1 | V2 |
|---|---|---|
| Endpoint | `graphql.bitquery.io` | `streaming.bitquery.io/graphql` |
| Schema | `solana(network: solana) { transfers(...) }` | `Solana { Transfers(...) }` |
| Filter args | `options`, `date`, lowercase | `where`, `orderBy`, `limit` (PascalCase) |
| Subscriptions | no | yes (WebSocket) |
| Pump.fun / Raydium cubes | no | yes |
| Kafka topics | no | yes (`solana.tokens.proto` etc.) |
| gRPC (CoreCast) | no | yes |

V2 transfers reference: [docs.bitquery.io/docs/blockchain/Solana/solana-transfers](https://docs.bitquery.io/docs/blockchain/Solana/solana-transfers/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana)

---

## Tools

- [Bitquery IDE](https://ide.bitquery.io/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana) — Where I write/test queries. Schema explorer is on the right side.
- [Solana explorer](https://explorer.bitquery.io/solana?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana) — "Get API" button on each widget gives you the underlying GraphQL, which is genuinely useful for learning the schema.

---

## Links

- V1 docs: [docs.bitquery.io/v1](https://docs.bitquery.io/v1/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana)
- V2 docs: [docs.bitquery.io](https://docs.bitquery.io/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana)
- Telegram (support): [t.me/Bloxy_info](https://t.me/Bloxy_info)
- Forum: [community.bitquery.io](https://community.bitquery.io/?utm_source=github&utm_medium=readme&utm_campaign=awesomelistsolana)
- GitHub: [github.com/bitquery](https://github.com/bitquery)

PRs welcome if you have queries that should be here.

---

## License

[CC0](https://creativecommons.org/publicdomain/zero/1.0/) — public domain. Use anything here however you want.
