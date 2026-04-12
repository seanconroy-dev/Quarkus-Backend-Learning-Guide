```markdown
---
title: "Error Handling in REST APIs"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-04-11
type: "lesson"
tags:
  - backend
  - rest
  - error-handling
  - http-status-codes
  - api-design
status: "in-progress"
---

# Error Handling in REST APIs

## Lesson Overview

This lesson introduces how a backend should handle errors and edge cases in a REST API.

The focus is on designing **predictable and consistent responses**, especially when requested data does not exist or input is invalid.

A key part of backend design is understanding when something is considered an error and how that should be communicated to the client.

---

## Core Concept — Error Handling in REST

### Definition

Error handling in REST APIs defines how the backend responds when:

- a requested resource does not exist  
- input is invalid  
- an operation cannot be completed  

The goal is to provide:

- correct HTTP status codes  
- consistent response structures  
- clear and meaningful error messages  

---

## Two Different Situations

## 1. Single Resource Not Found

```text
GET /api/cards/999
```

### Expected Response

```json
{
  "message": "Card not found",
  "status": 404,
  "timestamp": "2026-04-11T12:00:00Z"
}
```

### Explanation

- the client requested a **specific resource**  
- the resource does not exist  
- this is considered an **error condition**  

### Result

- HTTP status: `404 Not Found`  
- response contains an error object  

---

## 2. Filtered Collection with No Results

```text
GET /api/cards?module=DoesNotExist
```

### Expected Response

```json
[]
```

### Explanation

- the client requested a **collection of resources**  
- filtering resulted in **zero matches**  
- this is a **valid result**, not an error  

### Result

- HTTP status: `200 OK`  
- empty list is returned  

---

## Rule of Thumb

```text
Single resource missing → 404 Not Found
Collection with no matches → empty list []
Invalid input → 400 Bad Request
```

---

## Why This Distinction Matters

The difference is based on **intent**:

| Request Type | Meaning | Result |
|---|---|---|
| `/api/cards/999` | “Give me this specific object” | Error if missing |
| `/api/cards?module=X` | “Give me all matching objects” | Empty result is valid |
| `/api/cards?banana=test` | Invalid parameter | Error (400) |

---

## Implementation — Exception Mapper

```java
@Provider
public class GlobalExceptionMapper implements ExceptionMapper<NotFoundException> {

    @Override
    public Response toResponse(NotFoundException exception) {
        ErrorResponseDto error = new ErrorResponseDto(
            exception.getMessage(),
            404,
            Instant.now().toString()
        );

        return Response.status(Response.Status.NOT_FOUND)
            .entity(error)
            .type(MediaType.APPLICATION_JSON)
            .build();
    }
}
```

---

## Supporting DTO

```java
public class ErrorResponseDto {
    public String message;
    public int status;
    public String timestamp;

    public ErrorResponseDto(String message, int status, String timestamp) {
        this.message = message;
        this.status = status;
        this.timestamp = timestamp;
    }
}
```

---

## Explanation

- `timestamp` helps track when the error occurred  
- useful for debugging and logging  
- keeps error responses consistent and informative  

---

## What to Avoid

- returning `404` for filtered collections  
- returning HTML error pages instead of JSON  
- using inconsistent error formats  
- exposing internal errors directly  

---

## Key Insight

```text
APIs should be predictable.

Same type of request → same structure of response
```

Consistency is more important than strictness.

---

## Exam Relevance

This topic is relevant for:

- HTTP status codes  
- REST API design principles  
- distinguishing valid results vs errors  
- consistent response design  

You should be able to explain:

- when to return `404 Not Found`  
- why an empty list is not an error  
- when to return `400 Bad Request`  
- how backend systems standardize error responses  

---

## Next Step

Validation of input:

```text
GET /api/cards?banana=test → 400 Bad Request
```

---

## Core Insight

Error handling in REST is not only about technical failures.

It is about correctly interpreting the **intent of a request** and responding in a consistent and predictable way.
```