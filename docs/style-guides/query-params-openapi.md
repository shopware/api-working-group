# 📐 API Style Guide: Query Parameters in OpenAPI + Symfony

Query parameters are ideal for selecting, filtering, and shaping list or resource responses. **Overusing them for actions or complex payloads leads to unclear semantics and poor tooling**.

---

## ✅ Do

- **Use query params for resource selection and representation shaping:**
  - 🔎 Filtering, search, and facets → `status=pending`, `q=iphone`
  - 🧭 Pagination and sorting → `page=2`, `pageSize=50`, `sort=-createdAt`
  - 🧩 Sparse fieldsets and expansions → `fields=Id,Name`, `include=items`
  - 🌍 Locale/preferences that vary the representation → `lang=de`, `currency=EUR`

- **Keep them simple and explicit:**
  - Prefer primitives (`string`, `integer`, `number`, `boolean`, `enum`)
  - Use repeated params for multi-select: `?status=pending&status=paid`
  - Define clear defaults; make optional params truly optional

- **Model them correctly in OpenAPI:**
  - Use `in: query` with `schema`, `allowEmptyValue: false`
  - For arrays: `style: form`, `explode: true` (default) → `?tag=a&tag=b`
  - Document pagination shape consistently across endpoints

---

## ❌ Don’t

- ❌ **Don’t encode actions or state changes in query params**  
  Use the request body for actions/mutations; query params should be safe/idempotent selectors

- ❌ **Don’t send complex nested JSON in query params**  
  If you need objects or deep nesting, use a request body

- ❌ **Don’t overload GET with body-like semantics**  
  Long/encoded filters belong to `POST /search` with a body, not `GET` query

- ❌ **Don’t mix transport concerns**  
  Authentication, content negotiation, and idempotency belong in headers (see [Headers Style Guide](./headers-openapi.md)), not query

---

## 👎 Bad Examples

```yaml
paths:
  /orders/search:
    get:
      parameters:
        - in: query
          name: filter
          schema: { type: string } # JSON string like {"date":{"from":"..."}}
```

⚠️ Issues: unclear validation, fragile encoding, poor caching, hard to document.

---

## 👍 Good Patterns (4 route examples)

```yaml
paths:
  /products:
    get:
      summary: List products
      parameters:
        - in: query
          name: q
          description: Full-text search query
          schema: { type: string }
        - in: query
          name: category
          schema: { type: string, enum: [phones, laptops, accessories] }
        - in: query
          name: tag
          description: Multi-select tags
          schema:
            type: array
            items: { type: string }
          style: form
          explode: true
        - in: query
          name: sort
          description: Sort by field; prefix with '-' for descending
          schema: { type: string, enum: [name, -name, price, -price, createdAt, -createdAt] }
        - in: query
          name: page
          schema: { type: integer, minimum: 1, default: 1 }
        - in: query
          name: pageSize
          schema: { type: integer, minimum: 1, maximum: 100, default: 25 }
      responses:
        '200': { description: OK }

  /orders:
    get:
      summary: List orders with status and date filters
      parameters:
        - in: query
          name: status
          schema: { type: string, enum: [pending, paid, shipped, cancelled] }
        - in: query
          name: customerId
          schema: { type: string }
        - in: query
          name: from
          description: ISO 8601 date lower bound
          schema: { type: string, format: date }
        - in: query
          name: to
          description: ISO 8601 date upper bound
          schema: { type: string, format: date }
        - in: query
          name: include
          description: Expand related resources
          schema: { type: string, enum: [items, customer, items,items.product] }
      responses:
        '200': { description: OK }

  /customers:
    get:
      summary: List customers with projection and locale
      parameters:
        - in: query
          name: fields
          description: Comma-separated field selection
          schema: { type: string, example: "id,name,email" }
        - in: query
          name: lang
          schema: { type: string, minLength: 2, maxLength: 5, example: "de" }
      responses:
        '200': { description: OK }

  /reports/sales:
    get:
      summary: Retrieve a cached sales report (safe, cacheable)
      parameters:
        - in: query
          name: period
          schema: { type: string, enum: [daily, weekly, monthly] }
        - in: query
          name: region
          schema: { type: string }
      responses:
        '200': { description: OK }
```

---

## 🧭 Symfony Notes

- Use request query accessors: `$request->query->get('q')`, `$request->query->all('tag')`
- Prefer attribute-based validation (e.g., Symfony Validator) for query DTOs
- Keep IDs and routing decisions in the path; use query for optional filters

---

## 🎯 Quick Checklist

1. **Does this parameter select/filter/shape a representation?**  
   → ✅ Use a query param

2. **Is it an action or complex structure?**  
   → ❌ Use a request body or a dedicated route

3. **Will URLs be cacheable and linkable?**  
   → ✅ Prefer query params for safe, idempotent reads

---

## 📚 References

- [Headers Style Guide](./headers-openapi.md) – Headers in OpenAPI + Symfony
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0#parameterObject) – Query parameter modeling
- [RFC 3986 (URI)](https://www.rfc-editor.org/rfc/rfc3986) – Query component semantics
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#751-query-parameters) – Filtering/pagination guidance
- [Symfony HttpFoundation](https://symfony.com/doc/current/components/http_foundation.html#accessing-request-data) – Accessing query params

---
