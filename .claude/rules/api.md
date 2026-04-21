# Rule: API

Rules for HTTP API endpoints, handlers, and route definitions.

## Applies to
`src/api/**`, `src/routes/**`, `src/handlers/**`, `app/api/**`

## Hard rules
1. Validate all input at the handler boundary before it reaches services
2. Return typed error objects, never raw strings
3. Use the project's error catalog. Do not invent new error codes inline
4. HTTP status codes: 2xx success, 4xx client error, 5xx server error. No 200 with an error body
5. Handlers contain no business logic. Delegate to services
6. Every endpoint logs request ID, user ID (if authenticated), and operation name
7. Breaking changes require a version bump. Load versioning rules on demand

## Patterns

Handler shape (adapt to your stack):

```ts
export async function handler(req: Request): Promise<Response> {
  const input = schema.parse(await req.json());           // validate
  const result = await service.execute(input, ctx);        // delegate
  return Response.json(result, { status: 200 });           // shape
}
```

## Load on demand
- Full endpoint reference → `.context/api/endpoints.md`
- Request / response examples → `.context/api/examples.md`
- Error codes catalog → `.context/errors.md`
- Versioning policy → `.context/versioning.md`
