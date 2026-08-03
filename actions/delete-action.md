---
url: https://ent.dev/docs/actions/delete-action
title: "Delete Action"
description: ""
access_date: 2026-08-03T19:09:20.358Z
current_date: 2026-08-03T19:09:20.358Z
---

This is done via the `ActionOperation.Delete` or `ActionOperation.Mutations` [operation](../ent-schema/actions.md#operation).

Based on the [schema](action.md#schema) with the following extra configuration:

```ts
const EventSchema = new EntSchema({

actions: [
    {
      operation: ActionOperation.Delete,
    },
  ], 
}); 
export default EventSchema;
```

leads to 2 classes.

First, the base class:

```ts
export class DeleteEventActionBase 
  implements
    Action<
      Event,
      EventBuilder<EventInput, Event>,
      Viewer,
      EventInput,
      Event
    >
{
  public readonly builder: EventBuilder<EventInput, Event>;
  public readonly viewer: Viewer;
  protected readonly event: Event;

  constructor(viewer: Viewer, event: Event) {
    this.viewer = viewer;
    this.builder = new EventBuilder(
      this.viewer,
      WriteOperation.Delete,
      this,
      event,
    );
    this.event = event;
  }

  getPrivacyPolicy(): PrivacyPolicy {
    return AllowIfViewerHasIdentityPrivacyPolicy;
  }
  // ...
}
```

and then the subclass:

```ts
import {
import { DeleteEventActionBase } from "src/ent/generated/event/actions/delete_event_action_base"; 

export default class DeleteEventAction extends DeleteEventActionBase {}
```

The base class `DeleteEventActionBase` is where all shared functionality is and will be regenerated as the action configuration changes. It has the default privacy policy plus a bunch of other methods shown below.

The subclass will be generated **once** and any customizations can be applied there.

## Usage

```ts
const event = await createEvent();

// deletes event and throws if it couldn't delete
await DeleteEventAction.create(viewer, event).saveX();

// deletes event and fails silently if it couldn't delete
await DeleteEventAction.create(viewer, event).save();
```

## GraphQL

The following GraphQL schema is generated which uses the above API.

```markdown
type Mutation {
  eventDelete(input: EventDeleteInput!): EventDeletePayload!
}

input EventDeleteInput {
  eventID: ID!
}

type EventDeletePayload {
  deletedEventID: ID
}

type Event implements Node {
  creator: User
  id: ID!
  name: String!
  startTime: Time!
  endTime: Time
  eventLocation: String!
  ///.... 
}
```

and called as follows:

```graphql
mutation eventDeleteMutation($input: EventDeleteInput!) {
  eventDelete(input: $input) {
    deletedEventID
  }
}
```
