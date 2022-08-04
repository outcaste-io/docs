---
sidebar_position: 3
---

# 3. Ethereum APIs

Ethereum APIs listed here are the JSON-RPC APIs as [defined
here](https://ethereum.org/en/developers/docs/apis/json-rpc/).

Outserv exposes GraphQL APIs corresponding to each of these JSON-RPC APIs.

### eth_blockNumber ✓
*Returns the balance of the account of given address.*

```graphql
{
   eth_latestBlock {
      number
   }
}
```

### eth_getStorageAt
*Returns the value from a storage position at a given address.*

:::caution WIP
:::

### eth_getTransactionCount ✓
*Returns the number of transactions sent from an address.*

While the official spec only asks for outgoing transactions from an address,
Outserv tracks both incoming and outgoing transactions, and given any account,
you can query both.

```graphql
{
  getAccount(...) { # Query One Account
   outgoingAggregate { # Aggregate of Outgoing Txns
      count
   }
   incomingAggregate { # Aggregate of Incoming Txns
      count
   }
  }

  queryAccount(...) { # Query Multiple Accounts
   outgoingAggregate { # Aggregate of Outgoing Txns
      count
   }
   incomingAggregate { # Aggregate of Incoming Txns
      count
   }
  }
}
```
### eth_getBlockTransactionCountBy ✓

#### eth_getBlockTransactionCountByHash
*Returns the number of transactions in a block from a block matching the given block hash.*


#### eth_getBlockTransactionCountByNumber
*Returns the number of transactions in a block from a block matching the given block hash.*

```graphql
{
  getBlock(...) {
    transactionsAggregate {
      count
    }
  }
}
```

### eth_getUncleCountByBlock

#### eth_getUncleCountByBlockHash
*Returns the number of uncles in a block from a block matching the given block hash.*

#### eth_getUncleCountByBlockNumber
*Returns the number of uncles in a block from a block matching the given block number.*

:::caution WIP
:::

### eth_getCode
*Returns code at a given address.*

:::caution WIP
:::

### eth_sign
*The sign method calculates an Ethereum specific signature with: sign(keccak256("\x19Ethereum Signed Message:\n" + len(message) + message))).*

:::caution WIP
:::

### eth_signTransaction
*Signs a transaction that can be submitted to the network at a later time using with eth_sendRawTransaction.*

:::caution WIP
:::

### eth_sendTransaction
*Creates new message call transaction or a contract creation, if the data field contains code.*

:::caution WIP
:::

### eth_sendRawTransaction
*Creates new message call transaction or a contract creation for signed transactions.*

:::caution WIP
:::

### eth_call
*Executes a new message call immediately without creating a transaction on the block chain.*

:::caution WIP
:::

### eth_estimateGas
*Generates and returns an estimate of how much gas is necessary to allow the transaction to complete. The transaction will not be added to the blockchain. Note that the estimate may be significantly more than the amount of gas actually used by the transaction, for a variety of reasons including EVM mechanics and node performance.*

:::caution WIP
:::

### eth_getBlockBy ✓
#### eth_getBlockByHash
*Returns information about a block by hash.*
#### eth_getBlockByNumber
*Returns information about a block by block number.*

```graphql
{
  getBlock(...) { # Query One Block
    number
    # any fields you need
  }
  queryBlock(...) { # Query Multiple Blocks
    number
    # any fields you need
  }
}
```

### eth_getTransactionBy ✓
#### eth_getTransactionByHash
*Returns the information about a transaction requested by transaction hash.*
#### eth_getTransactionByBlockHashAndIndex
*Returns information about a transaction by block hash and transaction index position.*
#### eth_getTransactionByBlockNumberAndIndex
*Returns information about a transaction by block number and transaction index position.*

```graphql
{
  getTxn(...) { # Query One Txn
    hash
    # any fields you need
  }
  queryTxn(...) { # Query Multiple Txn
    hash
    # any fields you need
  }
}
```

### eth_getTransactionReceipt
*Returns the receipt of a transaction by transaction hash.*

:::caution WIP
:::

### eth_getUncleByBlock
#### eth_getUncleByBlockHashAndIndex
*Returns information about a uncle of a block by hash and uncle index position.*
#### eth_getUncleByBlockNumberAndIndex
*Returns information about a uncle of a block by number and uncle index position.*

:::caution WIP
:::

### eth_getCompilers
*Returns a list of available compilers in the client.*

:::caution WIP
:::

### eth_compile
#### eth_compileSolidity
*Returns compiled solidity code.*
#### eth_compileLLL
*Returns compiled LLL code.*
#### eth_compileSerpent
*Returns compiled serpent code.*

:::caution WIP
:::

### eth_getFilterChanges
*Polling method for a filter, which returns an array of logs which occurred since last poll.*

#### eth_newFilter
*Creates a filter object, based on filter options, to notify when the state changes (logs). To check if the state has changed, call eth_getFilterChanges.*

#### eth_newBlockFilter
*Creates a filter in the node, to notify when a new block arrives. To check if the state has changed, call eth_getFilterChanges.*

#### eth_newPendingTransactionFilter
*Creates a filter in the node, to notify when new pending transactions arrive. To check if the state has changed, call eth_getFilterChanges.*

#### eth_uninstallFilter
*Uninstalls a filter with given id. Should always be called when watch is no longer needed. Additionally Filters timeout when they aren't requested with eth_getFilterChanges for a period of time.*

#### eth_getFilterLogs
*Returns an array of all logs matching filter with given id.*

#### eth_getLogs
*Returns an array of all logs matching a given filter object.*

:::caution WIP
:::

### Others
#### eth_getWork
*Returns the hash of the current block, the seedHash, and the boundary condition to be met ("target").*

#### eth_submitWork
*Used for submitting a proof-of-work solution.*

#### eth_submitHashrate
*Used for submitting mining hashrate.*
