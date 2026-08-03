---
url: https://ent.dev/docs/ent-schema/indices
title: "Indices"
description: ""
access_date: 2026-08-03T17:27:01.267Z
current_date: 2026-08-03T17:27:01.267Z
---

This allows configuring indices in the database.

The easiest way to add an index on a single column is to use the [index modifier](fields.md#index) on the field.

However, to add a multi-column index if we're querying a lot on the price of items:

```ts
import {  FloatType, EntSchema } from "@snowtop/ent"; 

const ProductItemSchema = new EntSchema({
  fields: {
    price: FloatType(),
    discount_price: FloatType(),
  }, 

  indices: [
    {
      name: "product_items_idx",
      columns: ["price", "discount_price"],
    },
  ], 
}); 
export default ProductItemSchema;
```

which leads to

- Postgres
- SQLite

```markdown
ent-test=# \d+ product_items
                                             Table "public.product_items"
     Column     |            Type             | Collation | Nullable | Default | Storage | Stats target | Description 
----------------+-----------------------------+-----------+----------+---------+---------+--------------+-------------
 id             | uuid                        |           | not null |         | plain   |              | 
 created_at     | timestamp without time zone |           | not null |         | plain   |              | 
 updated_at     | timestamp without time zone |           | not null |         | plain   |              | 
 price          | double precision            |           | not null |         | plain   |              | 
 discount_price | double precision            |           | not null |         | plain   |              | 
Indexes:
    "product_items_id_pkey" PRIMARY KEY, btree (id)
    "product_items_idx" btree (price, discount_price)
```

## Concurrent indexes

Postgres only: set `concurrently: true` on the index or `indexConcurrently: true` on the field index to render `CREATE INDEX CONCURRENTLY`.

```ts
import { EntSchema, FloatType } from "@snowtop/ent";

const ProductItemSchema = new EntSchema({
  fields: {
    price: FloatType({ index: true, indexConcurrently: true }),
  },
  indices: [
    {
      name: "product_items_price_idx",
      columns: ["price"],
      concurrently: true,
    },
  ],
});
export default ProductItemSchema;
```

## Partial indexes

Partial indexes use a SQL `where` clause on the index, or `indexWhere` on a field index. The clause is raw SQL and should reference database column names.

```ts
import { EntSchema, FloatType } from "@snowtop/ent";

const ProductItemSchema = new EntSchema({
  fields: {
    price: FloatType({ index: true, indexWhere: "price > 0" }),
    discount_price: FloatType(),
  },
  indices: [
    {
      name: "product_items_discount_idx",
      columns: ["discount_price"],
      where: "discount_price IS NOT NULL",
    },
  ],
});
export default ProductItemSchema;
```
