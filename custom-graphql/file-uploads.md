---
url: https://ent.dev/docs/custom-graphql/file-uploads
title: "File Uploads"
description: ""
access_date: 2026-08-03T18:13:08.120Z
current_date: 2026-08-03T18:13:08.120Z
---

Uploading files is a common of part of applications and we need a way to support that.

## gqlFileUpload

```ts
const gqlFileUpload: CustomType = {
  type: "GraphQLUpload",
  importPath: "@snowtop/ent/graphql/upload",
  tsType: "FileUpload",
  tsImportPath: "@snowtop/ent/graphql/upload",
};
```

`gqlFileUpload` uses the `@snowtop/ent/graphql/upload` wrapper around the optional [graphql-upload](https://www.npmjs.com/package/graphql-upload) package to support File Uploads. It's also a good example of [CustomType](gql-field.md#customtype) usage.

Here's an example usage:

```ts
import type { FileUpload } from "@snowtop/ent/graphql/upload";

export class ImportGuestResolver {
  @gqlMutation({ 
    class: "ImportGuestResolver",
    async: true,
    type: Event,
    args: [
      gqlContextType(),
      {
        name: "eventID",
        type: GraphQLID,
      },
      {
        name: "file",
        type: gqlFileUpload,
      },
    ],
  })
  async importGuests(
    context: RequestContext,
    eventID: ID,
    file: Promise<FileUpload>,
  ) {
  }
}
```

leads to this schema:

```graphql
type Mutation {
  importGuests(eventID: ID!, file: Upload!): Event!
}
```

Note this requires the developer to manually run `npm install graphql-upload@15.0.2` to use uploads.
