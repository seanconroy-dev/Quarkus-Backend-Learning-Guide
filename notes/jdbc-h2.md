---
title: "jdbc + H2"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-05-20
type: "lesson"
tags:
	- backend
	- database
	- jdbc
	- h2
	- sql
	- quarkus
status: "reviewed"
---

# JDBC + H2

## Overview

H2 is a lightweight relational database often used for local development and testing.

In a Quarkus backend, JDBC + H2 is useful for:

- fast local setup without installing a full database server
- integration tests with isolated schemas
- quick prototyping of repository logic
- learning SQL and JDBC basics with minimal infrastructure

---

## Why H2 in Backend Projects

H2 is commonly used when you want short feedback loops:

- startup is fast
- database files can be local or fully in-memory
- easy reset between test runs
- no external service dependency for basic development

Important:

> H2 behavior can differ from PostgreSQL/MySQL in SQL features and data types.

For production-like validation, always test critical queries against the real target database as well.

---

## Adding JDBC Dependency (Quarkus)

Use the Quarkus H2 JDBC extension:

```xml
<dependency>
	<groupId>io.quarkus</groupId>
	<artifactId>quarkus-jdbc-h2</artifactId>
</dependency>
```

---

## Quarkus Datasource Configuration

### Option A: In-Memory H2 (good for tests and quick demos)

```properties
quarkus.datasource.db-kind=h2
quarkus.datasource.jdbc.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1;MODE=PostgreSQL
quarkus.datasource.username=sa
quarkus.datasource.password=sa
```

`MODE=PostgreSQL` improves compatibility for learning and migration scenarios.

### Option B: File-Based H2 (data persists locally)

```properties
quarkus.datasource.db-kind=h2
quarkus.datasource.jdbc.url=jdbc:h2:file:./data/myappdb;MODE=PostgreSQL;AUTO_SERVER=TRUE
quarkus.datasource.username=sa
quarkus.datasource.password=sa
```

Use file-based mode when you want data to survive restarts during local development.

---

## Connecting with JDBC in Quarkus

In Quarkus, use `DataSource` injection and pooled connections.

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.inject.Inject;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@ApplicationScoped
public class DatabaseConnection {

	@Inject
	DataSource dataSource;

	public Connection getConnection() throws SQLException {
		return dataSource.getConnection();
	}
}
```

---

## Executing SQL Queries

Use `PreparedStatement` as default.

```java
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

public class UserRepository {

	private final DatabaseConnection databaseConnection;

	public UserRepository(DatabaseConnection databaseConnection) {
		this.databaseConnection = databaseConnection;
	}

	public List<String> getUsers() {
		String sql = "SELECT id, name FROM users ORDER BY id";
		List<String> users = new ArrayList<>();

		try (Connection conn = databaseConnection.getConnection();
			 PreparedStatement stmt = conn.prepareStatement(sql);
			 ResultSet rs = stmt.executeQuery()) {

			while (rs.next()) {
				users.add(rs.getInt("id") + ":" + rs.getString("name"));
			}
		} catch (SQLException e) {
			throw new RuntimeException("Failed to fetch users", e);
		}

		return users;
	}
}
```

---

## Parameterized Query Example

Never build SQL with string concatenation for user input.

```java
String sql = "SELECT id, name FROM users WHERE email = ?";

try (Connection conn = databaseConnection.getConnection();
	 PreparedStatement stmt = conn.prepareStatement(sql)) {

	stmt.setString(1, email);

	try (ResultSet rs = stmt.executeQuery()) {
		// read result
	}
}
```

---

## Basic Schema Example for H2

```sql
CREATE TABLE users (
	id INT PRIMARY KEY,
	name VARCHAR(120) NOT NULL,
	email VARCHAR(255) NOT NULL UNIQUE
);

INSERT INTO users (id, name, email)
VALUES
	(1, 'Alice', 'alice@example.com'),
	(2, 'Bob', 'bob@example.com');
```

---

## Transactions (Basic)

For multiple dependent writes, use one transaction.

```java
try (Connection conn = databaseConnection.getConnection()) {
	conn.setAutoCommit(false);

	// execute multiple INSERT/UPDATE statements

	conn.commit();
} catch (SQLException e) {
	throw new RuntimeException("Transaction failed", e);
}
```

---

## H2 vs PostgreSQL (Backend Perspective)

| Topic | H2 | PostgreSQL |
|---|---|---|
| Setup effort | Very low | Higher |
| Runtime footprint | Very small | Larger |
| SQL compatibility | Good, but not identical | Real production behavior |
| Best use | Local dev/tests, quick prototypes | Production and realistic integration testing |

Recommendation:

- Use H2 for fast local iteration.
- Validate critical SQL, constraints, and migrations with PostgreSQL before release.

---

## Common Mistakes

- assuming H2 SQL behavior is always identical to PostgreSQL
- skipping tests against the real production database
- using string concatenation instead of `PreparedStatement`
- hardcoding credentials in code
- forgetting to reset in-memory data between tests when isolation is needed

---

## Key Concepts Summary

You should be able to explain:

1. why H2 is useful for local backend development and tests
2. how to configure H2 datasource settings in Quarkus
3. when to use in-memory vs file-based H2
4. why `PreparedStatement` is required for safe SQL
5. why production-like verification still needs PostgreSQL tests

---

## Related Notes

- [JDBC + PostgreSQL](jdbc-PostgresSQL.md)
- [Hibernate ORM with Panache](hibernate-panache-orm.md)
- [Flyway Database Migrations](flyway-database-migrations.md)

---

## Conclusion

JDBC + H2 is ideal for fast local backend development in Quarkus. It reduces setup complexity while preserving core JDBC patterns: datasource configuration, pooled connections, parameterized SQL, and transaction handling. Use it as a productivity tool, then verify critical behavior against PostgreSQL for production confidence.
