---
sidebar_position: 1
---

# 1. Ethereum GraphQL Endpoint

The best way to get an Ethereum GraphQL endpoint is to just run a `geth`
instance. You'd need 1TB of disk space.

:::tip
As of July 2022, Geth takes 630GB of disk. So, a TB is a safe choice.
:::

## Run Geth Like a Leet

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

This should take around 8 hours to sync up from block 1. Best is to run this
overnight.

Geth would expose a GraphQL endpoint at `http://localhost:8545/graphql`. This
endpoint can be fed to the importers.
