---
sidebar_position: 1
---

# Get Started with Outserv

Let's get started with **Outserv in less than 5 minutes.**

## 0. Prerequisites

:::tip Test Setup (July 2022)

- Go version 1.17.6
- Node 16.14.2
- Ubuntu 22.04 LTS

:::

We recommend you use a Linux system to run Outserv. To run lambda functions, we
recommend installing nodejs as well.

## 1. Install Outserv

```bash
sudo apt-get update
sudo apt-get install gcc make golang nodejs

git clone https://github.com/outcaste-io/outserv.git
cd ./outserv
git checkout release/v22.07
make install
```

## 2. Run Outserv GraphQL

We will run Outserv with Lambdas. So, ensure that you have NodeJS installed.

```bash
outserv graphql --lambda="num=2;" # Data would go into "data" dir.
```

:::tip Lambda Errors

If you get lambda errors, most likely the JS files in lambda directory are
incompatible with your nodejs installation (they were built with 16.14.2). You
can regenerate them like so, and re-run outserv.

```
cd outserv/lambda
make build
```

:::

## 3. Load up GraphQL Schema and Lambdas

You can pick up a GraphQL schema and Lambda code from
[`importers/eth`](https://github.com/outcaste-io/outserv/blob/main/importers/eth),
or use your own schema.

```bash
cd outserv/importers/eth
curl -X POST http://localhost:8080/admin/schema --data-binary '@schema.graphql'
curl -X POST http://localhost:8080/admin/lambda --data-binary '@lambda.js'
```

## 4. Run Ethereum loader

You can either run geth locally, or you can use a geth gateway provided by
Cloudflare, Alchemy, Infura or others. I recommend using GraphQL endpoint
because that's [much
faster](https://twitter.com/manishrjain/status/1546675483986710529) than
JSON-RPC.

```bash
cd outserv/importers/eth
go build . && ./eth --geth "GraphQL or JSON-RPC endpoint of Geth"
```

This would start loading ETH transactions into Outserv.


## 5. You're ready

You can now use any GraphQL editor (like Insomnia) to query Outserv at:
`http://localhost:8080/graphql`. The editor would show the API documentation
automatically.

