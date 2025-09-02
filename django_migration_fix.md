# Fixing Django Migration Inconsistencies in Docker

This document explains the complete process to resolve migration inconsistencies in a Django project running in Docker, without losing data.

---

## Problem Summary

* Deleted migrations in Docker but applied locally.
* Django raises `InconsistentMigrationHistory` errors.
* PostgreSQL already contains tables and constraints from previous migrations.
* Errors like:

  * `django.db.migrations.exceptions.InconsistentMigrationHistory`
  * `django.db.utils.ProgrammingError: relation "unique_currency_code_name" already exists`
  * `django.db.utils.ProgrammingError: relation "chatbot_aimodel" already exists`

The goal is to fix migration history and database schema consistency **without deleting data**.

---

## Step-by-Step Solution

### 1. Check current migration history

Connect to PostgreSQL:

```bash
docker exec -it fba-postgres-container psql -U postgres -d fbadb
```

Check migrations for `chatbot` and `localization`:

```sql
SELECT app, name, applied
FROM django_migrations
WHERE app IN ('chatbot','localization')
ORDER BY applied;
```

### 2. Fix missing initial migrations with `--fake`

Some initial migrations were missing in Docker but tables exist.

```bash
docker compose run --rm --entrypoint "" server python manage.py migrate --fake localization 0001_initial

docker compose run --rm --entrypoint "" server python manage.py migrate --fake chatbot 0001_initial
```

* `--fake` marks the migration as applied without modifying the DB.

### 3. Remove conflicting constraints or indexes

Check if a constraint exists:

```sql
SELECT conname, conrelid::regclass
FROM pg_constraint
WHERE conname = 'unique_currency_code_name';
```

Drop the constraint if it exists:

```sql
ALTER TABLE chatbot_currency DROP CONSTRAINT unique_currency_code_name;
```

For any other conflicting constraint or index, repeat the same process:

```sql
ALTER TABLE <table_name> DROP CONSTRAINT <constraint_name>;
-- or if it's an index
DROP INDEX <index_name>;
```

### 4. Verify the constraints are gone

```sql
SELECT conname, conrelid::regclass
FROM pg_constraint
WHERE conname = '<constraint_name>';
```

### 5. Re-run all migrations normally

```bash
docker compose run --rm --entrypoint "" server python manage.py migrate
```

* Django will now skip already-applied migrations and apply only pending ones.

### 6. Verify migration history

```sql
SELECT app, name, applied FROM django_migrations ORDER BY applied;
```

Ensure all apps have their migrations applied in the correct order.

---

## Notes

* Always **backup your database** before dropping constraints:

```bash
docker exec -t fba-postgres-container pg_dump -U postgres fbadb > backup_fbadb.sql
```

* Use `--fake` **only when the tables or constraints already exist**.
* Never delete tables if you want to preserve data.

---

## Outcome

* Migration inconsistencies are resolved.
* Database schema matches Django migration history.
* No data is lost.
* `chatbot` and `localization` migrations are fully applied and consistent.
