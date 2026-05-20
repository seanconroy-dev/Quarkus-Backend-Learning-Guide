---
title: "jdbc + PostgreSQL"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-05-20
type: "lesson"
tags:
    - backend
    - database
    - jdbc
    - postgresql
    - sql
    - quarkus
status: "reviewed"
---

# JDBC + PostgreSQL

## Overview

JDBC (Java Database Connectivity) is the standard Java API for connecting to relational databases and executing SQL statements.

In a Quarkus backend, JDBC with PostgreSQL is used for:

- reading and writing relational data
- executing parameterized SQL safely
- transaction-controlled data changes
- integrating SQL logic with REST endpoints and services

---

## Setting Up PostgreSQL

Install PostgreSQL locally or run it with Docker.

Create a dedicated database and user:

```sql
CREATE DATABASE myappdb;
CREATE USER myappuser WITH ENCRYPTED PASSWORD 'mypassword';
GRANT ALL PRIVILEGES ON DATABASE myappdb TO myappuser;
```

---

## Adding JDBC Dependency

For Quarkus, prefer the Quarkus JDBC extension instead of adding the plain driver manually:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-postgresql</artifactId>
</dependency>
```

Why this is better:

- Quarkus manages compatible driver/runtime integration
- cleaner setup for connection pooling and config
- less manual version management

---

## Quarkus Database Configuration

Configure the datasource in `application.properties`:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/myappdb
quarkus.datasource.username=myappuser
quarkus.datasource.password=mypassword

# Optional pool tuning
quarkus.datasource.jdbc.max-size=20
quarkus.datasource.jdbc.min-size=2
```

Use environment variables for production credentials.

---

## Connecting to PostgreSQL

In Quarkus, inject a `DataSource` and obtain connections from the pool:

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

Using `DataSource` is preferred over direct `DriverManager` usage in Quarkus applications.

---

## Executing SQL Queries

Use `PreparedStatement` for safety and reliability.

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
        String sql = "SELECT id, name FROM users";
        List<String> users = new ArrayList<>();

        try (Connection conn = databaseConnection.getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql);
             ResultSet rs = stmt.executeQuery()) {

            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                users.add(id + ":" + name);
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

Always bind user input as parameters:

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

This prevents SQL injection and avoids fragile string concatenation.

---

## Transactions (Basic)

For write operations involving multiple SQL statements:

```java
try (Connection conn = databaseConnection.getConnection()) {
    conn.setAutoCommit(false);

    // execute multiple statements here

    conn.commit();
} catch (SQLException e) {
    // rollback in real code if needed
    throw new RuntimeException(e);
}
```

In Quarkus services, transactional boundaries are usually handled at the service layer.

---

## Common Mistakes

- using string concatenation instead of `PreparedStatement`
- opening connections manually without closing resources
- hardcoding passwords in source code
- handling SQL exceptions with only `printStackTrace()`
- skipping pool and datasource configuration

---

## Key Concepts Summary

You should be able to explain:

1. what JDBC is and why it matters in Java backends
2. how Quarkus configures PostgreSQL via datasource properties
3. why `DataSource` + pooled connections are preferred
4. why `PreparedStatement` is required for safe SQL
5. where transactions are needed in write operations

---

## Related Notes

- [JDBC + H2](jdbc-h2.md)
- [Hibernate ORM with Panache](hibernate-panache-orm.md)
- [Flyway Database Migrations](flyway-database-migrations.md)

---

## Conclusion

JDBC with PostgreSQL is a core backend skill. In Quarkus, the best default is datasource-based configuration, pooled connections, parameterized SQL, and clean transaction handling. This gives a secure and production-ready foundation for repository and service design.

