---
url: https://ent.dev/docs/ent-schema/edges
title: "Edges"
description: ""
access_date: 2026-08-03T18:55:24.403Z
current_date: 2026-08-03T18:55:24.403Z
---

Edges provide the ability to express connections between nodes in the graph. They allow a standardized way to indicate a relationship.

They're often used when there's a many-to-many relationship between two entities. For example, the following are perfect use cases:

- friends in a social network
- attendees of an event
- members of a group
- followers of a profile in a social network
- photos of an album

They can also be used for 1-many or 1-1 relationships as needed.

The advantages of edges is they're standardized in the framework and make it easy to iterate quickly.

## Types of edges

There are three types of edges supported.

### one-way edge

This is the default. This can be used for 1-many edges.

For example, given an events based system, the creator can be stored in the `events` table and the list of created events for a user can be represented with an edge. This can also be represented with an [indexed foreign key](fields.md#foreignkey) on the `events` table but if you'd rather not have foreign keys, here's an easy alternative.

```ts
const EventSchema = new EntSchema({
  fields: {
    creatorID: UUIDType({
      fieldEdge: { schema: "User", inverseEdge: "createdEvents" },
      storageKey: "user_id",
    }),
  }, 
}); 
export default EventSchema;
```
```ts
const UserSchema = new EntSchema({
  fields: {},

  edges: Edge[] = [
    {
      name: "createdEvents",
      schemaName: "Event",
    },
  ],
});
export default UserSchema;
```

### symmetric edge

This represents an edge that has the same relationship on both sides. For example, friends in a social network system.

```ts
const UserSchema = new EntSchema({
  fields: {}, 

  edges: [
    {
      name: "friends",
      schemaName: "User",
      symmetric: true,
    },
  ], 
}); 
export default UserSchema;
```

Anytime an edge is written from `id1` to `id2`, the system automatically writes the inverse edge from `id2` to `id1` with the same `time` and `data` fields. This makes it easy to query from either side of the connection, e.g. fetching the list of friends of either user.

In the future, once we support different shards, the benefit of this design will be seen even more as each edge would/should be collocated on the same shard.

### inverse edge

This represents a many-many edge that has a different relationship on the other side. For example,

- attendees of an event
	- one-way: user -> events attending
		- other-way: event -> users attending
- members of a group
	- one-way: user -> groups user member of
		- other-way: group -> members of
- followers of a profile in a social network
	- one-way: user -> followers
		- other-way: user -> people followed by

Inverse edge should be used if you ever want the count or to list the nodes at the end of the inverse edge.

To express the hosts of an event and the inverse, events hosted by a user, the schema is expressed as follows:

```ts
const EventSchema = new EntSchema({
  fields: {},

  edges: [
    {
      name: "hosts",
      schemaName: "User",
      inverseEdge: {
        name: "userToHostedEvents",
      },
    },
  ],
});
export default EventSchema;
```

Anytime an edge is written from `id1` to `id2`, the system automatically writes the inverse edge from `id2` to `id1` with the same `time` and `data` fields. This makes it easy to query from either side of the connection.

In the future, once we support different shards, the benefit of this design will be seen even more as each edge would/should be collocated on the same shard.

## Database

A standard edge table has the following columns:

#### Postgres

```markdown
ent-rsvp=# \d+ event_rsvps
                                            Table "public.event_rsvps"
  Column   |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
-----------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 id1       | uuid                        |           | not null |         | plain    |              | 
 id1_type  | text                        |           | not null |         | extended |              | 
 edge_type | uuid                        |           | not null |         | plain    |              | 
 id2       | uuid                        |           | not null |         | plain    |              | 
 id2_type  | text                        |           | not null |         | extended |              | 
 time      | timestamp without time zone |           | not null |         | plain    |              | 
 data      | text                        |           |          |         | extended |              | 
Indexes:
    "event_rsvps_id1_edge_type_id2_pkey" PRIMARY KEY, btree (id1, edge_type, id2)
    "event_rsvps_time_idx" btree ("time")

ent-rsvp=#
```

#### SQLite

```markdown
sqlite> .schema event_rsvps
CREATE TABLE event_rsvps (
    id1 TEXT NOT NULL, 
    id1_type TEXT NOT NULL, 
    edge_type TEXT NOT NULL, 
    id2 TEXT NOT NULL, 
    id2_type TEXT NOT NULL, 
    time TIMESTAMP NOT NULL, 
    data TEXT, 
    CONSTRAINT event_rsvps_id1_edge_type_id2_pkey PRIMARY KEY (id1, edge_type, id2)
);
CREATE INDEX event_rsvps_time_idx ON event_rsvps (time);
```

- `id1` represents the source of the edge
- `id1_type` is the *type* of `id1`
- `edge_type` is a unique identifier that represents the edge that's created by the framework
- `id2` represents the destination of the edge
- `id2_type` represents the *type* of `id2`
- `time` represents the time the edge was created. However, it can be used to represent any sortKey since edges are sorted by `time` in descending order. So, it can be overriden to represent the native order of a collection for example
- `data` is just for any extra data that should be associated with this edge. It defaults to `NULL`

### indices

There's a primary key on three fields: `id1`, `edge_type`, and `id2` because each edge is unique on those three fields.

The `time` field is indexed to optimize querying since edges are by default sorted by time in descending order.

We currently don't support adding custom indices or customizing the table at the moment but could in the future.

## Options

### name

name of the edge. Edge names should be unique in each schema.

### schemaName

name of the schema at the end of the edge e.g. `User`, `Event`.

### symmetric

boolean indicating [symmetric](edges.md#symmetric-edge) edge.

### unique

boolean indicating edge is unique. [Current limitation](https://github.com/lolopinto/ent/issues/38) is that it doesn't work when edges are shared across the same table.

In a contact management system, to represent an edge from the `User` to their own `Contact`

```ts
const UserSchema = new EntSchema({
  fields: {}, 

  edges: [
    {
      name: "selfContact",
      unique: true,
      schemaName: "Contact",
    },
  ], 
}); 
export default UserSchema;
```

results in a unique constraint added to the db

#### Postgres

```markdown
tsent_test=# \d+ user_self_contact_edges
                                      Table "public.user_self_contact_edges"
  Column   |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
-----------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 id1       | uuid                        |           | not null |         | plain    |              | 
 id1_type  | text                        |           | not null |         | extended |              | 
 edge_type | uuid                        |           | not null |         | plain    |              | 
 id2       | uuid                        |           | not null |         | plain    |              | 
 id2_type  | text                        |           | not null |         | extended |              | 
 time      | timestamp without time zone |           | not null |         | plain    |              | 
 data      | text                        |           |          |         | extended |              | 
Indexes:
    "user_self_contact_edges_id1_edge_type_id2_pkey" PRIMARY KEY, btree (id1, edge_type, id2)
    "user_self_contact_edges_unique_id1_edge_type" UNIQUE CONSTRAINT, btree (id1, edge_type)
    "user_self_contact_edges_time_idx" btree ("time")

tsent_test=# 
\`\`\`
```

#### SQLite

```markdown
CREATE TABLE user_self_contact_edges (
    id1 TEXT NOT NULL, 
    id1_type TEXT NOT NULL, 
    edge_type TEXT NOT NULL, 
    id2 TEXT NOT NULL, 
    id2_type TEXT NOT NULL, 
    time TIMESTAMP NOT NULL, 
    data TEXT, 
    CONSTRAINT user_self_contact_edges_id1_edge_type_id2_pkey PRIMARY KEY (id1, edge_type, id2), 
    CONSTRAINT user_self_contact_edges_unique_id1_edge_type UNIQUE (id1, edge_type)
);
CREATE INDEX user_self_contact_edges_time_idx ON user_self_contact_edges (time);
sqlite>
```

### inverseEdge

allows configuring the [inverse edge](edges.md#inverse-edge).

### tableName

allows one to override the name of the table generated for this edge.

### hideFromGraphQL

hides the edge from being exposed as a `Connection` in GraphQL. This is used for things that shouldn't be exposed in the public API e.g. data that's internal to the system.

### edgeActions

allows configuring the [actions](../actions/action.md) generated for this edge. Two actions are currently supported:

- [add edge](../actions/add-edge-action.md)
- [remove edge](../actions/remove-edge-action.md)

## assoc-edge-config table

Each created edge is stored in the `assoc_edge_config` table. This is the source of truth for the edge and ensures a consistent data source for edge data.

Here's what the table looks like:

#### Postgres

```markdown
ent-starter=# \d+ assoc_edge_config
                                             Table "public.assoc_edge_config"
      Column       |            Type             | Collation | Nullable | Default | Storage  | Stats target | Description 
-------------------+-----------------------------+-----------+----------+---------+----------+--------------+-------------
 edge_type         | uuid                        |           | not null |         | plain    |              | 
 edge_name         | text                        |           | not null |         | extended |              | 
 symmetric_edge    | boolean                     |           | not null | false   | plain    |              | 
 inverse_edge_type | uuid                        |           |          |         | plain    |              | 
 edge_table        | text                        |           | not null |         | extended |              | 
 created_at        | timestamp without time zone |           | not null |         | plain    |              | 
 updated_at        | timestamp without time zone |           | not null |         | plain    |              | 
Indexes:
    "assoc_edge_config_edge_type_pkey" PRIMARY KEY, btree (edge_type)
    "assoc_edge_config_unique_edge_name" UNIQUE CONSTRAINT, btree (edge_name)
Foreign-key constraints:
    "assoc_edge_config_inverse_edge_type_fkey" FOREIGN KEY (inverse_edge_type) REFERENCES assoc_edge_config(edge_type) ON DELETE RESTRICT
Referenced by:
    TABLE "assoc_edge_config" CONSTRAINT "assoc_edge_config_inverse_edge_type_fkey" FOREIGN KEY (inverse_edge_type) REFERENCES assoc_edge_config(edge_type) ON DELETE RESTRICT

ent-starter=#
```

#### SQLite

```markdown
sqlite> .schema assoc_edge_config
CREATE TABLE assoc_edge_config (
    edge_type TEXT NOT NULL, 
    edge_name TEXT NOT NULL, 
    symmetric_edge BOOLEAN DEFAULT 'false' NOT NULL, 
    inverse_edge_type TEXT, 
    edge_table TEXT NOT NULL, 
    created_at TIMESTAMP NOT NULL, 
    updated_at TIMESTAMP NOT NULL, 
    CONSTRAINT assoc_edge_config_edge_type_pkey PRIMARY KEY (edge_type), 
    CONSTRAINT assoc_edge_config_inverse_edge_type_fkey FOREIGN KEY(inverse_edge_type) REFERENCES assoc_edge_config (edge_type) ON DELETE RESTRICT, 
    CONSTRAINT assoc_edge_config_unique_edge_name UNIQUE (edge_name)
);
sqlite>
```
