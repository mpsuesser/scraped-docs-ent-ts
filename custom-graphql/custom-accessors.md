---
url: https://ent.dev/docs/custom-graphql/custom-accessors
title: "Custom Accessors"
description: ""
access_date: 2026-08-03T18:23:20.955Z
current_date: 2026-08-03T18:23:20.955Z
---

We briefly showed how to add [custom functionality](../core-concepts/ent.md#custom-functionality) in an object but didn't show how to expose it in GraphQL to end users. This explains how to do so.

Given the following schema:

```ts
import { EntSchema, StringType } from "@snowtop/ent"; 
import { EmailType } from "@snowtop/ent-email"; 
import { PasswordType } from "@snowtop/ent-password"; 

const UserSchema = new EntSchema({
  fields: {
    FirstName: StringType(),
    LastName: StringType(),
    EmailAddress: EmailType(),
    Password: PasswordType(),
  }
}); 
export default UserSchema;
```

we'll end with the following GraphQL schema:

```ts
type User implements Node {
  id: ID!
  firstName: String!
  lastName: String!
  emailAddress: String!
}
```

Even after the custom method `howLong` is added below, it's not exposed to the GraphQL schema yet.

```ts
import { UserBase } from "src/ent/internal"; 
import { AlwaysAllowPrivacyPolicy, ID, LoggedOutViewer, PrivacyPolicy } from "@snowtop/ent"
import { Interval } from "luxon"; 

export class User extends UserBase {
  getPRivacyPolicy() {
    return AlwaysAllowPrivacyPolicy;
  }

  howLong() {
    return Interval.fromDateTimes(this.createdAt, new Date()).count('seconds');
  }
}
```

To do so, we'll dive into the Custom GraphQL API.

We use [TypeScript Decorators](https://devblogs.microsoft.com/typescript/announcing-typescript-5-0/#decorators) introduced in TypeScript 5.0 to annotate methods to indicate what we're exposing to GraphQL.

These decorators are evaluated during code generation and they do as little as possible (nothing) otherwise to reduce the overhead of using them.

[gqlField](gql-field.md) is how we annotate the property or method to expose in GraphQL.
