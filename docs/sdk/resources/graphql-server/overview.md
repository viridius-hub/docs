---
sidebar_position: 10
description: Zero-config GraphQL server
---

# Overview

The data indexed by a squid into a Postgres database can be automatically presented with a GraphQL API service powered by the [OpenReader](https://github.com/subsquid/squid-sdk/tree/master/graphql/openreader) lib of the Squid SDK. The OpenReader GraphQL server takes [the schema file](/sdk/reference/schema-file) as an input and serves a GraphQL API supporting [OpenCRUD](https://www.opencrud.org/) queries for the entities defined in the schema.

To start the API server based on the `schema.graphql` run in the squid project root:
```bash
npx squid-graphql-server
```
or, for more options,
```bash
npx squid-graphql-server
```
The `squid-graphql-server` binary supports multiple optional flags to enable caching, subscriptions, DoS protection etc. Its features are covered in the next sections.

The API server listens at port defined by `GQL_PORT` (defaults to `4350`). The database connection is configured with the env variables `DB_NAME`, `DB_USER`, `DB_PASS`, `DB_HOST`, `DB_PORT`.

The GraphQL API is enabled by the `api:` service in the `deploy` section of [squid.yaml](/cloud/reference/manifest) for Subsquid Cloud deployments.

## Supported Queries

The details of the supported OpenReader queries can be found in a separate section [Query a Squid](/sdk/reference/openreader). Here is a brief overview of the queries generated by OpenReader for each entity defined in the schema file:

- the squid last processed block is available with `squidStatus { height }` query 
- a "get one by ID" query with the name `{entityName}ById` for each [entity](/sdk/reference/schema-file/entities) defined in the schema file
- a "get one" query for [`@unique` fields](/sdk/reference/schema-file/indexes-and-constraints), with the name `{entityName}ByUniqueInput`
- Entity queries named `{entityName}sConnection`. Each query supports rich filtering support, including [field-level filters](/sdk/reference/openreader/queries), composite [`AND` and `OR` filters](/sdk/reference/openreader/and-or-filters), [nested queries](/sdk/reference/openreader/nested-field-queries), [cross-relation queries](/sdk/reference/openreader/cross-relation-field-queries) and [Relay-compatible](https://relay.dev/graphql/connections.htm) cursor-based [pagination](/sdk/reference/openreader/paginate-query-results).
- [Subsriptions](/sdk/resources/graphql-server/subscriptions) via live queries
- (Deprecated in favor of Relay connections) Lookup queries with the name `{entityName}s`. 

[Union and typed JSON types](/sdk/reference/schema-file/unions-and-typed-json) are mapped into [GraphQL Union Types](https://graphql.org/learn/schema/#union-types) with a [proper type resolution](/sdk/reference/openreader/resolve-union-types-interfaces) with `__typename`.

## Built-in custom scalar types

The OpenReader GraphQL API defines the following custom scalar types:

- `DateTime` entity field values are presented in the ISO format
- `Bytes` entity field values are presented as hex-encoded strings prefixed with `0x`
- `BigInt` entity field values are presented as strings
