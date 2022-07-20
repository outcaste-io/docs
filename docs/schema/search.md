---
sidebar_position: 5
---

# 5. Search Indices

The `@search` directive tells Outserv what search to build into your GraphQL API.

When a type contains an `@search` directive, Outserv constructs a search input
type and a query in the GraphQL `Query` type. For example, if the schema
contains

```graphql
type Txn {
    ...
}
```

then Outserv constructs a `queryTxn` GraphQL query for querying Txns.  The
`@search` directives in the `Txn` type control how Outserv builds indexes and
what kinds of search it builds into `queryTxn`.  If the type contains

```graphql
type Txn {
  ...
  timestamp: DateTime @search(by: [day])
}
```

then it's possible to filter Txns with a date-time search like:

```graphql
query {
    queryTxn(filter: { timestamp : { ge: "2022-06-15" }}) {
        ...
    }
}
```

Outserv can build search types with the ability to search between a range. For
example with the above Txn type with timestamp field, a query can find
Txns within a range

```graphql
query {
    queryTxn(filter: { timestamp : { between: { min: "2022-06-15", max: "2022-06-30" }}}) {
        ...
    }
}
```


If the type tells Outserv to build search capability based on a term (word) index for the `title` field

```graphql
type Post {
    ...
    title: String @search(by: [term])
}
```

then, the generated GraphQL API will allow search by terms in the title.

```graphql
query {
    queryPost(filter: { title: { anyofterms: "GraphQL" }}) {
        ...
    }
}
```

Outserv also builds search into the fields of each type, so searching is
available at deep levels in a query.  For example, if the schema contained these
types:

```graphql
type Post {
    ...
    title: String @search(by: [term])
}

type Author {
    name: String @search(by: [hash])
    posts: [Post]
}
```

then Outserv builds GraphQL search such that a query can, for example, find an
author by name (from the hash search on `name`) and return only their posts that
contain the term "GraphQL".

```graphql
queryAuthor(filter: { name: { eq: "Diggy" } } ) {
    posts(filter: { title: { anyofterms: "GraphQL" }}) {
        title
    }
}
```

Outserv can also build GraphQL search ability to find match a value from a list. For example with the above Author type with the name field, a query can return the Authors that match a list

```graphql
queryAuthor(filter: { name: { in: ["Diggy", "Jarvis"] } } ) {
    ...
}
```

There's different search possible for each type as explained below.

## Int, Float, DateTime

| argument | constructed filter |
|----------|----------------------|
| none | `lt`, `le`, `eq`, `in`, `between`, `ge`, and `gt` |

Search for fields of types `Int`, `Float` and `DateTime` is enabled by adding
`@search` to the field with no arguments.  For example, if a schema contains:

```graphql
type Txn {
  blockNumber: Int64 @search
}
```

## DateTime

| argument | constructed filters |
|----------|----------------------|
| `year`, `month`, `day`, or `hour` | `lt`, `le`, `eq`, `in`, `between`, `ge`, and `gt` |

As well as `@search` with no arguments, `DateTime` also allows specifying how
the search index should be built: by `year`, `month`, `day` or `hour`.
`@search` defaults to year, but once you understand your data and query
patterns, you might want to changes that like `@search(by: [day])`.

```graphql
type Txn {
  timestamp: DateTime @search(by: [day])
```

## Boolean

| argument | constructed filter |
|----------|----------------------|
| none | `true` and `false` |

Booleans can only be tested for true or false.  If `isPublished: Boolean @search` is in the schema, then the search allows

```graphql
type Txn {
  statusOk: Boolean @search
}

# Would allow filtering Txns with:

filter: { statusOk: true }
filter: { statusOk: false }
```

## String

Strings allow a wider variety of search options than other types.  For strings,
you have the following options as arguments to `@search`.

| argument | constructed searches |
|----------|----------------------|
| `hash` | `eq` and `in` |
| `exact` | `lt`, `le`, `eq`, `in`, `between`, `ge`, and `gt` (lexicographically) |
| `regexp` | `regexp` (regular expressions) |
| `term` | `allofterms` and `anyofterms` |
| `fulltext` | `alloftext` and `anyoftext` |

### Exact and Hash search

Exact and hash search has the standard lexicographic meaning.

```graphql
type Author {
    ...
    name: String! @search(by: [hash])
}

# Allows:
query {
    queryAuthor(filter: { name: { eq: "Diggy" } }) { ... }
}
```

And for exact search

```graphql
type Author {
    ...
    name: String! @search(by: [exact])
}

# Allows the following query to find names after "Diggy".

query {
    queryAuthor(filter: { name: { gt: "Diggy" } }) { ... }
}
```

### Regular expression search

Search by regular expression requires bracketing the expression with `/` and
`/`.  For example, query for "Diggy" and anyone else with "iggy" in their name:

```graphql
type Author {
    ...
    name: String! @search(by: [regexp])
}

# Allows:
query {
    queryAuthor(filter: { name: { regexp: "/.*iggy.*/" } }) { ... }
}
```

### Term and Fulltext search

`term` search is a term matching search, using non-alphanumeric separators to
break up the terms in a phrase. For example, "aaa-bbb" would match "aaa bbb".

`fulltext` search is Google-stye
text search with stop words, stemming. etc. In other words, `fulltext` search
would match even future or past participles of the word. In the following
example, "flying" would match with "fly", or "flew".

```graphql
type Post {
    title: String @search(by: [term])
    body: String @search(by: [fulltext])
    ...
}

# Allows:
query {
    # Match all titles with both 'GraphQL' and 'tutorial'.
    queryPost(filter: { title: { allofterms: "GraphQL tutorial" } } ) { ... }

    # Match all titles with either 'GraphQL' or 'tutorial'.
    queryPost(filter: { title: { anyofterms: "GraphQL tutorial" } } ) { ... }
}

query {
    # Match all body with both "flying" and "pig". But, also similar words like
    # "fly", or "flew".
    queryPost(filter: { body: { alloftext: "flying pig" } } ) { ... }

    # Match all body with either "flying" or "pig". But, also similar words like
    # "fly", or "flew".
    queryPost(filter: { body: { alloftext: "flying pig" } } ) { ... }
}
```


### Mix and Match

It's possible to add multiple string indexes to a field.  For example to search
for authors by `eq` and regular expressions, add both options to the type
definition as follows.

```graphql
type Author {
    ...
    name: String! @search(by: [hash, regexp])
}
```

:::tip hash and exact
`hash` and `exact` can't be used together on the same field.
:::


## Enums

| argument | constructed searches |
|----------|----------------------|
| none | `eq` and `in` |
| `hash` | `eq` and `in` |
| `exact` | `lt`, `le`, `eq`, `in`, `between`, `ge`, and `gt` (lexicographically) |
| `regexp` | `regexp` (regular expressions) |

Enums are serialized in Outserv as strings.  `@search` with no arguments is the same as `@search(by: [hash])` and provides `eq` and `in` searches.  Also available for enums are `exact` and `regexp`.  For hash and exact search on enums, the literal enum value, without quotes `"..."`, is used, for regexp, strings are required. For example:

```graphql
enum Status {
    TXN_OK
    TXN_FAILED
    UNKNOWN
}

type Txn {
  oid: ID!
  hash: String! @id
  status: Status @search
}

# Allows:
query {
    queryTxn(filter: { status: { eq: TXN_OK } } ) { ... }
}
```

While `@search(by: [exact, regexp]` would allow

```graphql
query {
    queryPost(filter: { status: { regexp: "/TXN_.*/" } } ) { ... }
}
```

## Geolocation

There are 3 Geolocation types: `Point`, `Polygon` and `MultiPolygon`. All of them are searchable.

The following table lists the generated filters for each type when you include `@search` on the corresponding field:

| type | constructed searches |
|----------|----------------------|
| `Point` | `near`, `within` |
| `Polygon` | `near`, `within`, `contains`, `intersects` |
| `MultiPolygon` | `near`, `within`, `contains`, `intersects` |

Take for example a `Hotel` type that has a `location` and an `area`:

```graphql
type Hotel {
  id: ID!
  name: String!
  location: Point @search
  area: Polygon @search
}
```

### Near

The `near` filter matches all entities where the location given by a field is within a distance `meters` from a coordinate.

```graphql
queryHotel(filter: {
    location: {
        near: {
            coordinate: {
                latitute: 37.771935,
                longitude: -122.469829
            },
            distance: 1000
        }
    }
}) {
  name
}
```

### Within

The `within` filter matches all entities where the location given by a field is within a defined `polygon`.

```graphql
queryHotel(filter: {
    location: {
        within: {
            polygon: {
                coordinates: [{
                    points: [{
                        latitude: 11.11,
                        longitude: 22.22
                    }, {
                        latitude: 15.15,
                        longitude: 16.16
                    }, {
                        latitude: 20.20,
                        longitude: 21.21
                    }, {
                        latitude: 11.11,
                        longitude: 22.22
                    }]
                }],
            }
        }
    }
}) {
  name
}
```

### Contains

The `contains` filter matches all entities where the `Polygon` or `MultiPolygon` field contains another given `point` or `polygon`.

:::tip
Only one `point` or `polygon` can be taken inside the `ContainsFilter` at a time.
:::

A `contains` example using `point`:

```graphql
queryHotel(filter: {
    area: {
        contains: {
            point: {
              latitude: 0.5,
              longitude: 2.5
            }
        }
    }
}) {
  name
}
```

A `contains` example using `polygon`:

```graphql
 queryHotel(filter: {
    area: {
        contains: {
            polygon: {
                coordinates: [{
                  points:[{
                    latitude: 37.771935,
                    longitude: -122.469829
                  }]
                }],
            }
        }
    }
}) {
  name
}
```

### Intersects

The `intersects` filter matches all entities where the `Polygon` or `MultiPolygon` field intersects another given `polygon` or `multiPolygon`.

:::tip
Only one `polygon` or `multiPolygon` can be given inside the `IntersectsFilter` at a time.
:::

```graphql
  queryHotel(filter: {
    area: {
      intersects: {
        multiPolygon: {
          polygons: [{
            coordinates: [{
              points: [{
                latitude: 11.11,
                longitude: 22.22
              }, {
                latitude: 15.15,
                longitude: 16.16
              }, {
                latitude: 20.20,
                longitude: 21.21
              }, {
                latitude: 11.11,
                longitude: 22.22
              }]
            }, {
              points: [{
                latitude: 11.18,
                longitude: 22.28
              }, {
                latitude: 15.18,
                longitude: 16.18
              }, {
                latitude: 20.28,
                longitude: 21.28
              }, {
                latitude: 11.18,
                longitude: 22.28
              }]
            }]
          }, {
            coordinates: [{
              points: [{
                latitude: 91.11,
                longitude: 92.22
              }, {
                latitude: 15.15,
                longitude: 16.16
              }, {
                latitude: 20.20,
                longitude: 21.21
              }, {
                latitude: 91.11,
                longitude: 92.22
              }]
            }, {
              points: [{
                latitude: 11.18,
                longitude: 22.28
              }, {
                latitude: 15.18,
                longitude: 16.18
              }, {
                latitude: 20.28,
                longitude: 21.28
              }, {
                latitude: 11.18,
                longitude: 22.28
              }]
            }]
          }]
        }
      }
    }
  }) {
    name
  }
```

## Union

Unions can be queried only as a field of a type. Union queries can't be ordered, but you can filter and paginate them.

:::note
Union queries do not support the `order` argument.
The results will be ordered by the `uid` of each node in ascending order.
:::

For example, the following schema will enable to query the `members` union field in the `Home` type with filters and pagination.

```graphql
union HomeMember = Dog | Parrot | Human

type Home {
  id: ID!
  address: String

  members(filter: HomeMemberFilter, first: Int, offset: Int): [HomeMember]
}

# Not specifying a field in the filter input will be considered as a null value for that field.
input HomeMemberFilter {
	# `homeMemberTypes` is used to specify which types to report back.
	homeMemberTypes: [HomeMemberType]

	# specifying a null value for this field means query all dogs
	dogFilter: DogFilter

	# specifying a null value for this field means query all parrots
	parrotFilter: ParrotFilter
	# note that there is no HumanFilter because the Human type wasn't filterable
}

enum HomeMemberType {
	dog
	parrot
	human
}

input DogFilter {
	id: [ID!]
	category: Category_hash
	breed: StringTermFilter
	and: DogFilter
	or: DogFilter
	not: DogFilter
}

input ParrotFilter {
	id: [ID!]
	category: Category_hash
	and: ParrotFilter
	or: ParrotFilter
	not: ParrotFilter
}
```

:::tip
Not specifying any filter at all or specifying any of the `null` values for a filter will query all members.
:::



The same example, but this time with filter and pagination arguments:

```graphql
query {
  queryHome {
    address
    members (
      filter: {
		homeMemberTypes: [dog, parrot] # means we don't want to query humans
		dogFilter: {
			# means in Dogs, we only want to query "German Shepherd" breed
			breed: { allofterms: "German Shepherd"}
		}
		# not specifying any filter for parrots means we want to query all parrots
      }
      first: 5
      offset: 10
    ) {
      ... on Animal {
        category
      }
      ... on Dog {
        breed
      }
      ... on Parrot {
        repeatsWords
      }
      ... on HomeMember {
        name
      }
    }
  }
}
```
