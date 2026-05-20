---
title: "Flyway Database Migrations"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-05-20
type: "lesson"
tags:
    - backend
    - quarkus
    - database
    - flyway
    - migrations
    - sql
    - postgresql
status: "reviewed"
---

# Flyway Database Migrations

## Overview

Flyway is a database migration tool that applies versioned SQL scripts in a controlled, repeatable order.

In Quarkus backends, Flyway is used to:

- create and evolve database schema safely
- keep schema changes consistent across team members and environments
- avoid manual database drift between local, test, and production
- make deployments predictable and auditable

---

## Why Flyway Matters in Backend Projects

Without migrations, schema changes are often applied manually, which creates risk:

- different developers run different SQL steps
- test and production schema can drift apart
- rollback and troubleshooting become difficult

With Flyway:

- schema changes are versioned in source control
- every environment runs the same ordered scripts
- migration history is tracked in the database (`flyway_schema_history`)

---

## Quarkus Dependency

Add Flyway support:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-flyway</artifactId>
</dependency>
```

Use it together with your JDBC driver extension, for example PostgreSQL:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-postgresql</artifactId>
</dependency>
```

---

## Basic Quarkus Configuration

Typical `application.properties` setup:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/myappdb
quarkus.datasource.username=myappuser
quarkus.datasource.password=mypassword

quarkus.flyway.migrate-at-start=true
quarkus.flyway.locations=db/migration

# Recommended when using Flyway for schema control
quarkus.hibernate-orm.database.generation=none
```

Important:

> If Flyway manages schema evolution, disable automatic schema generation by Hibernate.

---

## Migration File Naming Convention

Flyway versioned SQL scripts should follow this format:

```text
V<version>__<description>.sql
```

Examples:

```text
V1__create_users_table.sql
V2__add_email_unique_constraint.sql
V3__create_orders_table.sql
```

Rules:

- `V` indicates a versioned migration
- versions must be unique and increasing
- double underscore `__` separates version and description
- descriptions should be short and explicit

---

## Folder Structure

By default in Quarkus:

```text
src/main/resources/db/migration/
```

Example:

```text
src/main/resources/db/migration/
  V1__create_users_table.sql
  V2__add_user_status_column.sql
  V3__create_orders_table.sql
```

---

## Example Migration Scripts

### V1: Create Base Table

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) NOT NULL,
    name VARCHAR(120) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

### V2: Add Unique Constraint

```sql
ALTER TABLE users
ADD CONSTRAINT uq_users_email UNIQUE (email);
```

### V3: Add New Table

```sql
CREATE TABLE orders (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    total_amount NUMERIC(10, 2) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT fk_orders_user FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## How Flyway Runs

At application start (with `migrate-at-start=true`):

1. Flyway checks `flyway_schema_history`
2. It detects already applied versions
3. It applies new migration files in version order
4. It records success/failure metadata

If one migration fails:

- startup fails
- later migrations are not executed
- you must fix the migration state before continuing

---

## Development Workflow

Recommended workflow for schema changes:

1. create a new migration file with the next version
2. write deterministic SQL (no environment-specific assumptions)
3. run the app locally and verify migration applies
4. run tests
5. commit migration script with application code changes

Never edit old already-applied migration files in shared environments.

---

## Rollback Strategy (Practical)

Flyway Community Edition does not provide automatic rollback for versioned SQL migrations.

Practical approach:

- write forward-only migrations
- if needed, add a new migration that compensates/fixes the previous one
- for risky changes, test with backup/restore strategy in staging

Example:

- bad migration: `V5__drop_legacy_column.sql`
- fix migration: `V6__recreate_legacy_column.sql`

---

## Flyway + Quarkus + Hibernate ORM

Good default for production-style setups:

- Flyway handles schema changes
- Hibernate ORM handles entity mapping and runtime persistence
- Hibernate auto DDL generation stays disabled (`generation=none`)

This avoids conflicting schema management strategies.

---

## Common Mistakes

- mixing Flyway migrations with Hibernate auto schema generation
- renaming or editing already executed migration files
- using non-deterministic SQL in migrations
- skipping migration tests before merge/deploy
- putting large data fixes in schema migration scripts without performance checks

---

## Flyway vs Manual SQL Changes

| Topic | Manual SQL Changes | Flyway |
|---|---|---|
| Repeatability | Low | High |
| Team consistency | Low | High |
| Auditability | Weak | Strong |
| Deployment safety | Risky | Safer |
| Roll-forward strategy | Ad hoc | Structured |

---

## Key Concepts Summary

You should be able to explain:

1. what Flyway is and why backend teams use it
2. how migration versioning and naming work
3. where migration files are stored in Quarkus
4. why Hibernate schema auto-generation should be disabled with Flyway
5. how to handle failed or incorrect migrations in practice

---

## Related Notes

- [JDBC + PostgreSQL](jdbc-PostgresSQL.md)
- [JDBC + H2](jdbc-h2.md)
- [Hibernate ORM with Panache](hibernate-panache-orm.md)

---

## Conclusion

Flyway gives Quarkus backend projects controlled schema evolution through versioned SQL. It reduces database drift, improves deployment reliability, and makes schema changes traceable. For stable environments, use Flyway as the single source of truth for schema changes and keep migrations forward-only and testable.
