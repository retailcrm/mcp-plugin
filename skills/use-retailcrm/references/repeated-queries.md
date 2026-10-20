# Repeating queries

Use these instructions when one validated query document must be executed multiple times with different variables.

1. Execute the document for one representative entity first.
2. Execute the same document for the remaining variable values only after successful validation and execution.
3. After a validation error, do not repeat the document with other variables. Recheck the accumulated SDL, correct the document, and repeat the mandatory document validation.
4. Call `GqlDescribeOperations` only for an operation that has not yet been described, and `GqlDescribeTypes` only for a newly required type. Do not repeat successful Describe calls.

Do not treat repeated execution as permission for an N+1-style traversal or per-entity aggregation. For bulk processing, read the [list-operation instructions](list-operations.md).

Do not repeat the same read-only document without changing the document or variables.

