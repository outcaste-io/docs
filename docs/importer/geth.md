---
sidebar_position: 1
---

# 1. Ethereum GraphQL Endpoint

The best way to get an Ethereum GraphQL endpoint is to just run a `geth`
instance. You'd need 1TB of disk space.

Geth should take around 8 hours to sync up from block 1. If you're running it
for the first time, best is to run overnight.

:::tip
As of July 2022, Geth takes 630GB of disk. So, a TB is a safe choice.
:::

## Run Geth via Docker

You can run Geth via Docker, as [described
here](https://hub.docker.com/r/ethereum/client-go).

```
# Assuming /data has 1TB disk space.

docker run --name eth-docker -v /data/geth:/root/.ethereum -p 8545:8545 ethereum/client-go --http.addr 0.0.0.0 --http --graphql
```


## Build and Run Geth (like a Leet)

```bash
# You must have Go installed.

git clone https://github.com/ethereum/go-ethereum.git
cd go-ethereum

# Optionally, go to a release version. But, building from main has been OK.
# git checkout release/v1.10 # or, future versions

make geth
# This would leave a geth binary in `./build/bin/geth`

# Optionally, run the following geth command from screen to avoid it
# shutting down if you log out.
#
# screen -S geth

# Assuming your 1TB disk is mounted on `/data`
# --http would run an HTTP server.
# --graphql would expose a GraphQL endpoint.
./build/bin/geth --datadir /data/geth --http --graphql
```

:::tip
Geth would expose a GraphQL endpoint at `http://localhost:8545/graphql`. This
endpoint can be fed to the importers.
:::

## Note on Ethereum Services

:::note Alchemy, Cloudflare, Infura
As of July 2022, I couldn't see any of Alchemy, Cloudflare, or Infura support
the GraphQL interface of Geth. When using the JSON-RPC interface via Infura, I
got a speed of importing 60 blocks/minute, which is too slow for bootstrapping
the entire blockchain, but perhaps okay in the live mode. Moreover,
downloading the entire ETH blockchain would quickly get **very** expensive due
to charge per request.
:::
