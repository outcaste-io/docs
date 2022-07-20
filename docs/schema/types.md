---
sidebar_position: 3
---

# Types

This page describes how you can use GraphQL types to set up a GraphQL schema for
Outserv database.

## Scalars

Outserv's GraphQL implementation comes with these scalar types:

- `Boolean`
- `DateTime` (represented as string in RFC3339 format)
- `Float`
- `ID`
- `Int64`
- `Int`
- `String`
- `BigInt`

The `Int64` type represents a signed integer ranging between `-(2^63)` and
`(2^63 -1)`. Signed `Int64` values in this range will be parsed correctly by
Outserv as long as the client can serialize the number correctly in JSON. For
example, a JavaScript client might need to use a serialization library such as
[`json-bigint`](https://www.npmjs.com/package/json-bigint) to correctly write an
`Int64` value in JSON.

## Enums

You can define enums in your input schema.  For example:

```graphql
enum Status {
    OK
    FAILED
}

type Txn {
  oid: ID!
  hash: String! @id
  status: Status
}
```

## Types

From the built-in scalars and the enums you add, you can generate types in the
usual way for GraphQL.  For example:

```graphql
type Txn {
  uid: ID!
  hash: String! @id
  value: Int64 @search
  fee: Int64 @search
}

type Block {
  number: Int64 @id
  timestamp: Int64 @search
  transactions: [Txn]
}
```


## Lists

You can define a list with Scalars, or with other types.

Lists of lists aren't accepted.  For example: `multiTags: [[Tag!]]` isn't valid.

```graphql
type Account {
  oid: ID!
  address: String! @id
  notes: [String]
  incoming: [Txn]
  outgoing: [Txn]
}
```

:::danger
Scalar lists in Outserv act like unordered sets. So `tags: [String]` would
always contain unique tags. For example, `["e1", "e1", "e2"]` may get stored as
`["e2", "e1"]`
:::

## Interfaces

GraphQL interfaces allow you to define a generic pattern that multiple types
follow.  When a type implements an interface, that means it has all fields of
the interface and some extras.

According to GraphQL specifications, you can have the same fields in
implementing types as the interface. In such cases, the GraphQL layer will
generate the correct Outserv schema without duplicate fields.

If you repeat a field name in a type, it must be of the same type (including
list or scalar types), and it must have the same nullable condition as the
interface's field. Note that if the interface's field has a directive like
`@search` then it will be inherited by the implementing type's field.

For example:

```graphql
interface Fruit {
    id: ID!
    price: Int!
}

type Apple implements Fruit {
    id: ID!     # repeated with same properties
    price: Int! # repeated with same properties
    color: String!
}

type Banana implements Fruit {
    id: ID!     # repeated with same properties
    price: Int! # repeated with same properties
}
```

The following example defines the schema for posts with comment threads. As
mentioned, Outserv will fill in the `Question` and `Comment` types to make the
full GraphQL types.

```graphql
interface Post {
    id: ID!
    text: String
    datePublished: DateTime
}

type Question implements Post {
    title: String!
}
type Comment implements Post {
    commentsOn: Post!
}

# Generated Schema below

type Question implements Post {
    id: ID!                 # derived from interface
    text: String            # derived from interface
    datePublished: DateTime # derived from interface
    title: String!          # derived from type
}

type Comment implements Post {
    id: ID!                 # derived from interface
    text: String            # derived from interface
    datePublished: DateTime # derived from interface
    commentsOn: Post!       # derived from type
}
```

:::danger
If you have a type that implements two interfaces, Outserv won't allow a field of the same name in both interfaces, except for the `ID` field.
:::


## Union type

GraphQL Unions represent an object that could be one of a list of GraphQL Object
types, but provides for no guaranteed fields between those types. So no fields
may be queried on this type without the use of type refining fragments or inline
fragments.

Union types have the potential to be invalid if incorrectly defined:

- A `Union` type must include one or more unique member types.
- The member types of a `Union` type must all be Object base types; [Scalar](#scalars), [Interface](#interfaces) and `Union` types must not be member types of a Union. Similarly, wrapping types must not be member types of a Union.


For example, the following defines the `HomeMember` union type:

```graphql
enum Category {
  Fish
  Amphibian
  Reptile
  Bird
  Mammal
  InVertebrate
}

interface Animal {
  id: ID!
  category: Category @search
}

type Dog implements Animal {
  breed: String @search
}

type Parrot implements Animal {
  repeatsWords: [String]
}

type Cheetah implements Animal {
  speed: Float
}

type Human {
  name: String!
  pets: [Animal!]!
}

union HomeMember = Dog | Parrot | Human

type Zoo {
  id: ID!
  animals: [Animal]
  city: String
}

type Home {
  id: ID!
  address: String
  members: [HomeMember]
}
```

So, when you want to query members in a `Home`, you will be able to do a GraphQL query like this:

```graphql
query {
  queryHome {
    address
    members {
      ... on Animal {
        category
      }
      ... on Dog {
        breed
      }
      ... on Parrot {
        repeatsWords
      }
      ... on Human {
        name
      }
    }
  }
}

# And the results of the GraphQL query will look like the following:

{
  "data": {
    "queryHome": {
      "address": "Earth",
      "members": [
        {
          "category": "Mammal",
          "breed": "German Shepherd"
        }, {
          "category": "Bird",
          "repeatsWords": ["Good Morning!", "I am a GraphQL parrot"]
        }, {
          "name": "Alice"
        }
      ]
    }
  }
}
```

## Password type

A password for an entity is set with setting the schema for the node type with `@secret` directive. Passwords cannot be queried directly, only checked for a match using the `checkTypePassword` function where `Type` is the node type.
The passwords are encrypted using [bcrypt](https://en.wikipedia.org/wiki/Bcrypt).

:::tip
For security reasons, Outserv enforces a minimum password length of 6 characters on `@secret` fields.
:::

For example, to set a password, first set schema:

1. Cut-and-paste the following schema into a file called `schema.graphql`
    ```graphql
    type Author @secret(field: "pwd") {
      name: String! @id
    }
    ```

2. Run the following curl request:
    ```bash
    curl -X POST localhost:8080/admin/schema --data-binary '@schema.graphql'
    ```

3. Set the password by pointing to the `graphql` endpoint (http://localhost:8080/graphql):
    ```graphql
    mutation {
      addAuthor(input: [{name:"myname", pwd:"mypassword"}]) {
        author {
          name
        }
      }
    }
    ```

The output should look like:
```json
{
  "data": {
    "addAuthor": {
      "author": [
        {
          "name": "myname"
        }
      ]
    }
  }
}
```

You can check a password:
```graphql
query {
  checkAuthorPassword(name: "myname", pwd: "mypassword") {
   name
  }
}
```

output:
```json
{
  "data": {
    "checkAuthorPassword": {
      "name": "myname"
    }
  }
}
```

If the password is wrong you will get the following response:
```json
{
  "data": {
    "checkAuthorPassword": null
  }
}
```

## Geolocation types

Outserv GraphQL comes with built-in types to store Geolocation data. Currently, it supports `Point`, `Polygon` and `MultiPolygon`. These types are useful in scenarios like storing a location's GPS coordinates, representing a city on the map, etc.

For example:

```graphql
type Hotel {
  id: ID!
  name: String!
  location: Point
  area: Polygon
}
```

#### Point

```graphql
type Point {
	longitude: Float!
	latitude: Float!
}
```

#### PointList

```graphql
type PointList {
	points: [Point!]!
}
```

#### Polygon

```graphql
type Polygon {
	coordinates: [PointList!]!
}
```

#### MultiPolygon

```graphql
type MultiPolygon {
	polygons: [Polygon!]!
}
```
