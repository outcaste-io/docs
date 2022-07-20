---
sidebar_position: 1
---

# 1. Overview

This section describes all the things you can put in your input GraphQL schema
to Ouserv.

The process for serving GraphQL with Outserv is to upload your GraphQL schema to
Outserv. Outserv takes those definitions, generates queries and mutations, and
serves the generated GraphQL schema.

The input schema may contain interfaces, types and enums that follow the usual
GraphQL syntax and validation rules. It may also contain custom JS script,
called lamba, to execute javascript code.

:::tip
Point a GraphQL editor like [Insomnia](https://insomnia.rest/products/insomnia)
to Outserv's `/graphql` endpoint to see all the possible queries and mutations
exposed. The editor would also help autocomplete the queries for you, making
development much easier.
:::

## Uploading Schema to Outserv

You can write your schema in a file, say `schema.graphql`, and upload it to
Outserv like so:
```bash
curl -X POST http://localhost:8080/admin/schema --data-binary '@schema.graphql'
```

## Uploading Lambda code to Outserv

You can write your lambda code in a file, say `lambda.js`, and upload it to
Outserv like so:
```bash
curl -X POST http://localhost:8080/admin/lambda --data-binary '@lambda.js'
```

## Outserv's GraphQL endpoints

```bash
http://localhost:8080/graphql

# Outserv's admin interface is also exposed over GraphQL.
http://localhost:8080/admin
```
