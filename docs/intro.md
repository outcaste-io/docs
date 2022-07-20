---
sidebar_position: 1
---

# Tutorial Intro

Let's discover **Outserv in less than 5 minutes.**

## 0. Prerequisites

:::tip Test Setup (July 2022)

- Go version 1.17.6
- Node 16.14.2
- Ubuntu 22.04 LTS

:::

We recommend you use a Linux system to run Outserv. To run lambda functions, we
recommend installing nodejs as well.

```bash
sudo apt-get install nodejs
```

## Option 1.a) Download executable

Download the Outserv binary from
[releases](https://github.com/outcaste-io/outserv/releases) and place it in
`/usr/bin` or `/usr/local/bin`.

## Option 1.b) Build Outserv Yourself

```bash
sudo apt-get update
sudo apt-get install gcc make golang

git clone https://github.com/outcaste-io/outserv.git
cd ./outserv
git checkout v22.07.0
make install
```

## 2. Run Outserv GraphQL

We will run Outserv with Lambdas. So, ensure that you have NodeJS installed.

```bash
outserv graphql --lambda="num=2;" # Data would go into "data" dir.
```

## 3. Load up GraphQL Schema and Lambdas

You can pick up a GraphQL schema and Lambda code from
[`importers/eth`](https://github.com/outcaste-io/outserv/blob/main/importers/eth),
or use your own schema.

```bash
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
cd importers/eth
go build . && ./eth --geth "GraphQL or JSON-RPC endpoint of Geth"
```

This would start loading ETH transactions into Outserv.


## 5. You're ready

You can now use any GraphQL editor (like Insomnia) to query Outserv at:
`http://localhost:8080/graphql`. The editor would show the API documentation
automatically.

