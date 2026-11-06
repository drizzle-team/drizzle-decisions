# CDC event streams report changes to the query result

A CDC event stream belongs to a query with a `where` clause, and its events describe changes to the query result. A row that starts to match is an `insert` with `row`. A row that changes and still matches is an `update` with `row` and `oldRow`. A row that stops matching, or a matching row that is deleted, is a `delete` with `oldRow`. Every field that an event has is a full row.

An update in the table can make a row enter, stay in or leave the query result. Before this decision, all three arrived as `update` with a `matchesNew` flag, and the side that did not match was an empty object. The type promised a full `row` in every case, so `event.row.title` compiled and read `undefined` when the row left the result. Query-relative events remove the empty objects and the flag. They also match what a live collection does with the same changes.

## Considered options

- Keep `update` for every table update and make it a union on `matchesNew`: `row` is the full row when the flag is true and an empty object when it is false. The types would be correct, but every consumer would check the flag before reading `row`, and the empty `oldRow` of a row that enters the result could not be narrowed.

## Consequences

- A consumer cannot tell a table insert from a table update that made a row match. If that is needed, an optional field such as `source: 'insert' | 'update' | 'delete'` can be added later without breaking existing code.
- Columns outside the query never reach the subscriber, because each side of an event is sent only when it matches.
- The HTTP mode keeps its own event format. It is not covered by the stability promise.
