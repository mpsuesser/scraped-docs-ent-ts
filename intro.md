---
url: https://ent.dev/docs/intro
title: "Intro"
description: ""
access_date: 2026-08-03T19:01:25.904Z
current_date: 2026-08-03T19:01:25.904Z
---

The `Ent` framework was created to free up time for engineers and teams to focus on what's different about their app as opposed to spending time rebuilding the same cruft that's common across different projects.

This is done by providing a balance of code generation + ways to customize things as needed to integrate nicely with the generated code.

It takes a holistic approach and uses the Schema generated to integrate with the following 3 core layers:

- database.
- middle layer where most of the application logic lives.
- GraphQL layer for exposing the data to clients.

It handles the following high level things:

- managing the database along with database migrations using [alembic](https://alembic.sqlalchemy.org/en/latest/)
- handles CRUD operations based on the application
- first class GraphQL support
- built in permissions across the stack
- ability to write code to integrate with the generated code
- and lots more

## Getting Started

The easiest way to get started is by using the template from the `ent-starter` [repository](https://github.com/lolopinto/ent-starter).

This requires the following:

- [Docker](https://docs.docker.com/get-docker/) installed.
- [Node 16](https://nodejs.org/en/download/) or later installed.
- And one of the database options from below.

There are 3 Database options:

- a local [PostgresQL](https://www.postgresql.org/download/) instance installed
	- a database created via `createdb ent-starter` (or replace with your own database name).
		- if a different database is used, make sure to update the `DB_CONNECTION_STRING` env in `docker-compose.dev.yml`
- Postgres Docker image
	- follow the steps in the [README](https://github.com/lolopinto/ent-starter#postgres-docker-image) in the [ent-starter repository](https://github.com/lolopinto/ent-starter)
		- **Note**: instead of `psql`, run `npm run psql` when following the examples below.
- local [SQLite](https://www.sqlite.org/index.html) file.
	- update the `DB_CONNECTION_STRING` env in `docker-compose.dev.yml` to something like `sqlite:///ent-starter.db` to indicate where the db should be stored.

## Your first schema

In the root of your application, create the schema directory:

```shell
mkdir -p src/schema
```

After setting the environment up, make your first change by specifying a schema as follows:

```ts
import { EntSchema, StringType } from "@snowtop/ent";
import { EmailType } from "@snowtop/ent-email";
import { PasswordType } from "@snowtop/ent-password";

const UserSchema = new EntSchema({
  fields: {
    firstName: StringType(),
    lastName: StringType(),
    emailAddress: EmailType({ unique: true  }),
    password: PasswordType(),
  },
});
export default UserSchema;
```

This does a few things:

- It specifies a new node in the schema called `User`.
- It adds 4 explicitly listed fields here: `firstName`, `lastName`, `emailAddress`, `password`.
- and 3 implicitly listed by virtue of creating a new instance of `EntSchema`: `id`, `createdAt`, `updatedAt`.
- `firstName` and `lastName` are strings.
- `emailAddress` is of type `email` and will be [parsed](https://www.npmjs.com/package/email-addresses) to be a "valid" email address before storing in the database.
- `emailAddress` field is marked as unique so we don't have multiple users with the same email address.
- `password` is of type `password` and hashed and salted using [`bcrypt`](https://www.npmjs.com/package/bcryptjs) before storing in the database.
- The implicit fields are exactly what you'd expect. The default `id` provided by the framework is of type `uuid`.

Then run the following commands:

```shell
# install additional dependencies
npm install @snowtop/ent-email@0.1.0-alpha1 @snowtop/ent-password@0.1.0-alpha1
# generate code + update the database
npm run codegen
```

After running `npm run codegen`, here's a list of generated files:

(The first time this is run, it'll take a while because it's downloading the base Docker image).

```shell
new file:   src/ent/generated/types.ts
new file:   src/ent/generated/loadAny.ts
new file:   src/ent/generated/loaders.ts
new file:   src/ent/generated/user_base.ts
new file:   src/ent/index.ts
new file:   src/ent/internal.ts
new file:   src/ent/user.ts
new file:   src/graphql/index.ts
new file:   src/graphql/generated/resolvers/query_type.ts
new file:   src/graphql/generated/resolvers/user_type.ts
new file:   src/graphql/generated/schema.gql
new file:   src/graphql/generated/schema.ts
new file:   src/graphql/resolvers/index.ts
new file:   src/graphql/resolvers/internal.ts
new file:   src/graphql/generated/resolvers/node_query_type.ts
new file:   src/schema/schema.py
new file:   src/schema/versions/e3b78fe1bfa3_2021518203057_add_users_table.py
```

Let's go over a few:

- `src/ent/user.ts` is the public facing API of the `User` object and what's consumed. It's also where [**custom code**](custom-graphql/custom-accessors.md) can be added.
- `src/ent/generated/user_base.ts` is the base class for the `User` where more generated code will be added as the schema is changed over time.
- `src/schema/versions/*_add_users_tabl.py` is the generated database migration to add the `users` table
- `src/graphql` is where all the generated GraphQL files are with the GraphQL schema file being:
```graphql
type User implements Node {
  id: ID!
  firstName: String!
  lastName: String!
  emailAddress: String!
}

"""node interface"""
interface Node {
  id: ID!
}

type Query {
  node(id: ID!): Node
}
```

After running

```shell
npm run compile && npm start
```

we have a **live** GraphQL server.

## Database changes

Note that the database was also updated by the command run above.

#### Postgres

```markdown
psql ent-starter

# and then
ent-starter=# \d+ users
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
    "users_unique_email_address" UNIQUE CONSTRAINT, btree (email_address)

ent-starter=#
```

#### SQLite

```markdown
sqlite3 ent-starter.db

# and then 
sqlite> .schema users
CREATE TABLE users (
    id TEXT NOT NULL, 
    created_at TIMESTAMP NOT NULL, 
    updated_at TIMESTAMP NOT NULL, 
    first_name TEXT NOT NULL, 
    last_name TEXT NOT NULL, 
    email_address TEXT NOT NULL, 
    password TEXT NOT NULL, 
    CONSTRAINT users_id_pkey PRIMARY KEY (id), 
    CONSTRAINT users_unique_email_address UNIQUE (email_address)
);
sqlite>
```

## Adding writes

To support writes, update the schema as follows:

```ts
import { EntSchema, StringType, ActionOperation } from "@snowtop/ent/schema";
import { EmailType } from "@snowtop/ent-email";
import { PasswordType } from "@snowtop/ent-password";

const UserSchema = new EntSchema({
  fields: {
    firstName: StringType(),
    lastName: StringType(),
    emailAddress: EmailType({ unique: true  }),
    password: PasswordType(),
  },

  actions: [
    {
      operation: ActionOperation.Create,
      fields: ["firstName", "lastName", "emailAddress", "password"],
    }
  ],
});
export default UserSchema;
```

re-run `npm run codegen`

which leads to the following changed files:

```markdown
new file:   src/ent/generated/user/actions/create_user_action_base.ts
new file:   src/ent/generated/user/actions/user_builder.ts
new file:   src/ent/user/actions/create_user_action.ts
new file:   src/graphql/generated/mutations/mutation_type.ts
new file:   src/graphql/generated/mutations/user/user_create_type.ts
modified:   src/graphql/generated/schema.gql
modified:   src/graphql/generated/schema.ts
modified:   src/schema/user_schema.ts
```

and the GraphQL schema updated as follows:

```graphql
type Query {
  node(id: ID!): Node
}

"""node interface"""
interface Node {
  id: ID!
}

type Mutation {
  userCreate(input: UserCreateInput!): UserCreatePayload!
}

type UserCreatePayload {
  user: User!
}

type User implements Node {
  id: ID!
  firstName: String!
  lastName: String!
  emailAddress: String!
}

input UserCreateInput {
  firstName: String!
  lastName: String!
  emailAddress: String!
  password: String!
}
```

Re-compile and restart the server:

```shell
npm run compile && npm start
```

Visit `http://localhost:4000/graphql` in your browser and then execute this query:

```graphql
mutation {
  userCreate(input:{firstName:"John", lastName:"Snow", emailAddress:"test@foo.com", password:"12345678"}) {
    user {
      id
      firstName
      emailAddress
      lastName
    }
  }
}
```

We get this error back:

```json
{
  "errors": [
    {
      "message": "Logged out Viewer does not have permission to create User",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "userCreate"
      ]
    }
  ],
  "data": null
}
```

Update `src/ent/user/actions/create_user_action.ts` as follows:

```ts
import { Data, IDViewer, AlwaysAllowPrivacyPolicy } from "@snowtop/ent";
import {
  CreateUserActionBase,
  UserCreateInput,
} from "src/ent/generated/user/actions/create_user_action_base";

export { UserCreateInput };

export default class CreateUserAction extends CreateUserActionBase {
  getPrivacyPolicy() {
    return AlwaysAllowPrivacyPolicy;
  }

  viewerForEntLoad(data: Data) {
    return new IDViewer(data.id);
  }
}
```

What changed above:

- added `getPrivacyPolicy` method to change [permissions](actions/permissions.md) of who can perform the [action](actions/action.md).
- added `viewerForEntLoad` method to change who the [Viewer](core-concepts/viewer.md) is for the [Ent](core-concepts/ent.md) [load after](actions/viewer-ent-load.md).

Re-compile and restart the server:

```shell
npm run compile && npm start
```

Then rerun the GraphQL query again and you should get a response similar to:

```json
{
  "data": {
    "userCreate": {
      "user": {
        "id": "bm9kZTp1c2VyOjQ1Y2RkNmUyLWY2ZmItNDVlMC1iNWIwLWEwN2JlZWVmM2QxOQ==",
        "firstName": "John",
        "emailAddress": "test@foo.com",
        "lastName": "Snow"
      }
    }
  }
}
```

## Query via GraphQL

The generated GraphQL schema allows you to access any node in your entity graph via the following Query:

```graphql
type Query {
  node(id: ID!): Node
}
```

`Node` is a polymorphic type and can accept an ID for any type of object. You can query for the User you just created with the following GraphQL query - replacing the ID with the ID returned when you created the user!

we get this response back.

```json
{
  "data": {
    "node": null
  }
}
```

Update `src/ent/user.ts` as follows:

```ts
import { PrivacyPolicy, AlwaysAllowPrivacyPolicy } from "@snowtop/ent";
import { UserBase } from "src/ent/internal";

export class User extends UserBase {
  getPrivacyPolicy(): PrivacyPolicy<this> {
    return AlwaysAllowPrivacyPolicy;
  }
}
```

we get the right result back:

```graphql
query ByID {
  node(id:"bm9kZTp1c2VyOjQ1Y2RkNmUyLWY2ZmItNDVlMC1iNWIwLWEwN2JlZWVmM2QxOQ==") {
    id
    ... on User {
      firstName
      lastName
      emailAddress
    }
  }
}
```

Like the change in `create_user_action.ts` above, we've added a `getPrivacyPolicy` method to change [permissions](core-concepts/privacy-policy.md) of who can load the ent. The default generated privacy policy for each ent is [`AllowIfViewerPrivacyPolicy`](core-concepts/privacy-policy.md#allowifviewerprivacypolicy).

Note: The ID exposed to GraphQL is not the same as the ID for the row in the database. GraphQL IDs should be considered opaque by clients of your service. Learn more in the [GraphQL Docs](https://graphql.org/learn/global-object-identification/).

## Custom accessors

Update `src/ent/user.ts` as follows:

```ts
import {
  PrivacyPolicy,
  Viewer,
  Ent,
  ID,
  AlwaysAllowPrivacyPolicy,
} from "@snowtop/ent";
import { gqlField } from "@snowtop/ent/graphql";
import { GraphQLInt } from "graphql";
import { Interval } from "luxon";
import { UserBase } from "src/ent/internal";

export class User extends UserBase {
  getPrivacyPolicy(): PrivacyPolicy<this, Viewer<Ent<any> | null, ID | null>> {
    return AlwaysAllowPrivacyPolicy;
  }

  @gqlField({
    class: "User",
    type: GraphQLInt,
  })
  howLong() {
    return Interval.fromDateTimes(this.createdAt, new Date()).count("seconds");
  }
}
```

and then run the following command:

```shell
npm run codegen && npm run compile && npm start
```

and then run the following GraphQL query:

```graphql
mutation {
  userCreate(input:{firstName:"Sansa", lastName:"Stark", emailAddress:"sansa@stark.com", password:"12345678"}) {
    user {
      id
      firstName
      emailAddress
      lastName
      howLong
    }
  }
}
```

you should get a response similar to:

```json
{
  "data": {
    "userCreate": {
      "user": {
        "id": "bm9kZTp1c2VyOmQ3ZjMzODczLTgwOWUtNGZkMi04YjY4LWQxM2QwNGQwNjYwYw==",
        "firstName": "Sansa",
        "emailAddress": "sansa@stark.com",
        "lastName": "Stark",
        "howLong": 1
      }
    }
  }
}
```

What changed above:

- added a [custom accessor](custom-graphql/custom-accessors.md) using [gqlField](custom-graphql/gql-field.md)

## Query the Database

Run the following commands:

#### Postgres

```markdown
psql ent-starter

# and then

ent-starter=# \x on 
Expanded display is on.
ent-starter=# select * from users;
-[ RECORD 1 ]-+-------------------------------------------------------------
id            | 45cdd6e2-f6fb-45e0-b5b0-a07beeef3d19
created_at    | 2021-05-25 22:37:18.662
updated_at    | 2021-05-25 22:37:18.7
first_name    | John
last_name     | Snow
email_address | test@foo.com
password      | $2a$10$vMDRfwWIacuBHnQsLSym2OwB77Xd.ERj5myqRQEEaAqyXZ5r3xmby
-[ RECORD 2 ]-+-------------------------------------------------------------
id            | d7f33873-809e-4fd2-8b68-d13d04d0660c
created_at    | 2021-05-25 22:43:39.078
updated_at    | 2021-05-25 22:43:39.113
first_name    | Sansa
last_name     | Stark
email_address | sansa@stark.com
password      | $2a$10$q1cwrLhDIiXOXQAjz7zN5u2KC2.QJ.WADfA2ozNuOTvjxrntJGNEC

ent-starter=#
```

#### SQLite

```markdown
sqlite3 ent-starter.db
# and then the following commands
SQLite version 3.32.3 2020-06-18 14:16:19
Enter ".help" for usage hints.
sqlite> .headers on
sqlite> .mode column
sqlite> .separator ROW "\n"
sqlite> select * from users;
id                                    created_at                updated_at                first_name  last_name   email_address  password                                                    
------------------------------------  ------------------------  ------------------------  ----------  ----------  -------------  ------------------------------------------------------------
2b00ec39-12cc-4941-8343-6bb58d958a35  2021-07-06T19:24:55.268Z  2021-07-06T19:24:55.269Z  John        Snow        test@foo.com   $2a$10$s8OvbQbzGqXN6AZ9XujTLOg58u5bmS7sFi8VGbgz0gk/S3lBDki.m
c425044c-55d0-4e0e-a7f9-79e36744df68  2021-07-06T19:27:23.623Z  2021-07-06T19:27:23.637Z  Sansa       Stark       sansa@stark.c  $2a$10$tOr3jfrR/idizjrcgrrR1euibQqNJ3y.O2ntODMthUs94DnxHcQCm
```

## Unique Constraint

Running the following GraphQL query:

```graphql
mutation {
  userCreate(input:{firstName:"Arya", lastName:"Stark", emailAddress:"sansa@stark.com", password:"12345678"}) {
    user {
      id
      firstName
      emailAddress
      lastName
      howLong
    }
  }
}
```

should end with this error because we identified the `emailAddress` as `unique` in the schema above:

```json
{
  "errors": [
    {
      "message": "duplicate key value violates unique constraint \"users_unique_email_address\"",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": [
        "userCreate"
      ]
    }
  ],
  "data": null
}
```

That's a quick introduction to what's supported here. We'll dive deeper into these concepts in the following sections.
