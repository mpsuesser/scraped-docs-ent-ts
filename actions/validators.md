---
url: https://ent.dev/docs/actions/validators
title: "Validators"
description: ""
access_date: 2026-08-03T19:45:08.019Z
current_date: 2026-08-03T19:45:08.019Z
---

Sometimes you need more than [per-field validation](../ent-schema/fields.md#valid) in an action and Validators associated with an action enables that.

```ts
export interface Validator<
  TEnt extends Ent<TViewer>,
  TBuilder extends Builder<TEnt, TViewer, TExistingEnt>,
  TViewer extends Viewer = Viewer,
  TInput extends Data = Data,
  TExistingEnt extends TMaybleNullableEnt<TEnt> = MaybeNull<TEnt>,
> {
  validate(builder: TBuilder, input: TInput): Promise<void> | void;
}
```

Each Validator takes the [Builder](builder.md) and the [Input](input.md).

For example, to validate that the start time is before the end time in the example schema

```ts
export class EventTimeValidator
  implements Validator<Event, EventBuilder, Viewer, EventInput> {
  validate(builder: EventBuilder): void {
    const startTime = builder.getNewStartTimeValue();
    const endTime = builder.getNewEndTimeValue();

    if (!startTime) {
      throw new Error("startTime required");
    }

    if (!endTime) {
      // nothing to do here
      return;
    }

    if (startTime.getTime() > endTime.getTime()) {
      throw new Error("start time cannot be after end time");
    }
  }
}
```

and then update the actions as follows:

```ts
export default class CreateEventAction extends CreateEventActionBase {
  getValidators() {
    return [
      new EventTimeValidator(),
    ];
  }
}
```
```ts
export default class EditEventAction extends EditEventActionBase {
  getValidators() {
    return [
      new EventTimeValidator(),
    ];
  }
}
```

so now when creating or editing an event, we'll ensure that the `startTime` (either new or existing) is before the `endTime`.

```ts
const user = await getUser();

// throws error
await CreateEventAction.create(viewer, {
  name: "fun event",
  creatorID: user.id,
  startTime: new Date('December 25, 2021'),
  endTime: new Date('December 25, 2020'),
  location: "location",
}).saveX();

// success
const event = await CreateEventAction.create(viewer, {
  name: "fun event",
  creatorID: user.id,
  startTime: new Date('December 25, 2021'),
  location: "location",
}).saveX();

// throws!
const event2 = await EditEventAction.create(viewer, event, {
  endTime: new Date('December 25, 2020'),
}).saveX();
```
