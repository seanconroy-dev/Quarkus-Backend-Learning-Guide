---
title: "Service Layer Logging in REST APIs"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-04-17
type: "lesson"
tags:
  - backend
  - rest
  - logging
  - service-layer
  - clean-code
status: "in-progress"
---

# Service Layer Logging in REST APIs

## Lesson Overview

This lesson explains **when and how logging should be used in the service layer** of a backend application.

It builds on global error handling and focuses on:

- controlled logging  
- meaningful log messages  
- avoiding over-logging  

The goal is to make the backend **traceable without becoming noisy**.

---

## Core Concept — Service Layer Logging

### Definition

Service layer logging means:

- logging **important application events**  
- not just errors  
- at the level where business logic happens  

---

## Why Logging in the Service Layer?

Global logging (500 errors) only shows failures.

But in real systems, you also want to know:

- what the system is doing  
- what data is being processed  
- where something might go wrong  

---

## Key Principle

```text
Log what is meaningful, not everything.
```

---

## What SHOULD be logged

### 1. Important operations

```text
Loading data
Filtering data
Starting processes
```

---

### 2. Unexpected situations (non-fatal)

```text
Empty results
Missing optional data
Fallback logic triggered
```

---

### 3. Critical flow decisions

```text
Branching logic
Conditional processing
```

---

## What should NOT be logged

### ❌ Every method call

```java
LOG.info("Entering getCards()");
```

---

### ❌ Every variable

```java
LOG.debug("x = " + x);
```

---

### ❌ Duplicate error logging

Errors are already logged in:

```text
GlobalExceptionMapper
```

👉 Do NOT log them again in the service unless adding context

---

## Logging Levels

| Level | Use Case |
|------|--------|
| error | system failure |
| warn | unexpected but handled |
| info | important events |
| debug | detailed dev info |

---

## Recommended Usage (for this project)

- `error` → already handled globally  
- `info` → important operations  
- `warn` → unusual situations  

---

## Practical Example — CardService

### Add Logger

```java
private static final Logger LOG = Logger.getLogger(CardService.class);
```

---

### Example: Loading Data

```java
public List<CardDto> getCards() {
    LOG.info("Loading cards from JSON file");

    try (InputStream inputStream = getClass().getClassLoader().getResourceAsStream("seed/cards.json")) {
        if (inputStream == null) {
            LOG.warn("cards.json not found");
            throw new RuntimeException("cards.json not found");
        }

        List<CardDto> cards = objectMapper.readValue(
            inputStream,
            new TypeReference<List<CardDto>>() {}
        );

        LOG.info("Loaded " + cards.size() + " cards");

        return cards;

    } catch (Exception e) {
        throw new RuntimeException("Failed to load cards.json", e);
    }
}
```

---

### Example: Filtering

```java
public List<CardDto> getByModule(String module) {
    LOG.info("Filtering cards by module: " + module);

    List<CardDto> result = getCards().stream()
        .filter(card -> module.equalsIgnoreCase(card.module))
        .toList();

    if (result.isEmpty()) {
        LOG.warn("No cards found for module: " + module);
    }

    return result;
}
```

---

## Explanation

- `info` logs show normal system behavior  
- `warn` logs highlight unusual situations  
- no duplicate error logging is done  

---

## Key Concept — Signal vs Noise

```text
Good logs → useful, meaningful, readable
Bad logs  → spam, clutter, ignored
```

---

## Bad Example

```java
LOG.info("Entering method");
LOG.info("Variable x = " + x);
LOG.info("Leaving method");
```

👉 No real value

---

## Good Example

```java
LOG.info("Loading cards from JSON");
LOG.warn("No cards found for module: Security");
```

👉 Clear and useful

---

## Why This Matters

In real systems:

- logs are used for debugging  
- logs are used for monitoring  
- logs are used in production  

If logs are noisy → they are ignored  
If logs are meaningful → they are powerful  

---

## Key Concept — Complement to Error Handling

```text
Error Handling → controls API behavior
Logging        → explains system behavior
```

Both are required for a complete backend.

---

## Exam Relevance

Relevant concepts:

- logging levels  
- separation of concerns  
- clean backend design  
- maintainability  

You should be able to explain:

- when to use `info`, `warn`, `error`  
- why not everything should be logged  
- why logging belongs in the service layer  

---

## Common Mistakes

### ❌ Logging everything

→ creates noise

---

### ❌ Logging nothing

→ no visibility

---

### ❌ Logging errors twice

→ redundant and confusing

---

### ❌ Using wrong log levels

→ makes logs harder to read

---

## Next Step

Extend logging with:

```text
- structured logging (JSON logs)
- correlation IDs
- request tracing
```

---

## Core Insight

```text
Good logging makes a system understandable.
Bad logging makes it unreadable.
```

---

## Summary

Service layer logging focuses on:

- meaningful events  
- correct log levels  
- avoiding noise  

Together with error handling, it makes the backend:

- observable  
- debuggable  
- maintainable  