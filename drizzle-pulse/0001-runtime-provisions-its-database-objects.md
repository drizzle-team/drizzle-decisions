# The Pulse runtime creates its own database objects

When the runtime starts, it creates every database object that Pulse needs. It sets `REPLICA IDENTITY FULL` on each source table, and it creates the publication and adds every registered source table to it. In HTTP mode, it also creates the events schema (`drizzle_pulse` by default), the events tables and its own state tables. drizzle-kit does not generate or manage any of these objects.

The runtime knows the current set of source tables, so it can bring the database in line on every start. Users do not need a migration step for Pulse, and a new query takes effect on the next start. Only `wal_level=logical` stays a check, because it is a server setting that needs a restart.

Start only adds. Removing a table that no registered query reads, and resetting its replica identity, happens only in provisioning: `runtime.provision()` or the standalone `provision()`. During a rolling deploy, the old and the new version of an application run at the same time with different queries. If start also removed tables, each version would remove the other's tables from the shared publication, and the other version would stop receiving their changes.

## Considered options

- drizzle-kit generates the objects from `pulse(table)` exports in the schema. This was built, then dropped. It tied every Pulse change to a drizzle-kit release and made drizzle-kit load Pulse code from user schemas.
- Users write the migrations by hand from a documented convention. Each change to a query would need a matching manual migration, and a mistake shows up only at runtime.
- Start keeps the publication's table list equal to the registered tables, and a second runtime with other queries makes start throw. A rolling deploy always has two runtimes with different queries for a short time, so it would fail.

## Consequences

- The role that starts the runtime needs the rights to change source tables and create publications. A deploy step can run provisioning under an elevated role instead, so the application role can run without these rights. The standalone `provision({ queries, databaseUrl, wal })` does the same as `runtime.provision()` without a runtime, so the deploy step needs no application database connection.
- A table that no query reads any more stays in the publication until the next provisioning. Until then the runtime decodes its changes and drops them.
- drizzle-kit must not manage the events schema of HTTP mode. drizzle-kit has no default `schemaFilter` and reads every schema, so a project in HTTP mode must set `schemaFilter` without the events schema. Otherwise `push` tries to drop the Pulse tables. The embedded runtime creates no schema and no tables, so this does not apply to it.
- drizzle-kit does not model publications or replica identity, so it does not report the runtime's changes as drift.
