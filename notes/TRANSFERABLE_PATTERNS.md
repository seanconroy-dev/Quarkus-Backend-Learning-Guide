---
title: "Transferable Project Patterns"
project: "FIAE Exam Part 1 Backend"
author: "Sean"
date: 2026-05-20
type: "patterns"
tags:
  - architecture
  - project-design
  - patterns
  - markdown
  - rest-api
  - quarkus
  - deployment
  - content-driven
  - static-sites
status: "reviewed"
---

# Transferable Project Patterns

These notes capture patterns used in this project that apply to any future project —
including a Quarkus backend or any other content-driven app.

---

## 1. Markdown as Content Storage

**Pattern:** store structured content as Markdown files with YAML frontmatter.

Why it works:
- Content is version-controlled alongside code
- Editors can write in plain text
- Frontmatter gives typed metadata without a database

Key decisions:
- Define a strict frontmatter schema up front and never break it
- Use a stable string `id` field (not auto-incremented DB id) so references survive file moves
- Keep a `slug` field for human-readable URLs
- Keep a `status` field (`draft` / `published` / `deprecated`) so new content is invisible until ready

Schema tip: add optional fields (`image`, `examples`) from the start — it is safe to add but costly to remove later.

---

## 2. File Naming Convention

**Pattern:** `<category>/<prefix>-<zero-padded-id>-<slug>.md`

Example: `Beurteilen marktgängiger IT-Systeme/ap1-0071-usv-aufgaben.md`

Why:
- Files sort visually in order
- `id` in filename is redundant with frontmatter but avoids orphaned files
- Easy to grep: `find . -name 'ap1-*.md'`

Rule: ID in filename must always match ID in frontmatter.

---

## 3. REST API Contract — Serving Parsed Markdown

**Pattern:** backend exposes a single endpoint that returns all content as JSON.

```
GET /api/cards/markdown
→ 200 OK
→ Content-Type: application/json
→ Body: Card[]
```

Each item in the array contains:
- all frontmatter fields as flat JSON properties
- `body`: the raw Markdown body (as a string)
- optionally: `category` (folder name the file came from)

Frontend then parses Markdown → HTML client-side (using `marked` here).

Alternatively: backend can pre-render body to HTML and return that directly (reduces frontend dependency on a markdown parser).

---

## 4. Quarkus Implementation of This Pattern

If you implement the `/api/cards/markdown` endpoint in Quarkus:

**Step 1 — Read files at startup**

```java
// Use @ApplicationScoped bean
// Walk the classpath or a configured directory for *.md files
// Parse frontmatter with a YAML lib (e.g. SnakeYAML or Jackson-dataformat-yaml)
// Parse markdown body with a lib (e.g. flexmark-java or commonmark-java)
```

**Step 2 — Map to a record/POJO**

```java
public record Card(
    String id,
    String slug,
    String title,
    String module,
    List<String> topics,
    List<String> tags,
    CardFlashcard card,
    String status,
    String created,
    String updated,
    String body,
    String category
) {}
```

**Step 3 — Expose the endpoint**

```java
@Path("/api/cards")
@Produces(MediaType.APPLICATION_JSON)
public class CardResource {

    @Inject
    CardService cardService;

    @GET
    @Path("/markdown")
    public List<Card> getAll() {
        return cardService.getAllCards();
    }
}
```

**Step 4 — CORS**  
Because the Astro frontend calls this from a different origin, you must configure CORS in Quarkus:

```properties
# application.properties
quarkus.http.cors=true
quarkus.http.cors.origins=https://your-frontend-domain.com
```

For local dev you can also allow `http://localhost:4321` (default Astro dev port).

**Step 5 — Caching**  
Cards change rarely. Cache the result in-memory and reload only on startup (or provide an admin reload endpoint).

```java
@ApplicationScoped
public class CardService {
    private List<Card> cache;

    @PostConstruct
    void init() {
        cache = loadFromDisk();
    }
}
```

---

## 5. Static Site + API Architecture Split

**Pattern:** separate static frontend from a live API backend.

```
GitHub Pages (Astro static build)
        ↓  fetch()
Railway / Fly.io / Render (Quarkus API)
        ↑  reads
Markdown files on classpath or git clone
```

Benefits:
- Frontend is free to host (GitHub Pages)
- Backend is stateless: it only reads files, not a database
- Content can be updated by pushing Markdown to git

Deployment option: backend clones or embeds the repo at build time so Markdown files are on the classpath. No database needed at all.

---

## 6. Module/Category Slug Normalization

**Pattern:** store the human-readable name as the source of truth; derive the URL slug programmatically.

The normalization logic used here (TypeScript, but easy to port):

```
1. toLowerCase()
2. ä → ae, ö → oe, ü → ue, ß → ss
3. remove chars that are not a-z, 0-9, space, -, _
4. collapse spaces to -
5. trim
```

In Java (Quarkus or elsewhere):

```java
public static String toSlug(String module) {
    return module.toLowerCase()
        .replace("ä", "ae").replace("ö", "oe")
        .replace("ü", "ue").replace("ß", "ss")
        .replaceAll("[^a-z0-9\\s_-]", "")
        .trim()
        .replaceAll("\\s+", "-")
        .replaceAll("-+", "-");
}
```

Important: both frontend and backend must use the same algorithm so URLs resolve correctly.

---

## 7. GitHub Actions CI/CD Pattern for Static Sites

Minimal deploy-on-push pattern:

```yaml
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
          cache-dependency-path: site/package-lock.json
      - run: npm ci
        working-directory: site
      - run: npm run build
        working-directory: site
      - uses: actions/upload-pages-artifact@v3
        with:
          path: site/dist
      # deploy step needs pages environment + id-token write permission
```

For a Quarkus backend CI/CD the pattern is the same but replace the build step with:

```bash
./mvnw package -DskipTests
# then push Docker image or deploy JAR
```

---

## 8. Environment Variables Pattern

**Pattern:** never hardcode external URLs; always use env vars.

In Astro/Vite: `PUBLIC_` prefix makes the variable available in browser code.

```env
PUBLIC_API_BASE=https://your-backend.example.com
```

In Quarkus: use `application.properties` or environment override:

```properties
# application.properties
cards.source.path=${CARDS_PATH:/cards}
frontend.cors.origin=${FRONTEND_ORIGIN:http://localhost:4321}
```

Switching between local dev and production requires only changing env, not code.

---

## 9. Content Lifecycle States

Using a `status` field to control visibility is a simple but powerful pattern for any content system:

| Status | Meaning |
|--------|---------|
| draft | work in progress, not shown publicly |
| published | visible to users |
| deprecated | kept for reference, hidden from new views |

Filter published at the API or in the frontend query — your choice. Filtering at the API reduces payload size.

---

## 10. Reusable Takeaways (Any Stack)

- **Design the content schema before writing content.** Changing it later is painful.
- **Use stable IDs.** Auto-increment breaks when you move files or re-import.
- **One source of truth.** Frontmatter = metadata; body = explanation. Do not duplicate.
- **Filter at query time, not at write time.** Keep drafts in the repo; filter them out at the API layer.
- **Separate content from presentation.** Markdown body → any renderer → any UI.
- **Cache aggressively** for read-heavy, rarely-changing content (learning cards, docs).
- **CORS must be configured deliberately** on any API that is called by a different-origin frontend.
