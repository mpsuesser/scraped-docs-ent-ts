---
url: https://ent.dev/docs/actions/permissions
title: "Permissions"
description: ""
access_date: 2026-08-03T17:27:01.267Z
current_date: 2026-08-03T17:27:01.267Z
---

To control who can perform an action, a [Privacy Policy](../core-concepts/privacy-policy.md) is used.

The [Default Privacy Policy](action.md#default-privacy-policy) is that any logged in user can perform the action. This doesn't work for all scenarios so naturally, this can be overriden and it's respected *everywhere*.

## Create Action Example

For example, to specify who can [create an event](create-action.md), it can be configured as follows:

```ts
export default class CreateEventAction extends CreateEventActionBase {
  getPrivacyPolicy(): PrivacyPolicy<this> {
    return {
      rules: [
        new AllowIfViewerEqualsRule(this.input.creatorID),
        AlwaysDenyRule,
      ],
    };
  }
}
```

This specifies that the viewer is allowed to create the event assuming the `viewerID` is equal to the `creatorID` in the [input](input.md).

Another way to do this specific scenario is to keep the default privacy policy and to use a [validator](validators.md) to enforce this.

## Edit Action Example

For example, to specify who can [edit an event](edit-action.md), it can be configured as follows:

```ts
export default class EditEventAction extends EditEventActionBase {
  getPrivacyPolicy(): PrivacyPolicy<this> {
    return {
      rules: [new AllowIfViewerIsEntPropertyRule<Event>("creatorID"), AlwaysDenyRule],
    };
  }
}
```

This specifies that the viewer is allowed to edit the event assuming the `viewerID` is equal to the `creatorID` in the ent.

## Delete Action Example

For example, to specify who can [delete an event](delete-action.md), it can be configured as follows:

```ts
export default class DeleteEventAction extends DeleteEventActionBase {
  getPrivacyPolicy(): PrivacyPolicy {
    return {
      rules: [new AllowIfViewerIsEntPropertyRule<Event>("creatorID"), AlwaysDenyRule],
    };
  }
}
```

This specifies that the viewer is allowed to delete the event assuming the `viewerID` is equal to the `creatorID` in the ent.

Note that in this example, the policy to edit and delete is the same. Can be refactored into a common class that's shared especially if it's a complicated policy.

## API

The interface for [PrivacyPolicyRule](../core-concepts/privacy-policy.md#privacypolicyrule) is shown below again:

```ts
interface PrivacyPolicyRule {
  apply(v: Viewer, ent?: Ent): Promise<PrivacyResult>;
}
```

For create actions, a non-privacy checked (unsafe) Ent is passed in as the `ent` parameter. This helps to simplify privacy policies that depend on properties of the ent.

For edit and delete actions, the existing ent is passed as the second parameter.
