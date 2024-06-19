# wrangler-d1-execute-potential-defect

Minimal repro of potential defect when running wrangler d1 execute with --file option since 3.57.

## Overview

1. Init the environment.
2. Init the database.
3. Test for expected results (the results of the query).
4. Update `wrangler` version to `3.57`+.
5. Test for incorrect results (the metadata of the query).

## Steps

### Init the environment

`mv wrangler.toml.template wrangler.toml`

Change line 1, adding your Cloudflare `account_id`.

e.g. `account_id = ""` -> `account_id = "put-your-account-id-in here"` 

If you're not using `pnpm`, replace the `pnpm` commands with the appropriate commands from your chosen package manager, such as `npm`, `yarn` etc.

```
git clone git@github.com:smcstewart/wrangler-d1-execute-potential-defect.git
cd wrangler-d1-execute-potential-defect
pnpm install
```

> The environment setup here is for `dev`, but if you've called your environment something different, remember to replace that in the commands below and also in your `wrangler.toml` file.

### Init the database

```
pnpm exec wrangler d1 create wrangler-d1-execute-potential-defect --env dev
```

Change line 11, adding the ID of your database that was the result from the command above.
e.g. `{ binding = "DB", database_name = "wrangler-d1-execute-potential-defect", database_id = "" }` -> `{ binding = "DB", database_name = "wrangler-d1-execute-potential-defect", database_id = "put-your-database-id-in-here" }` 

```
pnpm exec wrangler d1 execute wrangler-d1-execute-potential-defect --command "CREATE TABLE IF NOT EXISTS xyz (id text PRIMARY KEY, name text NOT NULL)" --remote --env dev
pnpm exec wrangler d1 execute wrangler-d1-execute-potential-defect --command "INSERT INTO xyz VALUES ('001', 'John Smith'), ('002', 'Jenny Anderson'), ('003', 'Ann Jones'), ('004', 'William McBappe')" --remote --env dev
```

### Test for expected results

```
pnpm exec wrangler d1 execute wrangler-d1-execute-potential-defect --file "./test.sql" --remote --env dev --yes
```

You will see output similar to:

```
 ⛅️ wrangler 3.56.0 (update available 3.61.0)
-------------------------------------------------------
🌀 Mapping SQL input into an array of statements
🌀 Parsing 1 statements
🌀 Executing on remote database wrangler-d1-execute-potential-defect (database-id-will-show-here):
🌀 To execute on your local development database, remove the --remote flag from your wrangler command.
🚣 Executed 1 commands in 0.191ms
┌─────┬─────────────────┐
│ id  │ name            │
├─────┼─────────────────┤
│ 001 │ John Smith      │
├─────┼─────────────────┤
│ 002 │ Jenny Anderson  │
├─────┼─────────────────┤
│ 003 │ Ann Jones       │
├─────┼─────────────────┤
│ 004 │ William McBappe │
└─────┴─────────────────┘
```

### Change `wrangler` version to `3.57`

Change line 18 in the `package.json` file, updating the version of `wrangler` to `3.57`.
i.e. `"wrangler": "3.56.0"` -> `"wrangler": "3.57.0"`

### Test for incorrect results

```
pnpm install
pnpm exec wrangler d1 execute wrangler-d1-execute-potential-defect --file "./test.sql" --remote --env dev --yes
```

You will now see output similar to:

```
 ⛅️ wrangler 3.57.0 (update available 3.61.0)
-------------------------------------------------------
▲ [WARNING] ⚠️ This process may take some time, during which your D1 database will be unavailable to serve queries.


🌀 Executing on remote database wrangler-d1-execute-potential-defect (database-id-will-show-here):
🌀 To execute on your local development database, remove the --remote flag from your wrangler command.
Note: if the execution fails to complete, your DB will return to its original state and you can safely retry.
├ 🌀 Uploading 4be003e2-5785-4e2f-86a0-db3d2e1bfa8a.46a9e065997e9de0.sql 
│ 🌀 Uploading complete.
│ 
🌀 Starting import...
🌀 Processed 1 queries.
🚣 Executed 1 queries in 0.00 seconds (1 rows read, 0 rows written)
   Database is currently at bookmark 00000002-00000000-00002de5-cbc221a9f0d27df13a737029559e8319.
┌────────────────────────┬───────────┬──────────────┬───────────────────┐
│ Total queries executed │ Rows read │ Rows written │ Databas size (MB) │
├────────────────────────┼───────────┼──────────────┼───────────────────┤
│ 1                      │ 1         │ 0            │ 0.02              │
└────────────────────────┴───────────┴──────────────┴───────────────────┘
```
