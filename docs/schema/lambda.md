---
sidebar_position: 8
---

# 8. Lambda Functions

Outserv supports running custom Javascript code. It does so by spinning up
NodeJS child processes, which it can interact with over loopback interface. The
benefit of running JS code in this way is that the data never crosses over
network, hence allowing a much faster execution even for larger responses.

You can create expose Lambda functions via GraphQL, using the `@lambda`
directive. These functions can return either Scalars or Types. If you don't want
Outserv to auto-generate queries around the type, you can add a `@remote`
directive.

Complex queries which are not directly possible with the auto-generated GraphQL
functions can be run using custom JS, while exposing a simple function to the
end user. In the following example, we are going to check account balance at any
given block, by querying for the aggregate of incoming and outgoing transactions
up to that block.

:::tip
While developing, as you make modifications to the JS script, you can reload the
script as described in [overview](overview). For debugging, you can also do
`console.log`. These logs would be visible in Outserv logs.
:::

:::caution
If you try to upload a schema containing `@lambda` directives, it would fail
unless Outserv is run with lambdas, using the `--lambda` flag.
:::

```graphql title="importers/eth/schema.graphql"
type AccountBal @remote { # Outserv won't generate functions around AcccountBal.
  address: String
  value: Int64
}

type Query {
  accountBalance(address: String!, blockNumber: Int64): AccountBal @lambda
  test: String @lambda         # @lambda directive would invoke a lambda function.
  latestBlock: Int64 @lambda
}
```

```javascript title="importers/eth/lambda.js" showLineNumbers
async function test({args, graphql}) {
  return "Hello, World! This is a test."
}

async function latestBlock({args, graphql}) {
  // highlight-next-line
  // You can run a GraphQL query from here.
  const results = await graphql(`
  { queryBlock(order: {desc: number}, first: 1) {
    number
  }} `)
  if (results.data.queryBlock.length == 0) {
    return 0
  }
  return results.data.queryBlock[0].number;
}

async function accountBal({args, graphql}) {
  // highlight-next-line
  // args provides the arguments as specified by the function definition.
  const results = await graphql(`
  query q($address: String, $blockNumber: Int64) {
    queryAccount(filter: {address: {eq: $address}}) {
      incomingAggregate(filter: {blockNumber: {le: $blockNumber}}) { valueSum }
      outgoingAggregate(filter: {blockNumber: {le: $blockNumber}}) { valueSum }
    }
  }`, {"address": args.address, "blockNumber": args.blockNumber })

  if (results.data.queryAccount.length == 0) {
    return {"address": args.address, "value": 0}
  }
  acc = results.data.queryAccount[0]
  inc = acc.incomingAggregate.valueSum;
  out = acc.outgoingAggregate.valueSum;
  diff = inc - out
  return {"address": args.address, "value": diff}
}

// highlight-next-line
// This is the way to connect the GraphQL schema to the JS func.
self.addGraphQLResolvers({
  "Query.accountBalance": accountBal,
  "Query.test": test,
  "Query.latestBlock": latestBlock
})
```
