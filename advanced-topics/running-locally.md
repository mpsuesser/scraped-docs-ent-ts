---
url: https://ent.dev/docs/advanced-topics/running-locally
title: "Running Locally"
description: ""
access_date: 2026-08-03T19:01:25.904Z
current_date: 2026-08-03T19:01:25.904Z
---

- TypeScript: all the production code is in TypeScript
- Go: the CLI and orchestration
- Python: For the database migrations. We use the [Alembic](https://alembic.sqlalchemy.org/en/latest/) framework.

Here's the steps to install locally:

- Install [golang](https://golang.org/doc/install#download) version of 1.22.0
- Get the latest version of the Ent CLI:
```shell
go install github.com/lolopinto/ent/tsent@v0.3.6
```
```shell
python3 -m pip install auto_schema==0.0.38
```
- Install the following TypeScript packages globally:
```shell
npm install -g typescript@5.3.2 ts-node@11.0.0-beta.1 @swc/core@1.3.100 @swc/cli@0.1.63 @biomejs/biome@2.4.7
```

If you want generated TypeScript to follow project-specific formatting, add a `biome.json` or `biome.jsonc` file at the project root. `tsent codegen` will use that file automatically; otherwise it falls back to ent's bundled Biome config.

- Install `tsconfig-paths` and `@swc-node/register` locally:
```shell
npm install --save-dev tsconfig-paths@4.2.0 @swc-node/register@1.6.8
```
- Install `rg`

If on OSX, the command is:

```shell
brew install ripgrep
```

For different OSes, check out [github](https://github.com/BurntSushi/ripgrep#installation) for installation instructions.

You should now have everything you need to run locally.

You'd need to check this page regularly in case one of the dependencies has changed.

Check out the [CLI](cli.md) to see what the supported commands are.

When running locally, we have to pass the database connection to the CLI, there are two ways to do this:

## DB\_CONNECTION\_STRING

Env variable `DB_CONNECTION_STRING`. Run as follows:

```shell
DB_CONNECTION_STRING=postgres://ola:@localhost/ent-starter tsent codegen
```

Note the change from `host.docker.internal` in `docker-compose.dev.yml` from the `ent-starter` repository to `localhost`.

## config/database.yml

To specify the database connection parts, can put the details in a yml file: `config/database.yml`

```yml
dialect: postgres
database: ent-starter
user: ola
password: '' 
host: localhost
port: 5432
pool: 5
sslmode: disable
```

Run as follows:

```shell
tsent codegen
```
