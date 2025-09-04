# 📐 API Style Guide: Headers in OpenAPI + Symfony

Headers are part of HTTP, but **overusing them makes APIs hard to describe, generate, and use**.  
This guide explains when headers are appropriate and when they are not.

---

## ✅ Do

- **Use headers only for true request metadata:**
  - 🔑 Authentication → `Authorization`, `Bearer ...`
  - 📦 Content negotiation → `Accept`, `Content-Type`
  - 🗄️ Caching & preconditions → `ETag`, `If-None-Match`, `If-Match`
  - 🛠️ Operational metadata → `Idempotency-Key`, `X-Request-ID`

- **Model them correctly in OpenAPI:**
  - Authentication → `securitySchemes` / `security`
  - Content negotiation → `requestBody.content`, `responses[*].content`

- **Keep them simple:**
  - Use primitives (`string`, `boolean`, `integer`)
  - Avoid arrays/objects — tooling support is poor

- **Expect normalization:**
  - Header names are case-insensitive
  - Symfony’s `HeaderBag` lowercases names internally

- **Document operational headers clearly**
  - Example: `X-Request-ID` (diagnostics), `Idempotency-Key` (retry safety)

---

## ❌ Don’t

- ❌ **Don’t use headers for routing or filtering**  
  Symfony routing is based on **path + method**, not hidden header switches  
  See [Query Parameters Style Guide](./query-params-openapi.md).

- ❌ **Don’t version your API in headers**  
  Avoid `X-API-Version`. Use `/v1/...` in the URI or vendor-specific media types

- ❌ **Don’t put filters/toggles in headers**  
  Example:

  ```http
  GET /orders
  X-Region: EU
  X-Show-Archived: true
  ```

  ➡️ Should be query params instead — see [Query Parameters Style Guide](./query-params-openapi.md).

- ❌ **Don’t redefine standard headers**  
  Never add `Accept`, `Content-Type`, or `Authorization` as header parameters in OpenAPI

- ❌ **Don’t overload routes with many custom headers**  
  They are hard to document, poorly supported in SDKs, and break caching (`Vary` headers)

---

## 👎 Bad Example: Filters in Headers

```yaml
paths:
  /orders:
    get:
      parameters:
        - in: header
          name: X-Region
          schema: { type: string }
        - in: header
          name: X-Show-Archived
          schema: { type: boolean }
```

⚠️ **Issues:** Hidden API surface, poor client generation, messy caching.

---

## 👍 Good Example: Filters as Query Params

```yaml
paths:
  /orders:
    get:
      parameters:
        - in: query
          name: region
          schema: { type: string }
        - in: query
          name: showArchived
          schema: { type: boolean, default: false }
```

✅ **Benefits:** Clear docs, better SDKs, cache-friendly URLs, simpler Symfony routing.  
For modeling details, see [Query Parameters Style Guide](./query-params-openapi.md).

---

## 🎯 Quick Checklist

Before adding a header, ask yourself:

1. **Is this request metadata or transport concern?**  
   → ✅ Use a header  

2. **Does this change resource selection or representation?**  
   → ❌ Use query params, path, or body — see [Query Parameters Style Guide](./query-params-openapi.md)

3. **Does OpenAPI already model this explicitly?**  
   → ❌ Don’t override with custom headers  

---

## 📚 References

- [Query Parameters Style Guide](./query-params-openapi.md) – Filtering, pagination, projections
- [OpenAPI Specification](https://spec.openapis.org/oas/v3.1.0#parameterObject) – Parameter types & restrictions  
- [RFC 9110 (HTTP Semantics)](https://www.rfc-editor.org/rfc/rfc9110.html) – Header case-insensitivity, caching rules  
- [Symfony HttpFoundation](https://symfony.com/doc/current/components/http_foundation.html#accessing-headers) – `HeaderBag` behavior  
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#73-headers) – Minimal, standard header use  

---
