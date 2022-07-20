---
sidebar_position: 10
---

# Reserved Names

The following names are reserved and can't be used to define any other identifiers:

- `Int`
- `Float`
- `Boolean`
- `String`
- `DateTime`
- `ID`
- `uid`
- `Subscription`
- `as` (case-insensitive)
- `Query`
- `Mutation`
- `Point`
- `PointList`
- `Polygon`
- `MultiPolygon`
- `Aggregate` (as a suffix of any identifier name)


For each type, Outserv generates a number of GraphQL types needed to operate the
GraphQL API, these generated type names also can't be present in the input
schema.  For example, for a type `Txn`, Dgraph generates:

- `TxnFilter`
- `TxnOrderable`
- `TxnOrder`
- `TxnRef`
- `AddTxnInput`
- `UpdateTxnInput`
- `TxnPatch`
- `AddTxnPayload`
- `DeleteTxnPayload`
- `UpdateTxnPayload`
- `TxnAggregateResult`

**Mutations**

- `addTxn`
- `updateTxn`
- `deleteTxn`

**Queries**

- `getTxn`
- `queryTxn`
- `aggregateTxn`

Thus if `Txn` is present in the input schema, all of those become reserved type names.
