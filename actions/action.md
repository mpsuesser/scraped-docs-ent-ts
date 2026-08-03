---
url: https://ent.dev/docs/actions/action
title: "Action"
description: ""
access_date: 2026-08-03T18:55:24.403Z
current_date: 2026-08-03T18:55:24.403Z
---

Action is the way to perform writes in the system. The [configuration](../ent-schema/actions.md) determines what code is generated and the developer is able to further customize as needed.

There are high level 3 different modes when writing:

- creating a node
- editing the node
- deleting the node

There are however different configuration options that end up breaking down slightly differently:

- creating a node
	- [create action](create-action.md)
- editing a node
- deleting a node:
	- [delete action](delete-action.md)

## Customizations

There are different ways to customize the action after the code has been generated:

- permissions: is the viewer allowed to perform this action
- validators: validate the input in addition to the per-field validation that may already exist
- triggers: customize the list of side effects that should be added to the transaction related to this action
- observers: customize the list of external side effects e.g. send an email, send a text, add some logging, etc.

We'll dive into each of these in the following sections.

## Transactions

If you need to coordinate multiple actions at the call site, use a [Transaction](transactions.md) to run them together. For action-internal work, use [Triggers](triggers.md) since they run inside the action's own transaction.

## Default Privacy Policy

The default [privacy policy](../core-concepts/privacy-policy.md) is that any logged in user i.e. Viewer's [viewerID](../core-concepts/viewer.md#viewerid) is not `null` can perform the action.

## Schema

We'll use the following schema as our base example and go into each of them:

```ts
const EventSchema = new EntSchema({
  fields: {
    name: StringType(),
    creatorID: UUIDType({
      fieldEdge: { schema: "User", inverseEdge: "createdEvents" },
      storageKey: "user_id",
    }),
    start_time: TimestampType(),
    end_time: TimestampType({ nullable: true}),
    location: StringType({
      graphqlName: "eventLocation",
    }),
  }, 

  edges: [
    {
      name: "hosts",
      schemaName: "User",
      inverseEdge: {
        name: "userToHostedEvents",
      },
      edgeActions: [
        {
          operation: ActionOperation.AddEdge,
        },
        {
          operation: ActionOperation.RemoveEdge,
        },
      ],
    },
  ], 

  edgeGroups: [
    {
      name: "rsvps",
      groupStatusName: "rsvpStatus",
      nullStates: ["canRsvp"],
      statusEnums: ["attending", "declined", "maybe"],
      edgeAction: {
        operation: ActionOperation.EdgeGroup,
      },
      assocEdges: [
        {
          name: "invited",
          schemaName: "User",
          inverseEdge: {
            name: "invitedEvents",
          },
        },
        {
          // yes
          name: "attending",
          schemaName: "User",
          inverseEdge: {
            name: "eventsAttending",
          },
        },
        {
          // no
          name: "declined",
          schemaName: "User",
          inverseEdge: {
            name: "declinedEvents",
          },
        },
        {
          // maybe
          name: "maybe",
          schemaName: "User",
          inverseEdge: {
            name: "maybeEvents",
          },
        },
      ],
    },
  ], 
}); 
export default EventSchema;
```
