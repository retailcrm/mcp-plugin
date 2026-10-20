# Lists and bulk selections

Use these instructions when an operation returns a list or connection, the user requests a bulk selection, or the task requires a count, sum, grouping, or another aggregation.

## Build the selection

- Use only pagination arguments and list or connection fields explicitly described in the SDL.
- Do not place terminal entity fields directly on a connection, and do not assume that `nodes`, `edges`, or `pageInfo` exist.
- Limit a single page to no more than 50 entities.
- Request only the minimum required list-item and pagination fields.

## Limit bulk retrieval

Do not circumvent the page limit by sequentially retrieving every page when the user is effectively requesting a bulk export without an explicitly agreed limited range.

Do not execute a separate request for each entity, and do not treat repeated execution of one document as permission for an N+1-style traversal.

If the task allows a limited selection, propose a range of up to 50 entities and continue after the user agrees.

## Perform aggregations on the server

For a count, sum, grouping, or statistics, search for a server-side aggregation operation. Do not retrieve every page or execute one request per entity to calculate an aggregate on the agent side.

If no server-side aggregation exists, report the limitation and offer a limited selection of up to 50 entities. Do not start an N+1-style traversal without explicit agreement.

