---
url: https://ent.dev/docs/loaders/raw-count-loader
title: "Raw Count Loader"
description: ""
access_date: 2026-08-03T18:23:20.955Z
current_date: 2026-08-03T18:23:20.955Z
---

RawCountLoader is a [`Loader`](loader.md) which is used to generate the rawCount for the number of rows in the database.

It's used by [index based queries](../core-concepts/ent-query.md#index-based-query) as well as [custom ent queries](../custom-data-access/custom-queries.md#custom-entquery).

## RawCountLoaderFactory

Public Facing [LoaderFactory](loader.md#loaderfactory) of `RawCountLoader` used to create new instances of `RawCountLoader` as needed.

## API

```ts
interface QueryCountOptions {
    tableName: string;
    groupCol?: string;
    clause?: clause.Clause;
}
declare class RawCountLoader<K extends any> implements Loader<K, number> {
    context?: Context | undefined;
    constructor(options: QueryCountOptions, context?: Context | undefined);
}
declare class RawCountLoaderFactory implements LoaderFactory<ID, number> {
    name: string;
    constructor(options: QueryCountOptions);
    createLoader(context?: Context): any;
}
```
- `tableName`: relevant table in the database.
- `groupCol`: column in the database that can be converted into an [IN query](https://www.w3schools.com/sql/sql_in.asp) when querying for multiple sources
- `clause`: [Clause](../advanced-topics/clause.md) instance for filtering.

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
  }; 
}); 
export default TodoSchema;
```

here are some examples:

- count of open todos of a user. can be used to fetch multiple users at a time
```ts
const openTodosCountLoader = new RawCountLoaderFactory({
  tableName: Todo.loaderOptions().tableName,
  groupCol: "creator_id",
  clause: query.Eq("completed", false),
});
```

generates query like this

```sql
SELECT count(1) as count FROM todos WHERE creator_id IN ($1, $2, $3) AND completed = $4 GROUP BY creator_id;
```
- count of open todos of a user. *cannot* be used to fetch multiple users at a time
```ts
const openTodosCountLoader = new RawCountLoaderFactory({
  tableName: Todo.loaderOptions().tableName,
  clause: query.And(query.Eq("creator_id", user.id, query.Eq("completed", false)),
});
```

generates query like this

```sql
SELECT count(1) as count FROM todos WHERE creator_id = $1 AND completed = $2;
```
- count of all todos of a user. can be used to fetch multiple users at a time
```ts
const allTodosCountLoader = new RawCountLoaderFactory({
  tableName: Todo.loaderOptions().tableName,
  groupCol: "creator_id",
});
```

generates query like this

```sql
SELECT count(1) as count FROM todos WHERE creator_id IN ($1, $2, $3) GROUP BY creator_id;
```
- count of all todos of a user. *cannot* be used to fetch multiple users at a time
```ts
const allTodosCountLoader = new RawCountLoaderFactory({
  tableName: Todo.loaderOptions().tableName,
  clause: query.Eq("creator_id", user.id),
});
```

generates query like this

```sql
SELECT count(1) as count FROM todos WHERE creator_id = $1;
```
