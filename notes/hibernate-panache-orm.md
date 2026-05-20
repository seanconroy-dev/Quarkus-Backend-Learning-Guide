---
title: "Hibernate ORM with Panache"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-05-20
type: "lesson"
tags:
    - backend
    - quarkus
    - hibernate
    - panache
    - orm
    - jpa
    - database
    - rest
status: "reviewed"
---

# Hibernate ORM with Panache

## Overview

Hibernate ORM maps Java objects to relational database tables.

In Quarkus, Panache is a productivity layer on top of Hibernate ORM that reduces boilerplate for common persistence tasks.

Use Panache when you want:

- faster CRUD development
- concise entity and repository code
- readable query methods
- less manual `EntityManager` handling

---

## What Panache Adds

Without Panache, standard JPA often needs:

- verbose repositories
- manual query creation
- repetitive persistence code

With Panache, you get:

- built-in active record or repository patterns
- helper methods like `persist()`, `findById()`, `list()`, `deleteById()`
- compact query expressions with parameters

---

## Dependency

Add Panache for Hibernate ORM in Quarkus:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-orm-panache</artifactId>
</dependency>
```

For PostgreSQL projects, combine with the JDBC extension:

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-jdbc-postgresql</artifactId>
</dependency>
```

---

## Basic Configuration

Typical `application.properties` setup:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/myappdb
quarkus.datasource.username=myappuser
quarkus.datasource.password=mypassword

quarkus.hibernate-orm.database.generation=update
quarkus.hibernate-orm.log.sql=true
```

For production, avoid `update` and use controlled migrations.

---

## Entity with Panache (Active Record Style)

```java
import io.quarkus.hibernate.orm.panache.PanacheEntity;
import jakarta.persistence.Column;
import jakarta.persistence.Entity;

@Entity
public class Product extends PanacheEntity {

    @Column(nullable = false)
    public String name;

    @Column(nullable = false)
    public double price;

    public String category;
}
```

`PanacheEntity` already includes an auto-generated `Long id`.

---

## CRUD Example (Active Record)

```java
import jakarta.transaction.Transactional;
import jakarta.ws.rs.*;
import jakarta.ws.rs.core.MediaType;
import java.util.List;

@Path("/api/products")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ProductResource {

    @GET
    public List<Product> getAll() {
        return Product.listAll();
    }

    @GET
    @Path("/{id}")
    public Product getById(@PathParam("id") Long id) {
        Product product = Product.findById(id);
        if (product == null) {
            throw new NotFoundException("Product not found");
        }
        return product;
    }

    @POST
    @Transactional
    public Product create(Product product) {
        product.persist();
        return product;
    }

    @DELETE
    @Path("/{id}")
    @Transactional
    public void delete(@PathParam("id") Long id) {
        boolean deleted = Product.deleteById(id);
        if (!deleted) {
            throw new NotFoundException("Product not found");
        }
    }
}
```

`@Transactional` is required for write operations.

---

## Repository Style (PanacheRepository)

If you prefer clean separation between entity and data access, use repository style.

```java
import io.quarkus.hibernate.orm.panache.PanacheRepository;
import jakarta.enterprise.context.ApplicationScoped;
import java.util.List;

@ApplicationScoped
public class ProductRepository implements PanacheRepository<Product> {

    public List<Product> findByCategory(String category) {
        return find("category", category).list();
    }

    public List<Product> findExpensive(double minPrice) {
        return find("price > ?1", minPrice).list();
    }
}
```

Use this style when you want service-layer architecture and easier testability.

---

## Query Patterns

Panache supports concise query options:

```java
Product.find("category", "books").list();
Product.find("price > ?1 and category = ?2", 20.0, "tech").list();
Product.find("order by price desc").page(0, 10).list();
long count = Product.count("category", "books");
```

For complex reporting queries, standard JPA or native SQL can still be used.

---

## Transactions and Service Layer

Common backend pattern:

- Resource handles HTTP and validation basics
- Service handles business logic and transactions
- Repository handles persistence logic

Example transactional service:

```java
import jakarta.enterprise.context.ApplicationScoped;
import jakarta.transaction.Transactional;

@ApplicationScoped
public class ProductService {

    @Transactional
    public Product createProduct(String name, double price, String category) {
        Product p = new Product();
        p.name = name;
        p.price = price;
        p.category = category;
        p.persist();
        return p;
    }
}
```

---

## Panache and Validation

Combine Panache with Bean Validation in request DTOs:

```java
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Positive;

public record CreateProductRequest(
    @NotBlank String name,
    @Positive double price,
    String category
) {}
```

Validate at API boundaries, then map DTOs to entities.

---

## Common Mistakes

- returning entities directly in all APIs without considering DTO boundaries
- forgetting `@Transactional` on create/update/delete operations
- putting business rules directly in REST resources
- overusing active record in large projects where repository/service separation is clearer
- using `database.generation=update` in production

---

## Panache vs Plain JPA (Quick Comparison)

| Topic | Plain JPA/Hibernate | Panache |
|---|---|---|
| Boilerplate | Higher | Lower |
| Learning curve | Standard JPA concepts | Easier start in Quarkus |
| Query readability | Verbose in many cases | Compact for common queries |
| Control/flexibility | Maximum | High, with optional fallback to JPA |
| Best fit | Complex/custom persistence layers | Fast CRUD and clean Quarkus backends |

---

## Key Concepts Summary

You should be able to explain:

1. what Hibernate ORM does and what Panache adds
2. active record vs repository style in Quarkus
3. why `@Transactional` is mandatory for write operations
4. how Panache queries work for common filters and pagination
5. when to use DTOs and service-layer separation

---

## Related Notes

- [JDBC + PostgreSQL](jdbc-PostgresSQL.md)
- [JDBC + H2](jdbc-h2.md)
- [Flyway Database Migrations](flyway-database-migrations.md)

---

## Conclusion

Hibernate ORM with Panache is a practical default for Quarkus backend development. It keeps JPA power while removing repetitive code, which makes CRUD APIs faster to build and easier to maintain. Use Panache for productivity, and add stronger layering (DTO/service/repository) as project complexity grows.
