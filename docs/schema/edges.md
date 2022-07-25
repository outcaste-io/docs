---
sidebar_position: 4
---

# 4. References and Edges

All the data in your app forms a GraphQL data graph.  That graph has nodes of
particular types (the types you define in your schema) and links between the
nodes to form the data graph.

Outserv uses the types and fields in the schema to work out how to link that
graph, what to accept for mutations and what shape responses should take.

Edges in that graph are directed: either pointing in one direction or two.

### Unidirectional Edges

If you only ever need to traverse the graph between nodes in a particular
direction, then your schema can simply contain the types and the link.

In this schema, blocks have transactions, but there's no reverse edge from Txn
to Block. So, you can traverse from Block to Txn, but not from Txn to Block.

```graphql
type Txn {
    ...
}

type Block {
    ...
    transactions: [Txn]
}
```

### Bidirectional Edges

To have two types with edges to each other, you can use the `@hasInverse`
directive to tell Outserv to automatically connect them. With this, a Txn which
has specifies a Block, would automatically create an edge from Block to the Txn
as well.

```graphql
type Txn {
  oid: ID!
  hash: String! @id
  block: Block @hasInverse(field: transactions) # field in Block type.
  to: Account @hasInverse(field: incoming)      # field in Account type.
  from: Account @hasInverse(field: outgoing)    # field in Account type.
}

type Account {
  oid: ID!
  address: String! @id
  incoming: [Txn]
  outgoing: [Txn]
}

type Block {
  number: Int64 @id
  timestamp: Int64 @search
  transactions: [Txn]
}
```

Consider this output from eth importer. Here we're specifying that the Txn
belongs to a Block number `14900001`. This automatically creates an inverse edge
from Block to Txn as well, due to `@hasInverse` directive.

```json
{
  "hash": "0x08009d0e3abb12de45149150cdb9dd3620bdb810e43eff0c3eb3692d97990d38",
  "value": 1570000000,
  "fee": 19787878,
  "blockNumber": 14900001,
  "timestamp": 1654299399,
  "block": {
    "number": 14900001
  },
  "to": {
    "address": "0x881d40237659c251811cec9c364ef91dc08d300c"
  },
  "from": {
    "address": "0xc7e3ab2d82c2a2ca76a57c19420c6f5a315d36bb"
  }
}
```

Similarly, the Txn has `to` and `from` addresses specified. In this case, the
`to` account would add the Txn to `incoming` field, and the `from` account would
add the Txn to `outgoing` field, as specified in the schema.

This works the other way round as well. If the Account object adds an `incoming`
Txn, that would show up in the `to` field of the Txn object.

Outserv ensures that add, updates and deletes all work as expected to guarantee
[referential integrity](https://en.wikipedia.org/wiki/Referential_integrity).

:::danger
If Outserv is already running, and you add the `@hasInverse` directive, it
wouldn't automatically create the reverse edges yet. Outserv only does so at the
time of mutation. You'd have to reload the data.
:::

