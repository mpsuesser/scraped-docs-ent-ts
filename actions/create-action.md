---
url: https://ent.dev/docs/actions/create-action
title: "Create Action"
description: ""
access_date: 2026-08-03T18:23:20.955Z
current_date: 2026-08-03T18:23:20.955Z
---

This is done via the `ActionOperation.Create` or `ActionOperation.Mutations` [operation](../ent-schema/actions.md#operation).

Based on the [schema](action.md#schema) with the following extra configuration:

```ts
const EventSchema = new EntSchema({
  actions: [
    {
      operation: ActionOperation.Create,
    },
  ], 
}); 
export default EventSchema;
```

leads to 2 classes.

First, the base class:

```ts
export interface EventCreateInput {
  name: string;
  creatorID: ID | Builder<User, Viewer>;
  startTime: Date;
  endTime?: Date | null;
  location: string;
}

export class CreateEventActionBase 
  implements  
  Action<
    Event,
    EventBuilder<EventCreateInput, Event | null>,
    Viewer,
    EventCreateInput,
    Event | null
  > 
{
  public readonly builder: EventBuilder<EventCreateInput, Event | null>;
  public readonly viewer: Viewer;
  protected input: EventCreateInput;

  constructor(viewer: Viewer, input: EventCreateInput) {
    this.viewer = viewer;
    this.input = input;
    this.builder = new EventBuilder(this.viewer, WriteOperation.Insert, this, null);
  }

  getPrivacyPolicy(): PrivacyPolicy {
    return AllowIfViewerHasIdentityPrivacyPolicy;
  }

  ///....
}
```

and then the subclass:

```ts
import {
  CreateEventActionBase, 
  EventCreateInput, 
} from "src/ent/generated/event/actions/create_event_action_base"; 

export { EventCreateInput }; 

export default class CreateEventAction extends CreateEventActionBase {
}
```

The base class `CreateEventActionBase` is where all shared functionality is and will be regenerated as the action configuration changes. It has the default privacy policy plus a bunch of other methods shown below.

The subclass will be generated **once** and any customizations can be applied there.

`EventCreateInput` is an interface that indicates what the input for the action is. What's in there is determined by a combination of the fields in the [schema](action.md#schema) and the [fields](../ent-schema/actions.md#fields) property in the action.

## Usage

```ts
const user = await getUser();

const input: EventCreateInput = {
  name: "fun event",
  creatorID: user.id,
  startTime: startTime,
  location: "location",
};

// creates event and returns the newly created event. throws if it can't be created
const event = await CreateEventAction.create(viewer, input).saveX();

// creates event and returns it. returns null if it can't be created
const event2 = await CreateEventAction.create(viewer, input).save();

// validates that the input is valid to create said event. throws if invalid
const valid = await CreateEventAction.create(viewer, input).validX();

// validates that the input is valid to create said event. returns true or false
const valid2 = await CreateEventAction.create(viewer, input).valid();
```

## GraphQL

The following GraphQL schema is generated which uses the above API.

```markdown
type Mutation {
  eventCreate(input: EventCreateInput!): EventCreatePayload!
}

input EventCreateInput {
  name: String!
  creatorID: ID!
  startTime: Time!
  endTime: Time
  eventLocation: String!
}

type EventCreatePayload {
  event: Event!
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
mutation eventCreateMutation($input: EventCreateInput!) {
  eventCreate(input: $input) {
    event {
      id
      creator {
        id
      }
      name
      startTime
      endTime
      eventLocation
    }
  }
}
```
