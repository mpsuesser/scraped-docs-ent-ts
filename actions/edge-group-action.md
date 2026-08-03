---
url: https://ent.dev/docs/actions/edge-group-action
title: "Edge Group Action"
description: ""
access_date: 2026-08-03T19:01:25.904Z
current_date: 2026-08-03T19:01:25.904Z
---

This is done via the `ActionOperation.EdgeGroup` [operation](../ent-schema/actions.md#operation).

Based on the [schema](action.md#schema) with the `edgeAction` property in the group named `rsvps` leads to 2 classes.

First, the base class:

```ts
export enum EventRsvpStatusInput {
  Attending = "attending", 
  Declined = "declined", 
  Maybe = "maybe", 
}

export interface EditEventRsvpStatusInput {
  rsvpStatus: EventRsvpStatusInput; 
  userID: ID; 
}

export class EditEventRsvpStatusActionBase 
  implements
    Action<
      Event,
      EventBuilder<EditEventRsvpStatusInput, Event>,
      Viewer,
      EventInput,
      Event
{
  public readonly builder: EventBuilder<EditEventRsvpStatusInput, Event>;
  public readonly viewer: Viewer; 
  protected readonly event: Event;

  constructor(viewer: Viewer, event: Event, input: EditEventRsvpStatusInput) {
    this.viewer = viewer;
    this.input = input;
    this.builder = new EventBuilder(
      this.viewer,
      WriteOperation.Edit,
      this,
      event,
    );
    this.event = event;
  }

  getPrivacyPolicy(): PrivacyPolicy {
    return AllowIfViewerHasIdentityPrivacyPolicy;
  }
}
```

and then the subclass:

```ts
import {
  EditEventRsvpStatusActionBase,
  EditEventRsvpStatusInput,
} from "src/ent/generated/event/actions/edit_event_rsvp_status_action_base";

export { EditEventRsvpStatusInput };

export default class EditEventRsvpStatusAction extends EditEventRsvpStatusActionBase {}
```

The base class `EditEventRsvpStatusActionBase` is where all shared functionality is and will be regenerated as the action configuration changes. It has the default privacy policy plus a bunch of other methods shown below.

The subclass will be generated **once** and any customizations can be applied there.

## Usage

```ts
const [event, user] = await Promise.all([
  createEvent(),
  createUser(),
]);

// sets the user's rsvp status to attending
const event2 = await EditEventRsvpStatusAction.saveXFromID(viewer, event.id, {
  userID: user.id,
  rsvpStatus: EventRsvpStatusInput.Attending,
});

// sets the user's rsvp status to declined. changing the previous status
const event2 = await EditEventRsvpStatusAction.saveXFromID(viewer, event.id, {
  userID: user.id,
  rsvpStatus: EventRsvpStatusInput.Declined,
});
```

## GraphQL

The following GraphQL schema is generated which uses the above API.

```markdown
type Mutation {
  eventRsvpStatusEdit(input: EventRsvpStatusEditInput!): EventRsvpStatusEditPayload!
}

type EventRsvpStatusEditPayload {
  event: Event!
}

input EventRsvpStatusEditInput {
  eventID: ID!
  rsvpStatus: EventRsvpStatusInput!
  userID: ID!
}

enum EventRsvpStatusInput {
  ATTENDING
  DECLINED
  MAYBE
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
mutation eventRsvpStatusEditMutation($input: EventRsvpStatusEditInput!) {
  eventRsvpStatusEdit(input: $input) {
    event {
      id 
      creator {
        id
      }
    }
  }
}
```
