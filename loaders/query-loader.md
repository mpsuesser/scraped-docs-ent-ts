---
url: https://ent.dev/docs/loaders/query-loader
title: "Query Loader"
description: ""
access_date: 2026-08-03T18:55:24.403Z
current_date: 2026-08-03T18:55:24.403Z
---

QueryLoader is a [`Loader`](loader.md) which is used to fetch multiple rows in the database.

It's used by [index based queries](../core-concepts/ent-query.md#index-based-query) as well as [custom ent queries](../custom-data-access/custom-queries.md#custom-entquery).

## QueryLoaderFactory

Public Facing [LoaderFactory](loader.md#loaderfactory) of `QueryLoader` used to create new instances of `QueryLoader` as needed.

## API

```ts
interface QueryOptions {
    fields: string[];
    tableName: string;
    groupCol?: string;
    clause?: clause.Clause;
    orderby?: OrderBy;
    toPrime?: ObjectLoaderFactory<ID>[];
}
class QueryLoaderFactory<K extends any> implements LoaderFactory<K, Data[]> {
    constructor(options: QueryOptions);
}
```
- `fields`: columns in the database to query for.
- `tableName`: relevant table in the database.
- `groupCol`: column in the database that can be converted into an [IN query](https://www.w3schools.com/sql/sql_in.asp) when querying for multiple sources
- `clause`: [Clause](../advanced-topics/clause.md) instance for filtering.
- `orderby`: optional. Used to order the query. If not provided, we sort by the `id` field descending.
- `toPrime`: optional. Other loaders that can be primed by the result of this query with the result stored in the [context cache](../core-concepts/context-caching.md).

Note that at least one of `groupCol` and `clause` must be provided.

## Examples

Given the following schema:

```ts
const TodoSchema = new EntSchema({
  fields: {
    text: StringType(),
    completed: BooleanType({
      index: true,
      defaultValueOnCreate: () => {
        return false;
      },
    }),
    creatorID: UUIDType({
      foreignKey: { schema: "Account", column: "id" },
    }),
  }, 
}); 
export default TodoSchema;
```

here are some examples:

- open todos of a user. can be used to fetch multiple users at a time
```ts
const openTodosLoader = new QueryLoaderFactory({
  ...Todo.loaderOptions(),
  groupCol: "creator_id",
  clause: query.Eq("completed", false),
  toPrime: [todoLoader],
});
```
- open todos of a user. *cannot* be used to fetch multiple users at a time
```ts
const openTodosLoader = new QueryLoaderFactory({
  ...Todo.loaderOptions(),
  clause: query.And(query.Eq("creator_id", user.id, query.Eq("completed", false)),
  toPrime: [todoLoader],
});
```
- all todos of a user. can be used to fetch multiple users at a time
```ts
const allTodosLoader = new QueryLoaderFactory({
  ...Todo.loaderOptions(),
  groupCol: "creator_id",
  toPrime: [todoLoader],
});
```
- all todos of a user. *cannot* be used to fetch multiple users at a time
```ts
const allTodosLoader = new QueryLoaderFactory({
  ...Todo.loaderOptions(),
  clause: query.Eq("creator_id", user.id),
  toPrime: [todoLoader],
});
```

Note that the best way to use QueryLoader is via a [custom query](../custom-data-access/custom-queries.md#custom-entquery) as that gives you the benefits of pagination, sorting, filtering etc.
