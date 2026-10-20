# Resolving missing GraphQL types

Use these instructions when a required type is absent from the accumulated SDL or appears in `unresolvedTypeNames`.

## Identify required types

Treat `schemaTruncated: true` as an indication that some reachable types were omitted from the SDL because of the `typeDepth` limit. This does not prevent document construction when every type on the paths actually used by the document is already described.

Before building the document, identify types used:

- in root-operation arguments and supplied input objects;
- by selected response fields;
- in selection sets of composite types;
- in inline fragments for interfaces and unions.

Do not automatically resolve every value in `unresolvedTypeNames`. A type that does not appear on a used path needs no additional description.

## Accumulate and reuse SDL

Treat successful Describe results as immutable within one `schemaRef` and reuse them throughout the current task. Maintain accumulated SDL separately for each `schemaRef`; do not carry definitions across schema versions.

After a successful `GqlDescribeOperations` call, do not repeat a call with the same:

- `schemaRef`;
- `operationType`;
- set of `operationNames`, regardless of name order;
- `typeDepth`.

Normalize the name set when checking for duplicates: remove duplicates and compare sorted values. If another operation is required, pass only `operationNames` that have not yet been described; do not include names already retrieved for the same `schemaRef` and `operationType`.

Do not repeat a successful `GqlDescribeTypes` call with the same `schemaRef`, source-operation context, set of `typeNames` regardless of order, and `typeDepth`. Normalize `typeNames` in the same way as `operationNames`.

A Describe call may be repeated only when the previous call failed technically and returned no result, the `schemaRef` changed, or a new operation or newly required type is being requested.

## Retrieve missing types

Build the required schema incrementally: first obtain result and argument type definitions through `GqlDescribeOperations`, then request only missing types on paths actually used by the document.

Before calling `GqlDescribeTypes`, exclude from `typeNames`:

- types whose exact definitions already exist in the accumulated SDL;
- types not located on used paths;
- types added only because they appear in the overall `unresolvedTypeNames`.

If a required non-built-in type appears in `unresolvedTypeNames` or lacks an exact definition in the accumulated SDL:

1. Call `GqlDescribeTypes`.
2. Pass the same `schemaRef` and the name and type of the original root operation.
3. In `typeNames`, pass the exact names of only required types whose definitions are still absent from the accumulated SDL.
4. Use `typeDepth: 1`. If a required child type remains in `unresolvedTypeNames`, describe that child type directly in the next call instead of repeating the already described parent with a greater depth.
5. Merge the returned SDL with the original operation description and recheck the used paths.

`GqlDescribeTypes` returns one combined SDL for all supplied `typeNames`; shared types and overlapping subtrees are deduplicated. If one requested type is reachable from another, still pass both names when both are needed as independent traversal roots: the depth is applied independently to each.

If the new result contains `unresolvedTypeNames`, describe only child types not yet retrieved that occur on used paths. If the parent definition is already known, request the type of its required field instead of requesting the parent again.

Do not use `GqlSearchOperations` to search for types, and do not search for a different root operation solely to determine a nested type.

## Verify definition completeness

Treat a type as described only when the required name exactly matches a declaration name in the accumulated SDL. Do not use prefix or substring matches. Treat only the built-in GraphQL scalar types `Int`, `Float`, `String`, `Boolean`, and `ID` as defined without a separate declaration.

If the definition of a required type cannot be obtained:

- omit the corresponding block when it is optional for the task;
- if the block is required, report the limitation and do not execute the document;
- do not guess the type kind, fields, enum values, input structure, or serialization.

