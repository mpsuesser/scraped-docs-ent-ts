---
url: https://ent.dev/docs/custom-graphql/gql-connection
title: "Gql Connection"
description: ""
access_date: 2026-08-03T19:09:20.358Z
current_date: 2026-08-03T19:09:20.358Z
---

`gqlConnection` is the [type](gql-field.md#type) of a [`gqlField`](gql-field.md) to indicate that it should be exposed as a [GraphQL Connection](https://graphql.org/learn/pagination/#complete-connection-model) on the source object that follows the [Relay Spec](https://relay.dev/graphql/connections.htm).

It takes the name of the node that's going to be at the end of the connection.

Usually used to [query the database in custom ways](../custom-data-access/custom-queries.md).

It expects the method to return a Custom [EntQuery](../core-concepts/ent-query.md) otherwise things won't work.

For example:

```ts
export class Account extends AccountBase {

@gqlField({ class: "Account", name: "openTodos", type: gqlConnection("Todo") })
  openTodos() {
    return new AccountToOpenTodosQuery(this.viewer, this);
  }
}
```
