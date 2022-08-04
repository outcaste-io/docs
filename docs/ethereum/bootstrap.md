---
sidebar_position: 2
---

# 2. Import 15M Ethereum Blocks

In this tutorial, we will import Ethereum transactions, their corresponding
accounts and blocks. The relevant code is located in `outserv/importers/eth`.

The 15M ETH blocks produce quite a lot of data. To import them, we'd use the
`outserv boot` tool, aka the `boot loader`. This would allow us to *bootstrap*
our Outserv instance in the fastest possible time.

The `boot loader` takes in a GraphQL schema, and the input gzipped JSON files.
It can also input via interprocess communication. We'd use the latter.

## Run Eth importer

:::tip
To run this, you must have access to an Ethereum Geth node's [GraphQL
endpoint](geth). The GraphQL endpoint is 10x faster than the JSON-RPC endpoint.
:::
```bash
git clone https://github.com/outcaste-io/outserv.git
cd ./outserv/importers/eth

# If you don't have Go installed, run:
# sudo apt-get install golang
go build .

rm -rf ipc; mkdir ipc  # Ensure the directory is clean and exists.

./eth --help # Familiarize yourself with the options.
./eth --geth "GraphQL endpoint of Geth" --ipc ipc --start=1 --gor=8
```

This would run the Ethereum importer tool, which would
- `--geth=<GraphQL endpoint>` connect with the Geth's GraphQL endpoint,
- `--gor=8` run 8 goroutines to query transactions,
- `--ipc=ipc` write those on 8 IPC files created in the `ipc` directory,
- `--start=1` starting from block 1

## Run Boot Loader

```bash
outserv boot --ipc importers/eth/ipc --schema importers/eth/schema.graphql --out outboot
```

This would let the boot loader read data directly from the IPC files. The boot
loader runs in two phases: the map phase and the reduce phase. The map phase
would output to the `--tmp` directory, so ensure that you have enough disk
space. The reduce phase would output to the `--out` directory.

:::note
Based on my testing, this should take around 12 hours to index the entire
Ethereum blockchain.
:::

Once the process ends, it would have generated a `p` directory in `outboot/0`.
You can copy or move the `p` directory under some `data` dir.

## Run Outserv

:::tip
Ensure you have NodeJS installed. See [Intro](/docs/intro).
:::
```bash
outserv graphql --data="dir=<data dir from above>" --lambda="num=2;"

cd importers/eth
curl -X POST http://localhost:8080/admin/schema --data-binary '@schema.graphql'
curl -X POST http://localhost:8080/admin/lambda --data-binary '@lambda.js'
```

You can now query Outserv using any GraphQL editor (like Insomnia) at
`http://localhost:8080/graphql`.
