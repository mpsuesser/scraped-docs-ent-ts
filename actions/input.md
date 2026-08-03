---
url: https://ent.dev/docs/actions/input
title: "Input"
description: ""
access_date: 2026-08-03T19:09:20.358Z
current_date: 2026-08-03T19:09:20.358Z
---

Most Actions have an Input associated with them. This is generated based on how the [action](../ent-schema/actions.md#fields) is [configured](../ent-schema/actions.md#actiononlyfields).

## Action Input

The Input is usually a subset of all the fields exposed in the schema for the Ent although it can include other fields if there are [action only fields](action-only-fields.md).

For example, in the [create action](create-action.md), the input looks as follows:

```ts
export interface EventCreateInput {
  name: string;
  creatorID: ID | Builder<User, Viewer>;
  startTime: Date;
  endTime?: Date | null;
  location: string;
}
```

but in the [edit action](edit-action.md), it looks slightly different with each field optional:

```ts
export interface EventEditInput {
  name?: string;
  creatorID?: ID | Builder<User, Viewer>;
  startTime?: Date;
  endTime?: Date | null;
  location?: string;
}
```

This input is passed as the second argument to [Triggers](triggers.md), [Validators](validators.md), and [Observers](observers.md). This enables them to be strongly typed when needed e.g. in the [observer example](observers.md#example).

## Builder Input

Each [generated Builder](builder.md#generated-builder) has an associated Input which the Action Input is a subset of.

The Builder Input is the source of truth and any updates (in Triggers) is done via the `updateInput` method.

For example, in a confirm email action, can update the `emailVerified` field as follows:

```ts
export default ConfirmEmailAction extends ConfirmEmailActionBase {

  getTriggers() {
    return [
      {
        changeset(builder: UserBuilder<ConfirmEmailInput, Viewer>, input: ConfirmEmailInput) {
          builder.updateInput({
            emailVerified:true,
          });
        },
      },
    ];
  }
}
```
