# Executing mutations

Use these instructions before building a mutation document, requesting confirmation, and executing the mutation.

- Request the minimum response sufficient to confirm the change.
- Do not perform adjacent changes that the user did not request and confirm.

1. Describe in natural language the specific change, target entities, and material values.
2. Ask a direct confirmation question, for example: “Order 123 will be changed to the Completed status. Apply this change?”
3. Wait for separate explicit user confirmation. Do not treat the original request to make the change as confirmation.
4. If the user changes any parameters, rebuild the document, repeat the mandatory validation in step 5, and request confirmation again.
5. Call `GqlExecuteOperation` with the same `schemaRef` only after confirmation.

A complete GraphQL document does not replace clear business-level confirmation. Do not execute a mutation merely to validate the document: after successful validation, `GqlExecuteOperation` may apply the change immediately.

