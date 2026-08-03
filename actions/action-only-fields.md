---
url: https://ent.dev/docs/actions/action-only-fields
title: "Action Only Fields"
description: ""
access_date: 2026-08-03T19:01:25.904Z
current_date: 2026-08-03T19:01:25.904Z
---

Allows [configuring](../ent-schema/actions.md#actiononlyfields) other fields to be added in the [input](input.md#action-input) of an Action.

In the [address example](triggers.md#changeset), with the Event schema configured as follows:

```ts
const EventSchema = new EntSchema({
  actions: [
    {
      operation: ActionOperation.Create,
      actionOnlyFields: [
        {
          name: "address",
          type: "Object",
          nullable: true,
          actionName: "CreateAddressAction",
        },
      ],
    },
  ], 
}); 
export default EventSchema;
```

and the following `Address` schema:

```ts
const AddressSchema = new EntSchema({
  fields: {
    Street: StringType(),
    City: StringType(),
    State: StringType(),
    ZipCode: StringType(),
    Apartment: StringType({ nullable: true }),
    OwnerID: UUIDType({
      index: true, 
      polymorphic: {
        types: [NodeType.Event],
      }
    }),
  },

  actions: [
    {
      operation: ActionOperation.Create,
    },
  ],
});
export default AddressSchema
```

we end up with the following changes:

```ts
interface customAddressInput {
  street: string; 
  city: string; 
  state: string; 
  zipCode: string; 
  apartment?: string | null; 
}

export interface EventCreateInput {
  name: string; 
  creatorID: ID | Builder<User, Viewer>; 
  startTime: Date; 
  endTime?: Date | null; 
  location: string; 
  address?: customAddressInput | null; 
}
```
```graphql
input EventCreateInput {
  name: String!
  creatorID: ID!
  startTime: Time!
  endTime: Time
  eventLocation: String!
  address: AddressEventCreateInput
}

input AddressEventCreateInput {
  street: String!
  city: String!
  state: String!
  zipCode: String!
  apartment: String
}
```

and used as follows:

```ts
// creates event and associated address at once
const event = await CreateEventAction.create(vc, {
  name: "fun event",
  creatorID: user.id,
  startTime: yesterday,
  endTime: now,
  location: "location",
  address: {
    streetName: "1 Dr Carlton B Goodlett Pl",
    city: "San Francisco",
    state: "CA",
    zip: "94102",
  },
}).saveX();
```

Can be called via the GraphQL API similarly.

Note that nothing is automatically done with action only fields. It's up to the developer to use [Triggers](triggers.md) to actually process and use them effectively.

It's up to the developer's creativity to find other use cases than what's expressed here.
