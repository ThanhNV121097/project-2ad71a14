# ERD — hello-word-10

## Scope

Database stores one greeting row used by page render. No users, sessions, audit log, or edit history.

## Tables

### `greetings`

| Column | Type | Null | Default | Key | Notes |
|---|---:|---|---|---|---|
| `id` | `integer` | no | none | primary key | Must be `1`; singleton row constraint keeps exactly one logical greeting |
| `text` | `text` | no | none |  | Displayed greeting copy; must be non-empty after trim |
| `created_at` | `timestamptz` | no | `now()` |  | Insert time |
| `updated_at` | `timestamptz` | no | `now()` |  | Last update time; no update flow in initial scope |

Constraints:

- `PRIMARY KEY (id)`.
- `CHECK (id = 1)` enforces singleton identity.
- `CHECK (length(btrim(text)) > 0)` rejects empty greeting text.

Seed data:

```sql
INSERT INTO greetings (id, text) VALUES (1, 'Hello Word')
ON CONFLICT (id) DO NOTHING;
```

## Relationships

None. Project has one table.

## Migration ownership

Backend applies migrations on startup before listening. Migration state lives in `schema_migrations`, created by migrator code.

## Rejected alternatives

| Alternative | Rejected because |
|---|---|
| Store greeting in environment variable | Violates requirement that text is stored in PostgreSQL |
| Allow multiple greeting rows with `active` flag | Extra state not needed for one proof page |
| UUID primary key | Singleton row is simpler and easier to verify |
