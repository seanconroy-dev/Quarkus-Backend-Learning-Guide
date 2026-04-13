---
title: "Query Parameter Validation in REST APIs"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-04-12
type: "lesson"
tags:
  - backend
  - rest
  - validation
  - api-design
  - query-parameters
status: "in-progress"
---

# Query Parameter Validation in REST APIs

## Lesson Overview

This lesson documents how query parameters are validated in a REST API and why strict validation is important for backend design.

It is based on practical implementation and focuses on turning that implementation into reusable understanding.

The goal is to ensure that only **valid and supported input** is accepted and that invalid requests are rejected clearly and consistently.

---

## Core Concept — Query Parameter Validation

### Definition

Query parameter validation ensures that:

- only supported query parameters are accepted  
- unexpected or invalid parameters are rejected  
- incorrect input is not silently ignored  

---

## Problem Without Validation

Request:

```text
GET /api/cards?banana=test
```

### Behavior

```json
[]
```

### Issue

- invalid parameter is ignored  
- response appears valid but is misleading  
- debugging becomes difficult  
- API behavior becomes inconsistent  

---

## Correct Behavior

Request:

```text
GET /api/cards?banana=test
```

### Response

```json
{
  "message": "Invalid query parameter: banana",
  "status": 400,
  "timestamp": "2026-04-12T10:00:00Z"
}
```

### Explanation

- parameter is not supported  
- request is invalid  
- API responds with **400 Bad Request**  

---

## Rule of Thumb

```text
Allowed parameters → process request
Unknown parameters → reject request (400)
```

---

## Implementation — Validator Class

```java
@ApplicationScoped
public class QueryParamValidator {

    public void validateAllowedParams(
        MultivaluedMap<String, String> queryParams,
        Set<String> allowedParams
    ) {
        for (String param : queryParams.keySet()) {
            if (!allowedParams.contains(param)) {
                throw new BadRequestException(
                    "Invalid query parameter: " + param
                );
            }
        }
    }
}
```

---

## Implementation — Resource Usage

```java
private static final Set<String> ALLOWED_QUERY_PARAMS = Set.of("module");

@Inject
QueryParamValidator queryParamValidator;

@GET
@Produces(MediaType.APPLICATION_JSON)
public List<CardDto> getCards(@Context UriInfo uriInfo,
                             @QueryParam("module") String module) {

    queryParamValidator.validateAllowedParams(
        uriInfo.getQueryParameters(),
        ALLOWED_QUERY_PARAMS
    );

    if (module != null) {
        return cardService.getByModule(module);
    }

    return cardService.getCards();
}
```

---

## Explanation

- `UriInfo` provides all query parameters dynamically  
- validator checks each parameter  
- only explicitly allowed parameters are accepted  
- invalid input results in an exception  

---

## Key Concept — Separation of Concerns

```text
Resource  → handles HTTP layer
Validator → validates input
Service   → handles business logic
```

### Why this matters

- avoids mixing responsibilities  
- improves readability  
- makes validation reusable  
- keeps logic maintainable  

---

## Why This Approach Is Better

Before:

- hardcoded checks  
- not reusable  
- difficult to extend  

After:

- generic validation  
- reusable component  
- scalable design  

---

## What to Avoid

- hardcoding specific invalid parameters  
- ignoring unknown parameters  
- mixing validation with business logic  
- duplicating validation logic  

---

## Key Insight

```text
APIs should fail fast on invalid input.
Do not guess what the client meant.
Reject incorrect requests clearly.
```

---

## Exam Relevance

Relevant concepts:

- input validation  
- REST API design  
- HTTP status codes (`400 Bad Request`)  
- separation of concerns  

You should be able to explain:

- why invalid parameters must not be ignored  
- when to return `400 Bad Request`  
- how validation improves API reliability  

---

## Next Step

Extend validation to:

```text
- multiple parameters
- combined filters
- reusable validation strategies
```

Example:

```text
/api/cards?module=Hardware&title=DisplayPort
```

---

## Core Insight

Validation ensures that an API behaves in a **strict, predictable, and trustworthy** way.
