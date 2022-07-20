---
sidebar_position: 8
---

# 8. Migration

In every app's development lifecycle, there's a point where the underlying
schema doesn't fit the requirements and must be changed for good. That requires
a migration for both schema and the underlying data. This article will guide you
through common migration scenarios you can encounter with Outserv and help you
avoid any pitfalls around them.

## Adding or removing Types and Edges

You can add or remove types, or fields within a type, by just resubmitting the
schema to Outserv. You can add [references](edges) to other types in the same
way.

## Adding or removing search indices

You can add or remove [search](search) indices from any field in a Type,
resubmit the schema to Outserv. It would automatically generate or drop the
index on-the-fly, without any downtime.

## Changing a field's type

Most scalars can be changed in a live Outserv instance. For example, a `String`
can be switched to an `Int`. Or, an `Int` to a `Float` or a `DateTime`. Their
search indices, if present, would also be automatically re-generated on-the-fly.
For example, `fee: Int64` can be switched to `fee: Float`, `fee: String` or to
`Fee: BigInt` without any downtime.

Converting a single item to a list can also be achieved live, without any
downtime. However, converting a list to a single-item isn't currently possible,
due to fears of data loss. For example, `incoming: [Txn]` to `incoming: Txn`
isn't possible. While `incoming: Txn` to `incoming: [Txn]` is.

## Renaming a Type or a field

Renaming a Type or a field would require you to reload the data for the type or
the field. There's currently no easy way to move the data over.

:::note
We plan to add a way to allow quick migrations in the future.
:::

## Adding `@id` to an existing field

Let's say you had the following schema:

```graphql
type User {
    id: ID!
    username: String
}
```

and now you think that `username` must be unique for every user. So, you change the schema to this:

```graphql
type User {
    id: ID!
    username: String! @id
}
```

Outserv would now start enforcing the uniqueness constraint over `username`.
However, here's a catch: With the old schema, multiple users could have had the
username `Alice`. The queries involving `Alice` would now break.

```graphql
query {
    getUser(username: "Alice") {
        id
    }
}

# Might error out with:
# highlight-next-line
# A list was returned, but GraphQL was expecting just one item.
```

:::caution
When making such a schema change, you must ensure that the underlying data
respects the `username` field's uniqueness constraint. If not, you must reload
the data to comply with the constraint.
:::
