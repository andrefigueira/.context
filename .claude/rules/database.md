# Rule: Database

Rules for data access, models, migrations, and SQL.

## Applies to
`src/models/**`, `src/db/**`, `src/repositories/**`, `migrations/**`, `*.sql`

## Hard rules
1. Parameterized queries only. Never concatenate user input into SQL
2. Use repositories for data access. No direct DB calls from services or handlers
3. Multi-step writes run inside a transaction
4. Migrations are forward-only in production. Write a new migration instead of editing a shipped one
5. Adding a NOT NULL column to an existing table requires a backfill plan in the migration
6. Never drop a column in the same migration as code that stops using it. Ship removal after deploy
7. Indexes: add one when a query needs it; do not add speculative indexes
8. Never log row contents for tables that contain PII

## Patterns

Repository interface (adapt to your stack):

```ts
export interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(input: CreateUser): Promise<User>;
}
```

Transaction wrapper:

```ts
await db.transaction(async (tx) => {
  await tx.users.update(...);
  await tx.audit.log(...);
});
```

## Load on demand
- Full schema and ERD → `.context/database/schema.md`
- Model definitions and validation → `.context/database/models.md`
- Migration strategy → `.context/database/migrations.md`
- Decision record for database choice → `.context/decisions/003-postgresql-database.md`
- Repository pattern rationale → `.context/decisions/002-repository-pattern.md`
