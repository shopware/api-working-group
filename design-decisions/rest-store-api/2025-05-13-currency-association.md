## Overview

**Title:** Allow currency to be associated with price in API response

## Description

Currently, the Store API returns product prices (e.g., `product.price`) without currency information. To display complete pricing such as `€29.99`, clients must query `/store-api/context` or hardcode the currency context on the frontend. This adds complexity, particularly for server-side rendering, headless storefronts, and internationalized environments.

This design requires developers to manage and inject the context manually for what is typically considered basic product information.

## Proposed Changes

Allow the `currency` entity to be optionally loaded via the existing `associations` parameter, consistent with how related entities like `manufacturer`, `media`, or `properties` are handled.

This approach makes the currency data available when explicitly requested by the client, keeping the default response minimal while improving developer experience and use-case flexibility.

## Example Usage

### Request

```http
POST /store-api/product
Content-Type: application/json

{
  "ids": ["abc123"],
  "associations": {
    "currency": {}
  }
}
```

### Response

```json
{
  "elements": [
    {
      "id": "abc123",
      "price": 29.99,
      "currency": {
        "isoCode": "EUR",
        "symbol": "€"
      }
    }
  ]
}
```

If the currency association is not requested, it is omitted from the response as expected.

## Alternatives Considered

1. Query parameter: ?expand=currency
This approach would be easy to use in GET endpoints but is inconsistent with how the Store API handles relations for POST endpoints like /store-api/product.

2. HTTP Header: X-Expand: currency
Using a custom header would also allow optional enrichment, but it introduces a new API behavior pattern that is not currently part of Shopware’s REST API structure.

3. Always include currency in responses
This would simplify frontend handling but result in bloated responses and duplicate data when returning multiple products. It also breaks from the principle of explicitly requested expansions used elsewhere in the API.

The associations pattern is already supported and provides a natural and scalable way to conditionally include related data.

## OpenAPI Snippet

The following OpenAPI addition documents the optional currency field as part of the Product schema when requested via associations.

```yaml

components:
  schemas:
    Product:
      type: object
      properties:
        id:
          type: string
        name:
          type: string
        price:
          type: number
        currency:
          $ref: '#/components/schemas/Currency'

    Currency:
      type: object
      properties:
        isoCode:
          type: string
          example: "EUR"
        symbol:
          type: string
          example: "€"
        name:
          type: string
          example: "Euro"

```

In all endpoints that support associations, such as /store-api/product, it should be documented that currency can be included as an optional relationship.

## Proof of Concept

A draft implementation is available in the following repository and branch:

[shopware/shopware@poc/currency-association-for-price](https://github.com/shopware/shopware/tree/poc/currency-association-for-price)


## Benefits

- Follows existing API design patterns (associations)

- Reduces need for context fetching in simple product use-cases

- Improves frontend development speed and reduces complexity

- Enables SSR and multi-channel support without custom hacks

- Fully backwards-compatible