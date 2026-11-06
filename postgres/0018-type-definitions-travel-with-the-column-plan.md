# Type definitions travel with the column plan

A column declared with a `defineType()` marker, or as `vector`, `halfvec` or `sparsevec`, carries its type definition on the column plan. Its `oid` is `0`, so the driver fills in the real OID from the result description, the same way as for an `'unknown'` column. The decoder reads the definition from the column. `oid` therefore always means the database OID, and no shape column uses a made-up negative number.

A project can load two copies of the driver: the standalone package and the copy bundled in `drizzle-orm`. Before this decision, `defineType()` took a negative number from a module-level counter and stored the definition in a module-level map, and the column plan kept only the number. A marker built by one copy found nothing in the other copy's map, so `geometry` values arrived as raw EWKB hex. Both counters started at -2000, so a number could also mean a different type in each copy.

## Considered options

- Share the map and the counter between copies through `globalThis` and `Symbol.for`, as `brand.ts` does for classes. This works, but two driver versions would then read each other's definition objects, and the shape of `CustomTypeDef` would become a contract between versions.
- Read extension OIDs from the database catalog. The driver does not introspect the database. It uses only the OIDs that a result description reports.

## Consequences

- The mapper cache key needs a token for a definition. Each copy numbers the definitions it sees in a module-level `WeakMap`. The ids never leave that copy, because the cache that compares them lives in the same copy.
- A declared type wins over a `config.types` override for the same real OID.
- Replication decodes these columns as text, because their targets parse text.
- `CustomTypeDef.oid`, `EXT_VECTOR`, `EXT_HALFVEC` and `EXT_SPARSEVEC` are removed from the public API.
- Module-level state must not be keyed by values that one copy creates and another copy reads. `brand.ts` and the `Transform()` id counter in `json.ts` share their state through `globalThis` for this reason.
- The HTTP clients cache compiled row mappers per client, with the same key function as a connection.
