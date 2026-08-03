---
url: https://ent.dev/docs/custom-graphql/gql-object-type
title: "Gql Object Type"
description: ""
access_date: 2026-08-03T19:09:20.358Z
current_date: 2026-08-03T19:09:20.358Z
---

Adds a new object to the schema. See example [usage](custom-queries.md#viewer).

Until [this bug](https://github.com/microsoft/TypeScript/issues/53332) is fixed, custom objects need to be defined in a separate file from where they're consumed.

Options:

- name

Name of the object. If not specified, defaults to the name of the class

- description

Description of the object. Will be added to the Schema and exposed in tools like [GraphiQL](https://github.com/graphql/graphiql) or [Playground](https://github.com/graphql/graphql-playground).

- interfaces

[Custom interfaces](gql-interface-type.md) that this object should implement.
