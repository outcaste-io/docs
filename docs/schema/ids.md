---
sidebar_position: 2
---

# 2. Unique Identifiers

Outserv provides two types of built-in identifiers: the `ID` scalar type and the `@id` directive.

* The `ID` scalar type is used to refer to internal Outserv Unique Identifiers,
    also known as UIDs.
* The `@id` directive is used for external identifiers, such as hashes.

```graphql
type Txn {
  oid: ID!
  hash: String! @id
  ...
}
```

Outserv generates queries and mutations so you can query, update, and delete
data in nodes, using the fields with `ID` and with the `@id` directive as
references.

## The `ID` type

In Outserv, every node has a unique 64-bit integer identifier that you can
expose in GraphQL using the `ID` type. An `ID` is auto-generated, immutable and
never reused. Each type can have at most one `ID` field.

Irrespective of whether an external id `@id` exists or not, `ID` is always
generated. That's how Outserv internally refers to the nodes.

Fields of type `ID` can be listed as nullable in a schema, but Outserv will
never return null. `ID` lists aren't allowed - e.g. `tags: [String]` is valid,
but `ids: [ID]` is not.

## The `@id` directive

The `@id` directive tells Outserv to keep that field's values unique and use
them as identifiers. The field with `@id` directive must hold unique values,
like transaction or address hashes.

Identities created with `@id` are reusable. If you delete an existing object, you can reuse the username.

It's possible to use the `@id` directive on more than one field in a type. For example, you can define a type like the following:

```graphql
type Book {
    name: String! @id
    isbn: String! @id
    genre: String!
    ...
}
```

You can then use multiple `@id` fields in arguments to `get` queries, and while
searching, these fields will be combined with the `AND` operator.

The following would yield a positive response only if both the `name` **and**
`isbn` match any node.

```graphql
query {
  getBook(name: "The Metamorphosis", isbn: "9871165072") {
    name
    genre
    ...
  }
}
```

Outserv enforces a uniqueness check on the `@id` field(s) when creating new
nodes. If there are multiple such fields, then uniqueness is enforced by
intersecting across all those field values (AND operator).

<!--
## Interfaces and `@id`

By default, if used in an interface, the `@id` directive will ensure field
uniqueness for each implementing type separately. This allows two different
types implementing the same interface to have the same value for the inherited
`@id` field.

There are scenarios where this behavior might not be desired, and you may want
to constrain the `@id` field to be unique across all the implementing types. In
that case, you can set the `interface` argument of the `@id` directive to
`true`, and Outserv will ensure that the field has unique values across all the
implementing types of an interface.

For example:

```graphql
interface Item {
  refID: Int! @id(interface: true) # if there is a Book with refID = 1, then there can't be a chair with that refID.
  itemID: Int! @id # If there is a Book with itemID = 1, there can still be a Chair with the same itemID.
}

type Book implements Item { ... }
type Chair implements Item { ... }
```

In the above example, `itemID` won't be present as an argument to the `getItem` query as it might return more than one `Item`.
-->
