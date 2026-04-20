---
title: "DTO Validation and Request Body Validation with @Valid"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-04-18
type: "lesson"
tags:
  - backend
  - rest
  - validation
  - bean-validation
  - dto
  - api-design
status: "in-progress"
---

# DTO Validation and Request Body Validation with @Valid

## Overview

This lesson extends Bean Validation to **full request bodies (DTOs)** in REST APIs.

It focuses on:

- validating complex objects  
- using `@Valid`  
- nested validation  
- structuring validation in real-world APIs  

```text
In practice:
DTO validation is the standard approach for POST and PUT endpoints.
```

---

## Core Concept — DTO Validation

### Definition

Instead of validating individual parameters, validation rules are defined directly on **DTO classes**.

Example:

```java
public class CardCreateDto {

    @NotBlank
    private String question;

    @NotBlank
    private String answer;

    @NotBlank
    private String module;
}
```

-> Validation is attached to the **data model**, not the controller

---

## Why DTO Validation Matters

In real applications:

- clients send JSON request bodies  
- these are mapped to Java objects (DTOs)  
- invalid data must be rejected early  

Without DTO validation:

- validation logic spreads across controllers  
- code becomes repetitive  
- consistency is lost  

-> DTO validation keeps rules centralized and reusable

---

## Using `@Valid` in REST Endpoints

```java
@POST
public Response createCard(@Valid CardCreateDto dto) {
    return Response.ok(cardService.create(dto)).build();
}
```

### What `@Valid` does

```text
Triggers validation of the entire object before method execution
```

- all annotated fields are checked  
- invalid input stops execution  
- a validation exception is thrown automatically  

```text
Important:
Without @Valid, no validation is executed.
```

---

## Validation Flow for Request Bodies

```mermaid
flowchart TD
    A[HTTP Request (JSON)] --> B[Deserialization to DTO]
    B --> C[Bean Validation (@Valid)]
    C --> D{Valid?}
    D -- Yes --> E[Resource Method]
    E --> F[Service Layer]
    D -- No --> G[ConstraintViolationException]
    G --> H[ExceptionMapper]
    H --> I[HTTP 400 Response]
```

-> Invalid data never reaches your business logic

---

## Nested Validation (Important)

DTOs can contain other objects or collections.

```java
public class DeckDto {

    @NotBlank
    private String name;

    @Valid
    private List<CardCreateDto> cards;
}
```

### Key Idea

- `@Valid` enables **recursive validation**
- each nested object is validated automatically

```text
In practice:
This is essential when working with complex request structures.
```

---

## DTO vs Parameter Validation

| Aspect | Query Parameter | DTO Validation |
|-------|----------------|----------------|
| Scope | single value | full object |
| Location | method parameter | class fields |
| Trigger | automatic | requires `@Valid` |
| Use case | filtering | create/update requests |

```text
In practice:
Both approaches are used together in real APIs.
```

---

## Validation Errors

DTO validation can return **multiple errors at once**.

Example:

```json
{
  "message": "Validation failed",
  "errors": [
    "question must not be blank",
    "answer must not be blank"
  ],
  "status": 400
}
```

-> All violations are collected automatically

---

## Custom Error Messages

```java
@NotBlank(message = "Question must not be empty")
private String question;
```

-> Makes API responses clearer and more user-friendly

---

## Common Pitfall — Missing `@Valid`

```java
@POST
public Response createCard(CardCreateDto dto) {
    return Response.ok(service.create(dto)).build();
}
```

❌ No validation happens

-> Always add `@Valid` for request bodies

---

## Additional Useful Annotations

| Annotation | Use Case |
|-----------|--------|
| `@Email` | validate email format |
| `@Positive` | numbers must be > 0 |
| `@Size` | limit length |
| `@Pattern` | enforce formats |

---

## Best Practices

- define validation rules inside DTOs  
- always use `@Valid` for request bodies  
- avoid manual validation for simple rules  
- keep validation separate from business logic  
- use clear and meaningful error messages  

```text
In practice:
Bean Validation handles structure and format,
business logic handles domain rules.
```

---

## Common Mistakes

### 1. Forgetting `@Valid`
-> validation is never triggered  

### 2. Duplicating validation logic
-> annotations + manual checks for same rule  

### 3. Mixing validation and business logic
-> leads to messy and hard-to-maintain code  

---

## Exam Relevance

You should understand:

- what `@Valid` does  
- when validation is triggered  
- difference between parameter and DTO validation  
- how validation errors are handled  
- why validation should be declarative  

---

## Core Insight

```text
Validation belongs to the data model, not the controller logic.
```

---

## Summary

DTO validation with `@Valid` extends Bean Validation to full request bodies.

It allows you to:

- validate complex input structures  
- enforce rules consistently  
- keep controllers clean  
- prevent invalid data from reaching business logic  

-> It is a core concept in modern REST API design and essential for clean backend architecture.