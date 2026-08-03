---
url: https://ent.dev/docs/custom-graphql/gql-input-object-type
title: "Gql Input Object Type"
description: ""
access_date: 2026-08-03T19:39:33.704Z
current_date: 2026-08-03T19:39:33.704Z
---

Adds a new input object to the schema. See [example usage](custom-mutations.md#auth-example).

Until [this bug](https://github.com/microsoft/TypeScript/issues/53332) is fixed, custom input objects need to be defined in a separate file from where they're consumed.

Options:

- name

Name of the input object. If not specified, defaults to the name of the class.

- description

Description of the input object. Will be added to the Schema and exposed in tools like [GraphiQL](https://github.com/graphql/graphiql) or [Playground](https://github.com/graphql/graphql-playground).
