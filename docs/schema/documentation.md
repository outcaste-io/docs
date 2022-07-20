---
sidebar_position: 6
---

# 6. Documentation

Outserv accepts GraphQL documentation comments (e.g. `""" This is a graphql comment """`), which get passed through to the generated API and thus shown as documentation in GraphQL tools like GraphiQL, GraphQL Playground, Insomnia etc.

You can also add `# ...` comments where ever you like.  These comments are not passed via the generated API and are not visible in the API docs.

An example that adds comments to a type as well as fields within the type would be as below.

```graphql
"""
Author of questions and answers in a website
"""
type Author {
# ... username is the author name , this is an example of a dropped comment
  username: String! @id
"""
The questions submitted by this author
"""
  questions: [Question] @hasInverse(field: author)
"""
The answers submitted by this author
"""
  answers: [Answer] @hasInverse(field: author)
}
```

