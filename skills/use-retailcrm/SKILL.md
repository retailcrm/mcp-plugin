---
name: use-retailcrm
description: "Use retailcrm_mcp to work with the official RetailCRM GraphQL API: search and inspect the schema, then build, validate, and execute arbitrary GraphQL queries and mutations. Apply when reading or changing data in a connected RetailCRM account, including orders, customers, deliveries, payments, products, inventory, users, chats, conversations, messages, quick replies, bots, channels, templates, and other CRM entities."
---

# Use RetailCRM

Communicate with the user in the language of their request.

Work with RetailCRM only through `retailcrm_mcp`. Do not bypass `retailcrm_mcp` with direct HTTP requests or use guessed API methods.

Do not request, expose, or store access tokens.

If `retailcrm_mcp` is unavailable or returns `401`/`403`, explain that its connection or authentication must be configured outside this skill, then stop. Do not request a token or bypass MCP with direct HTTP requests.

## Work with requests

1. Find suitable root operations with `GqlSearchOperations`.
2. Obtain the contracts of the selected operations with `GqlDescribeOperations`.
3. When needed, use `GqlDescribeTypes` to resolve only missing types on paths actually used by the document.
4. Build a minimal GraphQL document using only the retrieved SDL.
5. Validate the entire document against the accumulated SDL.
6. Execute the document with `GqlExecuteOperation` using the same `schemaRef`; execute a mutation only after separate explicit user confirmation.

### 1. Find suitable root operations

Call `GqlSearchOperations` with a short search query based on a stable entity or action term that matches the user's business goal.

- Use `operationType = query` for reads and `operationType = mutation` for changes.
- Build a short search query from the most stable entity or action term.
- Use Search only for root operations. Do not use it to search for nested fields or types.
- For chats, conversations, messages, quick replies, and bots, prefer a discovered operation whose `schemaRef` starts with `node`. If no compatible Node operation exists, select a discovered alternative and explain the choice; do not invent a Node operation.
- Retain the names and signatures of suitable operations and the returned `schemaRef`.
- If the result is empty or ambiguous, refine the search query with domain terms and repeat Search. Do not select a method from memory.

### 2. Obtain the contracts of selected operations

Call `GqlDescribeOperations` before building the document.

- Pass the operation names found in the previous step.
- Pass the same `operationType` and corresponding `schemaRef` returned by Search.
- Start `GqlDescribeOperations` with the minimum depth, `typeDepth: 1`. Retrieve missing types selectively with `GqlDescribeTypes`.
- Inspect arguments, required fields, input types, enums, pagination, and available response fields.
- If a required type is absent from the SDL or appears in `unresolvedTypeNames`, continue to step 3. Do not search for another root operation to obtain a nested type.
- Do not combine operations from different `schemaRef` values in one document.
- Reuse a successful `GqlDescribeOperations` result within the same `schemaRef`; do not repeat an identical call.

### 3. Resolve missing types when needed

Skip this step when the accumulated SDL already contains exact definitions for every type on the paths actually used by the document.

If a required type is absent from the SDL or appears in `unresolvedTypeNames`, use `GqlDescribeTypes`. Before calling it, read the [type-resolution instructions](references/schema-resolution.md).

Request only missing types on paths actually used by the document, and merge the returned SDL with the operation description within the same `schemaRef`. Do not request descriptions for unknown or guessed type names.

If the definition of a required type cannot be obtained, do not guess its structure; return to search.

### 4. Build the GraphQL document

Construct the complete GraphQL document using only Search and Describe results.

If an operation returns a list or connection, read the [list-operation instructions](references/list-operations.md) before building the document.

- Use only discovered operation names, arguments, types, and fields; do not add guessed elements.
- Give the operation a name and move user-supplied values into GraphQL variables when the tool supports a separate `variables` parameter.
- Select only the minimum required fields for each GraphQL object. Request only nested objects and collections that will be used.
- For a complex task, divide the document into independent data blocks. Validate each block independently against the accumulated SDL.
- Combine blocks in one document only when they use the same `schemaRef`, have the same operation type, and do not depend on each other's results.
- Every field with a composite type (object, interface, or union) must have a non-empty selection set containing the required nested fields. Apply this recursively to every nested composite field; do not request an object as a scalar.
- Do not include `__typename`, `__schema`, `__type`, or any other introspection or meta fields. Do not run introspection queries.

### 5. Validate the document before execution

Before every `GqlExecuteOperation` call, validate the final document against the accumulated SDL. Do not call Execute until every item is confirmed:

- the root operation and every supplied argument exist in the SDL, and all required arguments are supplied;
- variable types and used enum values match the SDL;
- every selected field exists on its immediate parent type;
- every field of type object, interface, union, or a list of such objects has a non-empty selection set;
- scalar and enum fields do not have a selection set;
- the document contains no fields whose names start with `__`;
- every used non-built-in type has an exact definition, with no missing or unresolved types on used paths;
- the document contains no guessed operations, arguments, fields, types, or enum values.

If any item cannot be confirmed, do not call Execute. Correct the document using the accumulated SDL; if a required type definition is missing, return to step 3. If a required part of the document cannot be confirmed, report the limitation. Revalidate the entire document after every change.

### 6. Execute the document

After completing step 5, execute a query with `GqlExecuteOperation`, passing `document`, variables, and the same `schemaRef`.

If the same query document must be executed several times with different variables, read the [repeated-query instructions](references/repeated-queries.md) before repeating it.

A mutation requires separate explicit user confirmation. Do not treat the initial request to change data as that confirmation. Before building the document, requesting confirmation, or executing a mutation, read the [mutation instructions](references/mutations.md).

If `GqlExecuteOperation` returns a validation error, do not repeat the same document; return to step 5.

## Handle the result

- Distinguish document, permission, and business-validation errors from a successful empty result.
- Do not claim success if the response contains GraphQL errors.
- After a mutation, state what changed and include stable identifiers only when the response confirms the change.
