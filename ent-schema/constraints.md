---
url: https://ent.dev/docs/ent-schema/constraints
title: "Constraints"
description: ""
access_date: 2026-08-03T17:27:01.267Z
current_date: 2026-08-03T17:27:01.267Z
---

This allows configuring constraints in the database.

## unique constraint

The easiest way to add a unique constraint on a single column is to use the [unique modifier](fields.md#unique) on the field.

However, to add a multi-column constraint:

```ts
const GuestSchema = new EntSchema({
  fields: [
    eventID: UUIDType({
      foreignKey: { schema: "Event", column: "id" },
    }),
    emailAddress: EmailType({ nullable: true }),
  ],

  constraints: [
    {
      name: "uniqueEmail",
      type: ConstraintType.Unique,
      columns: ["eventID", "emailAddress"],
    },
  ],
});
export default GuestSchema;
```

leads to database change

- Postgres
- SQLite

```markdown
ent-rsvp=# \d+ guests
                                                 Table "public.guests"
     Column     |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
----------------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 id             | uuid                        |           | not null |         | plain    |              | 
 created_at     | timestamp without time zone |           | not null |         | plain    |              | 
 updated_at     | timestamp without time zone |           | not null |         | plain    |              | 
 email_address  | text                        |           |          |         | extended |              | 
 event_id       | uuid                        |           | not null |         | plain    |              | 
Indexes:
    "guests_id_pkey" PRIMARY KEY, btree (id)
    "uniqueEmail" UNIQUE CONSTRAINT, btree (event_id, email_address)
    "guests_event_id_idx" btree (event_id)
Foreign-key constraints:
    "guests_event_id_fkey" FOREIGN KEY (event_id) REFERENCES events(id) ON DELETE CASCADE
ent-rsvp=#
```

## primary key constraint

The easiest way to add a primary key constraint on a single column is to use the [primaryKey modifier](fields.md#primarykey) on the field.

To add a multi-column constraint:

```ts
import { EntSchema, UUIDType, ConstraintType } from "@snowtop/ent";

const UserPhotoSchema = new EntSchema({
  fields: {
    UserID: UUIDType(),
    PhotoID: UUIDType(),
  },

  constraints: [
    {
      name: "user_photos_pkey",
      type: ConstraintType.PrimaryKey,
      columns: ["UserID", "PhotoID"],
    },
  ],
});
export default UserPhotoSchema;
```

leads to database change:

- Postgres
- SQLite

```markdown
ent-test=# \d+ user_photos
                               Table "public.user_photos"
  Column  | Type | Collation | Nullable | Default | Storage | Stats target | Description 
----------+------+-----------+----------+---------+---------+--------------+-------------
 user_id  | uuid |           | not null |         | plain   |              | 
 photo_id | uuid |           | not null |         | plain   |              | 
Indexes:
    "user_photos_pkey" PRIMARY KEY, btree (user_id, photo_id)
```

TODO: Currently, there's an issue here that needs to be fixed: [https://github.com/lolopinto/ent/issues/328](https://github.com/lolopinto/ent/issues/328)

## foreign key constraint

The easiest way to add a foreign key constraint on a single column is to use the [foreignKey modifier](fields.md#foreignkey) on the field.

The columns being referenced on the other table need to be unique either via a multi-column unique constraint or primary key.

In this contrived example with the following schema,

```ts
import { EntSchema, StringType, ConstraintType } from "@snowtop/ent/schema";
import { EmailType } from "@snowtop/ent-email";
import { PasswordType } from "@snowtop/ent-password";

const UserSchema = new EntSchema({
  fields: {
    FirstName: StringType(),
    LastName: StringType(),
    EmailAddress: EmailType(),
    Password: PasswordType(),
  },

  constraints: [
    {
      name: "user_uniqueEmail",
      type: ConstraintType.Unique,
      columns: ["id", "EmailAddress"],
    },
  ],
});
export default UserSchema;
```
```ts
import { EntSchema, UUIDType, StringType, ConstraintType } from "@snowtop/ent";

const ContactSchema = new EntSchema({
  fields: {
    emailAddress: StringType(),
    userID: UUIDType(),
  };

  constraints: [
    {
      name: "contacts_user_fkey",
      type: ConstraintType.ForeignKey,
      columns: ["userID", "emailAddress"],
      fkey: {
        tableName: "users",
        ondelete: "CASCADE",
        columns: ["id", "EmailAddress"],
      }
    },
  ],
});
export default ContactSchema;
```

leads to

- Postgres
- SQLite

```markdown
ent-test=# \d+ users
                                                 Table "public.users"
    Column     |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
---------------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 id            | uuid                        |           | not null |         | plain    |              | 
 created_at    | timestamp without time zone |           | not null |         | plain    |              | 
 updated_at    | timestamp without time zone |           | not null |         | plain    |              | 
 first_name    | text                        |           | not null |         | extended |              | 
 last_name     | text                        |           | not null |         | extended |              | 
 email_address | text                        |           | not null |         | extended |              | 
 password      | text                        |           | not null |         | extended |              | 
Indexes:
    "users_id_pkey" PRIMARY KEY, btree (id)
    "user_uniqueEmail" UNIQUE CONSTRAINT, btree (id, email_address)
Referenced by:
    TABLE "contacts" CONSTRAINT "contacts_user_fkey" FOREIGN KEY (user_id, email_address) REFERENCES users(id, email_address) ON DELETE CASCADE

ent-test=# \d+ contacts
                                               Table "public.contacts"
    Column     |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
---------------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 id            | uuid                        |           | not null |         | plain    |              | 
 created_at    | timestamp without time zone |           | not null |         | plain    |              | 
 updated_at    | timestamp without time zone |           | not null |         | plain    |              | 
 email_address | text                        |           | not null |         | extended |              | 
 user_id       | uuid                        |           | not null |         | plain    |              | 
Indexes:
    "contacts_id_pkey" PRIMARY KEY, btree (id)
Foreign-key constraints:
    "contacts_user_fkey" FOREIGN KEY (user_id, email_address) REFERENCES users(id, email_address) ON DELETE CASCADE
```

## check constraint

adds a [check constraint](https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS) to the schema.

For example,

```ts
import { FloatType, EntSchema, ConstraintType } from "@snowtop/ent";

const ItemSchema = new EntSchema({
  fields: {
    price: FloatType(),
  },

  constraints: [
    {
      name: "item_positive_price",
      type: ConstraintType.Check,
      condition: 'price > 0',
      columns: [],
    },
  ],
});
export default ItemSchema;
```

leads to

- Postgres
- SQLite

```markdown
ent-test=# \d+ items
                                               Table "public.items"
   Column   |            Type             | Collation | Nullable | Default | Storage | Stats target | Description 
------------+-----------------------------+-----------+----------+---------+---------+--------------+-------------
 id         | uuid                        |           | not null |         | plain   |              | 
 created_at | timestamp without time zone |           | not null |         | plain   |              | 
 updated_at | timestamp without time zone |           | not null |         | plain   |              | 
 price      | double precision            |           | not null |         | plain   |              | 
Indexes:
    "items_id_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "item_positive_price" CHECK (price > 0::double precision)

ent-test=#
```

or for something more complicated

```ts
import { FloatType, EntSchema, ConstraintType } from "@snowtop/ent";

const ProductItemSchema = new EntSchema({
  fields: {
    price: FloatType(),
    discount_price: FloatType(),
  },

  constraints: [
    {
      name: "item_positive_price",
      type: ConstraintType.Check,
      condition: 'price > 0',
      columns: [],
    },
    {
      name: "item_positive_discount_price",
      type: ConstraintType.Check,
      condition: 'discount_price > 0',
      columns: [],
    },
    {
      name: "item_price_greater_than_discount",
      type: ConstraintType.Check,
      condition: 'price > discount_price',
      columns: [],
    },
  ],
});
export default ProductItemSchema;
```

leads to

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
Check constraints:
    "item_positive_discount_price" CHECK (discount_price > 0::double precision)
    "item_positive_price" CHECK (price > 0::double precision)
    "item_price_greater_than_discount" CHECK (price > discount_price)
```

## options

### name

name of the constraint

### type

constraint type. currently supported:

- [ConstraintType.PrimaryKey](#primary-key-constraint)
- [ConstraintType.ForeignKey](#foreign-key-constraint)
- [ConstraintType.Unique](#unique-constraint)
- [ConstraintType.Check](#check-constraint)

### columns

list of columns constraint applies to. At least 1 column required for primary key, foreign key and unique constraints

### fkey

configures the foreignKey constraint. [See above](#foreign-key-constraint).

`ondelete` options are: `RESTRICT`, `CASCADE`, `SET NULL`, `SET DEFAULT`, `NO ACTION`.

### condition

condition that resolves to a boolean that should be added to a check constraint.
