---
url: https://ent.dev/docs/custom-graphql/gql-context
title: "Gql Context"
description: ""
access_date: 2026-08-03T19:01:25.904Z
current_date: 2026-08-03T19:01:25.904Z
---

`gqlContextType` annotates a method to indicate that it needs the [RequestContext](../core-concepts/context.md#requestcontext) and the generated GraphQL code should pass it down to the method.

It's required to be the first argument to the method. It can be used in custom [queries](custom-queries.md) or [mutations](custom-mutations.md).

For example:

```ts
export class AuthResolver {
  @gqlMutation({ 
    class: "AuthResolver",
    name: "userAuth", 
    type: UserAuthPayload,
    args: [
      gqlContextType(),
      {
        name: 'input',
        type: 'UserAuthInput',
      },
    ],
  })
  async userAuth(
    context: RequestContext,
    input: UserAuthInput,
  ): Promise<UserAuthPayload> {
    return new UserAuthPayload("1");
  }
}
```

and the generated code looks like:

```ts
export const UserAuthType: GraphQLFieldConfig<
  undefined,
  RequestContext,
  { [input: string]: UserAuthInput }
> = {
  // ...
  resolve: async (
    _source,
    { input },
    context: RequestContext,
    _info: GraphQLResolveInfo,
  ): Promise<UserAuthPayload> => {
    const r = new AuthResolver();
    return r.userAuth(context, {
      emailAddress: input.emailAddress,
      password: input.password,
    });
  },
};
```
